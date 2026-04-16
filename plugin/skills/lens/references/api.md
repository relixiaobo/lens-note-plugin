# API Reference

Complete reference for lens read and write APIs, output formats, common workflows, and pitfalls.

## Contents

- [JSON Envelope](#json-envelope)
- [Read API — Output Formats](#read-api--output-formats)
- [Write API Reference](#write-api-reference)
- [Combining Commands](#combining-commands)
- [Common Pitfalls](#common-pitfalls)

---

## JSON Envelope

All `--json` output uses a stable envelope:

```json
// Success
{"ok": true, "data": { ... payload ... }}

// Error
{"ok": false, "error": {"code": "command_error", "message": "..."}, "hint": "..."}

// Deprecation
{"ok": false, "error": {"code": "deprecated_command", "message": "..."}, "replacement": "..."}
```

Always check `ok` first. On success, read `data`. On failure, read `error.code` and `error.message`. Optional fields: `hint` (next action), `command` (which command failed), `candidates` (ambiguous match results).

**Partial batch failure**: `{"ok": false, "error": {"code": "partial_failure", "message": "2 of 5 item(s) failed"}, "data": {"results": [...]}}` — individual results are still in `data.results`, each with its own `action` field.

**All examples below show the `data` payload only** (the object inside `data`), not the full envelope.

---

## Read API — Output Formats

### search

```json
{"query": "...", "total": 15, "count": 3, "results": [
  {"id": "note_01A", "type": "note", "title": "...", "links": 5},
  {"id": "src_01B", "type": "source", "title": "...", "source_type": "web_article", "word_count": 2718},
  {"id": "task_01C", "type": "task", "title": "...", "status": "open"}
], "limit": 3}
```

`total` is total matches; `count` is returned (after `--limit`). `links` is the number of forward links (notes only). Use `--limit N` to cap results.

`--resolve` resolves a query to a single object ID using three fallback strategies:

1. **ID match**: If the query looks like a valid ID (`note_`/`src_`/`task_` + 26 chars), returns it directly as `{id, title}`.
2. **Title match**: Case-insensitive exact title search. Returns `{id, title}` on unique match, or `{ok: false, error: {code: "ambiguous_match"}, candidates: [...]}` if multiple notes share the title.
3. **FTS5 ranked search**: Full-text search. Returns `{id, title}` if exactly one result, `{ok: false, error: {code: "ambiguous_match"}, candidates: [...]}` (top 5) if multiple, or `{ok: false, error: {code: "no_match"}}` if zero.

### show

Accepts ID or title. If the input is not a valid ID format, auto-resolves by exact title match → FTS5 search. If ambiguous, returns `{ok: false, error: {code: "ambiguous_match"}, candidates: [...]}`.

**Batch mode**: `lens show id1 id2 id3 --json` returns `{"count": 3, "items": [{...}, {...}, {...}]}` (inside `data`).

Single object output:

```json
{"id": "note_01A", "type": "note", "title": "...", "body": "...",
 "forward_links": [{"id": "note_01B", "rel": "supports", "reason": "...", "title": "Target title"}],
 "backward_links": [{"id": "note_01C", "rel": "refines", "title": "Source title"}]}
```

`forward_links` and `backward_links` are arrays at the top level. Each link item has `id`, `rel`, `title`, and optionally `reason`.

### links

Accepts ID or title (same auto-resolve as `show`). Shows all relationships for an object:

```json
{"id": "note_01A",
 "forward": [{"id": "note_01B", "rel": "supports", "type": "note", "title": "Target title", "reason": "provides evidence for..."}],
 "backward": [{"id": "note_01C", "rel": "refines", "type": "note", "title": "Source title"}]}
```

Only real graph edges (from `links[]` in YAML) appear here. Forward links include `reason` when present. The `source` metadata field is NOT included — use `show` to see a note's source.

**Filters**: `--rel related` returns only related links. `--direction forward` returns only outgoing links. Combine both: `--rel related --direction forward`. Valid rels: supports, contradicts, refines, related, indexes. Valid directions: forward, backward.

Use `links` to explore the graph — follow connections to discover related knowledge.

### list

```json
{"type": "notes", "total": 897, "count": 3, "items": [
  {"id": "note_01A", "title": "...", "links": 5},
  {"id": "note_01B", "title": "...", "links": 2},
  {"id": "note_01C", "title": "..."}
], "limit": 3}
```

`total` is total matching objects; `count` is returned (after `--limit`/`--offset`). `links` is the number of forward links (count only, not full arrays). Use `--limit N` and `--offset N` for pagination.

With `--orphans`: `{"type": "notes", "filter": "orphans", "count": 5, "items": [{"id": "...", "title": "...", "preview": "..."}]}`

Flags: `--since 7d` (time filter: `Nd`/`Nw`/`Nm`/`Ny`), `--orphans` (notes only), `--min-links N` / `--max-links N` (total link count), `--source-type <type>` (sources only), `--status open|done` (tasks only), `--limit N`, `--offset N`.

### search --expand

Use `search --expand` when you need to **synthesize** across multiple notes (e.g., answering "what do I know about X"). Use plain `search` when you just need to **find** matching notes by title/keyword.

```json
{"query": "...", "timestamp": "...", "total_results": 5,
 "notes": [{"id": "note_01A", "title": "...", "source": "src_01B", "forward_links": [...], "body": "full body...",
            "body_refs": [{"id": "note_01B", "title": "Referenced note"}]}]}
```

`body_refs` is optional — only included when the body contains `[[ID]]` inline references. Only includes notes, not sources or tasks.

### digest

Groups recent notes by link type. Categorization: has `contradicts` link → **tensions**, has other links → **connected**, no links → **seeds**.

```json
{"period": "7d", "total": 12,
 "tensions": [{"id": "note_01A", "title": "...", "links": {"count": 3, "rels": {"supports": 2, "contradicts": 1}}}],
 "connected": [{"id": "note_01C", "title": "...", "links": {"count": 2, "rels": {"related": 1, "refines": 1}}}],
 "seeds": [{"id": "note_01D", "title": "..."}]}
```

Each note's `links` is a compact summary: `count` (total) and `rels` (breakdown by type). Use `lens show <id>` to see full link details.

Accepts `week`/`month`/`year` or `--days N` (default: 1 day). Use to review recent work and spot contradictions worth exploring.

### lint

Full quality report with 9 checks and offender IDs. Checks: `supports_density`, `contradicts_count`, `super_connectors`, `missing_reasons`, `vague_reasons`, `duplicate_links`, `dead_links`, `thin_notes`, `superseded_alive`.

Use `--check` for CI: exit code 1 on failures (warnings don't fail).

Note: `indexes` links are exempt from `missing_reasons` and `vague_reasons` checks (structural links are self-explanatory; only semantic links need reasons).

```json
{"checks": [{"name": "supports_density", "status": "ok", "value": 0.42, "threshold": 0.1, "message": "supports density is 0.42 (460 supports / 1100 notes, 1360 related). Evidence backbone is healthy."}],
 "summary": {"total_checks": 9, "passed": 8, "warnings": 1, "failures": 0}}
```

### lint --audit \<check\>

Deep-dive mode: returns ALL offenders for one check with full context (titles, reasons), for batch fixing. Supports `--limit`/`--offset`.

Available audit checks: `supports_density` (all supports links, for evidence backbone review), `related_dominance` (all related links, useful for curation — not a lint check, audit-only), `missing_reasons`, `vague_reasons`, `duplicate_links`, `thin_notes`, `superseded_alive`.

```bash
lens lint --audit supports_density --json     # review evidence backbone
lens lint --audit related_dominance --json    # find related links to cull or upgrade
lens lint --audit duplicate_links --json
lens lint --audit missing_reasons --limit 20 --json
```

Output varies by check type:

**Edge-shaped** (supports_density, related_dominance, missing_reasons, vague_reasons):
```json
{"check": "supports_density", "total_links": 460, "count": 460,
 "offenders": [{"from": "note_01A", "from_title": "...", "to": "note_01B", "to_title": "...", "rel": "supports", "reason": "..."}]}
```

**Pair-shaped** (duplicate_links — includes keep/remove suggestion by rel strength):
```json
{"check": "duplicate_links", "total_pairs": 5,
 "offenders": [{"from": "note_01A", "from_title": "...", "to": "note_01B", "to_title": "...", "rels": ["refines","supports"], "keep": "refines", "remove": ["supports"]}]}
```

**Note-shaped** (thin_notes, superseded_alive):
```json
{"check": "superseded_alive", "total_notes": 2,
 "offenders": [{"id": "note_01A", "title": "...", "active_inbound": [{"from": "note_01B", "rel": "supports"}]}]}
```

**Workflow**: `lint` → identify failing check → `lint --audit <check>` → agent processes offenders → `write` batch retype/unlink → `lint` verify.

### lint --summary

Quick stats + graph health + user context (replaces `status`):

```json
{"path": "/Users/.../.lens", "notes": 900, "sources": 105,
 "tasks": {"open": 7, "done": 0, "total": 7}, "total": 1012,
 "connectivity": {"orphan_count": 4, "orphan_rate": 0.4, "total_links": 2424, "cross_source_pct": 2.0},
 "link_types": {"related": 1651, "supports": 451, "refines": 200, "contradicts": 16, "indexes": 110},
 "context": {"role": "...", "audience": "...", "language": "...", "style": "..."}}
```

### fetch

Extracts web content as clean markdown. With `--save`, also creates a source object.

```json
{"title": "Article Title", "author": "Author Name", "url": "https://...",
 "word_count": 2718, "markdown": "# Article...", "source_id": "src_01A (only with --save)"}
```

### note (shortcut)

`lens note "title"` is equivalent to `lens write '{"type":"note","title":"..."}'`. Creates a note with no body or links — use for quick capture.

### ingest (shortcut)

`lens ingest <url>` is equivalent to `lens fetch <url> --save`. Fetches web content and saves as a source.

`lens ingest <file>` imports a local file as a source. Auto-detects file type: `.md`/`.markdown` files become `source_type: "markdown"`, all others become `source_type: "plain_text"`.

### feed subcommands

```bash
lens feed add <url> --json        # Subscribe (auto-discovers RSS from HTML)
lens feed list --json              # List subscriptions
lens feed check [--dry-run] --json # Check for new articles (--dry-run: preview only)
lens feed import <opml-file> --json # Import OPML subscriptions
lens feed remove <id|url> --json  # Unsubscribe
```

`feed add` output: `{"id": "...", "url": "...", "title": "..."}}`
`feed list` output: `{"count": N, "feeds": [{"id": "...", "url": "...", "title": "...", "last_checked_at": "..."}]}`
`feed check` output: `{"new_articles": N, "articles": [{"title": "...", "url": "...", "feedTitle": "..."}]}`
`feed import` output: `{"imported": N, "skipped": N, "feeds": [...]}`
`feed remove` output: `{"removed": "feed_id", "url": "..."}`

### rebuild-index

Rebuilds SQLite cache from markdown files. Use when the index is corrupted or after manual file edits outside lens:

```json
{"indexed": 350, "elapsed_ms": 120}
```

### index (Schlagwortregister)

A sparse keyword index — curated entry points into the knowledge graph. Each keyword maps to 1-3 note IDs (the best starting points for a topic). From the entry point, follow links to explore.

```bash
lens index --json                              # list all keywords
lens index "distributed systems" --json          # show entries for keyword
lens index add "distributed systems" note_01ABC --json  # register entry
lens index remove "distributed systems" --json  # remove keyword
lens index remove "distributed systems" note_01ABC --json # remove single entry
```

List output: `{"count": N, "keywords": {"kw": [{"id": "...", "title": "..."}]}}`
Show output: `{"keyword": "...", "entries": [{"id": "...", "title": "..."}]}`
Add output: `{"action": "added", "keyword": "...", "id": "...", "title": "...", "entry_count": N}`

**When to create an entry**: After you notice a cluster of 5+ interconnected notes on a topic — pick the best "starting point" note. The index should be extremely sparse (10-20 keywords for hundreds of notes).

**Usage in Compile/Query mode**: Before searching, check `lens index --json` for relevant keywords. If found, start from that entry point instead of free search — it leads to the best-connected part of the graph.

---

## Write API Reference

Pass JSON via `--stdin` (recommended) or `--file`. The `type` field routes:

```json
{"type": "note", "title": "...", "links": [{"to": "note_ID", "rel": "supports", "reason": "..."}], "body": "..."}
{"type": "source", "title": "...", "url": "...", "source_type": "web_article"}
{"type": "task", "title": "...", "status": "open"}
{"type": "link", "from": "note_A", "rel": "supports", "to": "note_B", "reason": "..."}
{"type": "unlink", "from": "note_A", "rel": "supports", "to": "note_B"}
{"type": "retype", "from": "note_A", "to": "note_B", "old_rel": "related", "new_rel": "supports", "reason": "..."}
{"type": "merge", "from": "note_B", "into": "note_A"}
{"type": "update", "id": "note_A", "set": {"title": "...", "body": "..."}, "add": {"links": [...]}}
{"type": "delete", "id": "note_A"}
[{...}, {...}]  // batch — $0/$1 reference earlier items' IDs
```

Link types: supports, contradicts (auto-bidirectional), refines, related, indexes (MOC → child). **`related` requires a `reason` field** — the CLI rejects related links without one. **`indexes` is exempt from reason requirements** (structural link). Prefer precise rels (contradicts → refines → supports) over related.

**Links are idempotent.** Writing the same link twice returns `"action": "unchanged"`. Writing with a different reason returns `"action": "updated"`. No duplicates are ever created.

**`retype` inherits reason.** When retyping without specifying a new reason, the existing reason is carried forward — no data loss on rel changes.

**Batch writes are partial-success.** If one item in a batch fails, the rest still process. Failed items and their dependents get `"action": "error"` with a message. The envelope returns `ok: false` with `error.code: "partial_failure"` and `data.results` containing per-item results. Link/unlink results include `from`, `to`, `rel` fields.

### Batch: creating notes with links (recommended pattern)

**Use inline `links[]` on notes** to create notes and links in a single batch. This is simpler than separate `link` items because the note already knows its own ID:

```json
[
  {"type": "note", "title": "First insight", "body": "..."},
  {"type": "note", "title": "Second insight", "body": "...", "links": [{"to": "$0", "rel": "supports", "reason": "builds on first"}]},
  {"type": "note", "title": "Third insight", "body": "...", "links": [{"to": "$0", "rel": "refines"}, {"to": "$1", "rel": "related", "reason": "both explore the same design tradeoff"}]}
]
```

`$0`, `$1`, `$2` refer to IDs of earlier items in the same batch (by index). Use inline `links[]` for forward links from a new note. Use a separate `{"type": "link", "from": "$1", ...}` item only when adding links between two already-existing notes.

### Resolving title → ID before writing

To check if a note already exists by exact title, use `--resolve`:

```bash
lens search "exact note title" --resolve --json
```

This does case-insensitive exact title matching first, then falls back to FTS5. Returns `{ok: true, data: {id, title}}` on unique match, or `{ok: false, error: {code: "ambiguous_match"}, candidates: [...]}` if multiple matches. Use this before writing to avoid duplicates.

**Body is free-form markdown.** Evidence, confidence, scope, perspective — all go in body, not frontmatter. Supports standard markdown including code blocks (Mermaid, etc.); rendering depends on the viewer.

**Inline references in body**: Use `[[note_ID]]`, `[[src_ID]]`, or `[[task_ID]]` to reference other objects. Same ID format as `links[].to`. On read, `lens show --json` returns body unchanged plus `body_refs: [{id, title}]` with resolved titles.

For full field reference, read [note-fields.md](note-fields.md).

---

## Combining Commands

Common multi-step workflows:

**Compile an article** (Capture → Collide → Crystallize):
1. `lens fetch <url> --save --json` → get source_id
2. `lens search "key concepts" --json` → find related notes
3. `lens show <id> --json` → read related notes, follow forward_links
4. `lens write --file batch.json --json` → create notes with links
5. Check if any existing structure note should index these new notes → update it with `indexes` links

**Explore a topic** (Index → Walk):
1. `lens index "topic" --json` → get entry point note
2. `lens show <entry_id> --json` → read it, see forward_links
3. `lens links <id> --json` → see all connections (forward + backward)
4. Repeat show → links for each interesting connection

**Dedup after batch import**:
1. `lens similar --all --threshold 0.8 --json` → find duplicate groups
2. For each group: `lens show <id1> <id2> --json` → compare content
3. `lens write '{"type":"merge","from":"dup_id","into":"keep_id"}' --json` → atomic merge (redirects links, appends body, rewrites [[ID]] refs)

**Link quality audit**:
1. `lens links <id> --rel related --json` → see all related links
2. `lens show <target1> <target2> --json` → compare targets
3. `lens write '{"type":"retype","from":"...","to":"...","old_rel":"related","new_rel":"supports","reason":"..."}' --json` → upgrade link type

**Review recent work**:
1. `lens digest week --json` → see tensions/connected/seeds
2. For tensions: `lens show <id> --json` → investigate contradictions
3. For seeds: `lens search "keywords" --json` → find connections
4. `lens write '{"type":"link",...}' --json` → connect orphans

---

## Common Pitfalls

1. **Link field naming varies by command.** `show` returns full `forward_links[]` and `backward_links[]` arrays (each item has `id`, `rel`, `reason`, `title`). `links` returns `forward[]` and `backward[]` (filterable with `--rel` and `--direction`; forward links include `reason`). `search`, `list`, and `digest` return `links` as a compact count or summary — use `show` or `links` to get full link details.

2. **Never truncate note IDs.** IDs are exactly `prefix_` + 26 uppercase chars (ULID). `note_01KP2SFME1Z07MX` is too short and will be rejected. Always copy the full ID.

3. **`--stdin` is always JSON mode.** Do not add `--json` flag with `--stdin` — it's redundant (not harmful, but unnecessary).

4. **Curly quotes break JSON.** `"word"` (U+201C/U+201D) is not valid JSON. Use straight quotes `"word"`. When writing JSON files with special characters, use the Write tool — do not construct JSON strings by hand.

5. **All JSON output uses the envelope.** Success: `{"ok": true, "data": {...}}`. Errors: `{"ok": false, "error": {"code": "...", "message": "..."}}`. Always check `ok` before reading `data`.

## Errors

All errors follow the envelope: `{"ok": false, "error": {"code": "...", "message": "..."}, "hint": "...", "command": "..."}`

Error codes: `command_error` (general), `deprecated_command`, `unknown_command`, `ambiguous_match`, `no_match`, `partial_failure` (batch), `invalid_request`, `empty_stdin`, `no_input`.
