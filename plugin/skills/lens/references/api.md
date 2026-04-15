# API Reference

Complete reference for lens read and write APIs, output formats, common workflows, and pitfalls.

## Contents

- [Read API — Output Formats](#read-api--output-formats)
- [Write API Reference](#write-api-reference)
- [Combining Commands](#combining-commands)
- [Common Pitfalls](#common-pitfalls)

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
2. **Title match**: Case-insensitive exact title search. Returns `{id, title}` on unique match, or `{error: {code: "ambiguous_match", candidates: [...]}}` if multiple notes share the title.
3. **FTS5 ranked search**: Full-text search. Returns `{id, title}` if exactly one result, `{error: {code: "ambiguous_match", candidates: [...]}}` (top 5) if multiple, or `{error: {code: "no_match"}}` if zero.

### show

Accepts ID or title. If the input is not a valid ID format, auto-resolves by exact title match → FTS5 search. If ambiguous, returns `{error: {code: "ambiguous_match", candidates: [...]}}`.

Returns full object with body and links as top-level arrays:

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
 "forward": [{"id": "note_01B", "rel": "supports", "type": "note", "title": "Target title"}],
 "backward": [{"id": "note_01C", "rel": "refines", "type": "note", "title": "Source title"}]}
```

Only real graph edges (from `links[]` in YAML) appear here. The `source` metadata field is NOT included — use `show` to see a note's source.

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

Flags: `--since 7d` (time filter: `Nd`/`Nw`/`Nm`/`Ny`), `--orphans` (notes only), `--limit N`, `--offset N`.

### context

Assembles a context pack with full note bodies. Use `context` when you need to **synthesize** across multiple notes (e.g., answering "what do I know about X"). Use `search` when you just need to **find** matching notes by title/keyword.

```json
{"query": "...", "timestamp": "...", "total_results": 5,
 "notes": [{"id": "note_01A", "title": "...", "source": "src_01B", "forward_links": [...], "body": "full body...",
            "body_refs": [{"id": "note_01B", "title": "Referenced note"}]}]}
```

`body_refs` is optional — only included when the body contains `[[ID]]` inline references. Always returns JSON (no `--json` flag needed). Only includes notes, not sources or tasks.

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

### status

```json
{"path": "/Users/.../.lens", "notes": 903, "sources": 99,
 "tasks": {"open": 2, "done": 0, "total": 2}, "total": 1004,
 "connectivity": {"orphan_count": 4, "orphan_rate": 0.4, "total_links": 2874, "cross_source_pct": 0},
 "link_types": {"related": 2457, "supports": 279, "refines": 136, "contradicts": 2},
 "context": {"role": "PM", "audience": "engineering team"}}
```

`context` is optional — only included when the user has configured their context via `lens config set context.*`.

### fetch

Extracts web content as clean markdown. With `--save`, also creates a source object.

```json
{"title": "Article Title", "author": "Author Name", "url": "https://...",
 "word_count": 2718, "markdown": "# Article...", "source_id": "src_01A (only with --save)"}
```

### tasks

Shortcut for listing tasks. `tasks` = open only, `tasks --all` = all, `tasks --done` = done only.

```json
{"type": "tasks", "count": 2, "items": [{"id": "task_01A", "title": "...", "status": "open"}]}
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
{"type": "update", "id": "note_A", "set": {"title": "..."}, "add": {"links": [...]}, "body": "..."}
{"type": "delete", "id": "note_A"}
[{...}, {...}]  // batch — $0/$1 reference earlier items' IDs
```

Link types: supports, contradicts (auto-bidirectional), refines, related, indexes (MOC → child).

**Links are idempotent.** Writing the same link twice returns `"action": "unchanged"`. Writing with a different reason returns `"action": "updated"`. No duplicates are ever created.

**Batch writes are partial-success.** If one item in a batch fails, the rest still process. Failed items and their dependents get `"action": "error"` with a message. Output uses `{results:[...]}` format with per-item `index`. Link/unlink results include `from`, `to`, `rel` fields.

### Batch: creating notes with links (recommended pattern)

**Use inline `links[]` on notes** to create notes and links in a single batch. This is simpler than separate `link` items because the note already knows its own ID:

```json
[
  {"type": "note", "title": "First insight", "body": "..."},
  {"type": "note", "title": "Second insight", "body": "...", "links": [{"to": "$0", "rel": "supports", "reason": "builds on first"}]},
  {"type": "note", "title": "Third insight", "body": "...", "links": [{"to": "$0", "rel": "refines"}, {"to": "$1", "rel": "related"}]}
]
```

`$0`, `$1`, `$2` refer to IDs of earlier items in the same batch (by index). Use inline `links[]` for forward links from a new note. Use a separate `{"type": "link", "from": "$1", ...}` item only when adding links between two already-existing notes.

### Resolving title → ID before writing

To check if a note already exists by exact title, use `--resolve`:

```bash
lens search "exact note title" --resolve --json
```

This does case-insensitive exact title matching first, then falls back to FTS5. Returns `{id, title}` on unique match, or `{error: {code: "ambiguous_match", candidates: [...]}}` if multiple matches. Use this before writing to avoid duplicates.

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
2. For each group: `lens show <id> --json` → compare content
3. `lens write '{"type":"update","id":"keep_id","body":"merged"}' --json` → merge
4. `lens write '{"type":"delete","id":"dup_id"}' --json` → remove duplicate

**Review recent work**:
1. `lens digest week --json` → see tensions/connected/seeds
2. For tensions: `lens show <id> --json` → investigate contradictions
3. For seeds: `lens search "keywords" --json` → find connections
4. `lens write '{"type":"link",...}' --json` → connect orphans

---

## Common Pitfalls

1. **Link field naming varies by command.** `show` returns full `forward_links[]` and `backward_links[]` arrays (each item has `id`, `rel`, `title`). `links` returns `forward[]` and `backward[]`. `search`, `list`, and `digest` return `links` as a compact count or summary — use `show` to get full link details.

2. **Never truncate note IDs.** IDs are exactly `prefix_` + 26 uppercase chars (ULID). `note_01KP2SFME1Z07MX` is too short and will be rejected. Always copy the full ID.

3. **`--stdin` is always JSON mode.** Do not add `--json` flag with `--stdin` — it's redundant (not harmful, but unnecessary).

4. **Curly quotes break JSON.** `"word"` (U+201C/U+201D) is not valid JSON. Use straight quotes `"word"`. When writing JSON files with special characters, use the Write tool — do not construct JSON strings by hand.

5. **Error responses are JSON too.** When a command fails, stdout is `{"error": {"code": "...", "message": "..."}}`. Always check exit code or parse the response before assuming success.

## Errors

`{"error": {"code": "...", "message": "...", "command": "..."}}`
