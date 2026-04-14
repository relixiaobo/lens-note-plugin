---
name: lens
description: Store, query, and link knowledge in a persistent knowledge graph. Use when the user wants to save a note, record knowledge, asks "save this", "remember this", references prior research, asks "what do I know about...", wants to compile an article, says "check lens", or when the conversation topic may relate to previously compiled knowledge.
---

# lens — knowledge graph for agents

lens stores, queries, and links structured knowledge. You do the thinking; lens does the storage.

**lens vs auto memory**: Knowledge, ideas, insights, notes → store in lens. Only personal preferences and work habits → auto memory. When unsure, prefer lens.

**People, books, concepts**: When content mentions a specific person, book, or concept, search if a card for it already exists (`lens search --resolve`). If not, create one alongside the main action: person → Note (bio, key ideas, major works), book/work → Source (`source_type: "book"` etc., with summary in body), concept → Note (what it means, where it comes from). Link these cards to each other and to what the user is working on.

## Setup

Check: `which lens`. If missing: `npm install -g lens-note && lens init`

## Decide Your Mode

When lens is relevant, identify the mode before acting:

| User says | Mode | What to do |
|-----------|------|------------|
| "save this" / quick thought | **Capture** | Write immediately. No search needed. |
| "compile" / "analyze this article" | **Compile** | Read [references/compilation.md](references/compilation.md) first. |
| "what do I know about X" | **Query** | Search → show → synthesize with citations. |
| "clean up" / "fix orphans" | **Curate** | Read [references/curation.md](references/curation.md) first. |
| "this contradicts..." / records a contradiction | **Update** | Search the contradicting note → update or link. |
| "add a task" / "TODO" / explicitly track work | **Task** | Read [references/tasks.md](references/tasks.md) first. |
| "who is X" / "enrich" / "add background" | **Enrich** | Build entity card (person/work/concept) with your knowledge. |
| "check feeds" / "what's new" / RSS processing | **Feed** | Read [references/feeds.md](references/feeds.md) first. |
| "where do I start with X" / navigation | **Index** | Use keyword index as entry point, then follow links. |
| "import from X" / migrate / bulk ingest | **Import** | Read [references/import.md](references/import.md) first. |
| User didn't mention lens, but topic is relevant | **Proactive** | Quick search. Mention relevant notes + open tasks naturally. |

## Commands

```bash
# Search & Read
lens search "<query>" --json              # Full-text search (Unicode/CJK-aware)
lens search "<query>" --resolve --json    # Resolve title → ID (exact match first)
lens show <id> --json                     # Read one object with body + links
lens links <id> --json                    # Show all relationships (forward + backward)
lens context "<query>" --json             # Assemble full context pack for a topic

# Write
lens write --file <path> --json           # Write note/source/task/link/unlink/update/delete/batch
lens fetch <url> [--save] --json          # Extract web content (--save to create source)

# Browse
lens list notes --json                    # List all notes
lens list notes --orphans --json          # List unlinked notes (+ --limit/--offset)
lens list notes --since 7d --json         # List notes from last 7 days (7d/2w/1m/1y)
lens list sources --json                  # List all sources
lens tasks [--all|--done] --json          # List tasks (default: open only)

# Analyze
lens similar <id> --json                  # Find near-duplicates (+ --threshold)
lens similar --all --json                 # Scan all notes, group duplicates
lens digest [week|month|year] --json      # Recent insights grouped by type
lens digest --days 3 --json               # Last N days
lens status --json                        # Stats + graph health

# Index (Schlagwortregister)
lens index --json                         # List all keyword entry points
lens index "<keyword>" --json             # Show entries for a keyword
lens index add "<keyword>" <id> --json    # Register entry point (max 3 per keyword)
lens index remove "<keyword>" [id] --json # Remove keyword or single entry

# System
lens rebuild-index --json                 # Rebuild SQLite cache from markdown files
```

### --stdin vs --file

Two ways to pass JSON input to lens. Choose based on content:

| Method | When to use | Pros |
|--------|-------------|------|
| `--stdin` | Agent envelope protocol, simple commands | Single pipe, no temp file |
| `--file` | Batch writes, content with special chars (Chinese, curly quotes, newlines) | Encoding-safe, debuggable |

**For content with Chinese or special characters**, prefer `--file`: write JSON to a temp file first, then `lens write --file <path> --json`. This avoids shell escaping issues entirely.

**`--stdin` envelope format** — all commands via stdin bypass shell escaping:

```bash
# Write (content goes in "input", never through shell)
printf '%s' '{"command":"write","input":{"type":"note","title":"My insight","body":"Details..."}}' | lens --stdin

# Search
printf '%s' '{"command":"search","positional":["query text"]}' | lens --stdin

# Fetch with flags
printf '%s' '{"command":"fetch","positional":["https://..."],"flags":{"save":true}}' | lens --stdin
```

Envelope format: `{"command":"...", "positional":[], "flags":{}, "input":{}}`

## Mode: Capture

For quick thoughts, observations, ideas. No overhead.

```bash
printf '%s' '{"command":"write","input":{"type":"note","title":"Simple tools composed together beat complex frameworks"}}' | lens --stdin
```

One rule: **one idea per note.** If the thought has multiple claims, split into separate notes.

### Dedup check with `lens similar`

After creating a note, check for near-duplicates:

```bash
lens similar <id> --json                # default threshold: 0.3
lens similar <id> --threshold 0.5 --json  # stricter matching
```

To scan all notes at once and group duplicates:

```bash
lens similar --all --json               # all groups above 0.3
lens similar --all --threshold 0.8 --json  # only high-confidence duplicates
```

- Only works on notes (not sources or tasks)
- Uses character trigrams + Dice coefficient — language-agnostic (works for CJK, Latin, etc.)
- `--threshold`: 0–1 float. Default 0.3 catches loose duplicates; 0.5+ for stricter matching
- Single-note output: `{"id": "...", "count": N, "results": [{"id": "...", "title": "...", "similarity": 0.65}, ...]}`
- `--all` output: `{"count": N, "groups": [{"notes": [...], "pairs": [{"a": "...", "b": "...", "similarity": 0.9}]}]}`

If a near-duplicate is found (similarity > 0.5), consider merging instead of keeping both: update the existing note with new evidence and delete the new one.

## Mode: Query

Search → read top results → tell the user what you found, in plain language.

```bash
lens search "distributed systems" --json
lens show note_01ABC --json
```

Write like a friend who has read all the notes and is giving a straight answer — not a summary report. Say what the user actually thinks (cite note IDs), call out contradictions directly, and admit if a topic has no notes yet.

Follow links — the most valuable connections are the ones you didn't go looking for.

## Mode: Update

When new information changes existing knowledge:

```bash
# Update body with new evidence
printf '%s' '{"command":"write","input":{"type":"update","id":"note_01ABC","body":"Updated body with new evidence:\n\n> new quote — Source\n\n**certain** — confirmed by 3 sources."}}' | lens --stdin

# Add a link
printf '%s' '{"command":"write","input":{"type":"link","from":"note_01ABC","rel":"supports","to":"note_01DEF","reason":"both argue for the same principle"}}' | lens --stdin

# Record contradiction
printf '%s' '{"command":"write","input":{"type":"link","from":"note_NEW","rel":"contradicts","to":"note_01ABC","reason":"AI changes the cost equation"}}' | lens --stdin
```

## Mode: Proactive

When the conversation topic might connect to something in the graph — search quietly, mention it if it's actually relevant. Don't force it.

```bash
lens search "topic keywords" --json
# If results found: mention naturally — "you have a note on this: X (note_01ABC)"
# If no results: say nothing. Don't mention lens.
```

## Mode: Compile

Deep processing using the **Collision Method**: Spark → Collide → Crystallize.

**Read [references/compilation.md](references/compilation.md) before proceeding** — it contains the full methodology.

Quick summary: fetch content → carry your thoughts into the knowledge graph → wander through existing notes following links → crystallize what emerges from the collision. Update existing notes when possible. Zero new notes is acceptable.

## Mode: Enrich

Build entity cards for people, works, or concepts using your own knowledge.

**People** → Note. Body structure:
```markdown
## Basic Info
- Identity, field, era

## Core Ideas
- Key theories, contributions

## Major Works
- Titles with years
```

**Works** (books, papers, talks) → Source with `source_type: "book"` / `"paper"` / `"video"` etc. Body: summary + key arguments.

**Concepts** (recurring theories/frameworks) → Note. Body: definition, origin, related concepts.

Workflow:
1. `lens search "entity name" --resolve --json` — check if exists
2. If exists → update with new info. If not → create.
3. Link to related notes: person ↔ works ↔ ideas.

## Entity Extraction (during Compile)

While compiling content, automatically detect mentions of people, works, and concepts. For each:
1. Search if the entity already has a card in lens
2. If not, create one (person → Note, work → Source, concept → Note)
3. Link the insight notes to the entity cards

This happens naturally during the Collide step — you're already searching the graph. Extend the search to include entity names.

## Mode: Curate

Maintain graph health. **Read [references/curation.md](references/curation.md) before proceeding.**

Quick summary: check orphan count → find connections for unlinked notes → only add links you can justify.

## Read API — Output Formats

### search

```json
{"query": "...", "count": 3, "results": [
  {"id": "note_01A", "type": "note", "title": "...", "forward_links": [...]},
  {"id": "src_01B", "type": "source", "title": "...", "source_type": "web_article", "word_count": 2718},
  {"id": "task_01C", "type": "task", "title": "...", "status": "open"}
]}
```

`--resolve` returns `{id, title}` on unique match, or `{error: {code: "ambiguous_match", candidates: [...]}}`.

### show

Returns full object with body and links as top-level arrays:

```json
{"id": "note_01A", "type": "note", "title": "...", "body": "...",
 "forward_links": [{"id": "note_01B", "rel": "supports", "reason": "...", "title": "Target title"}],
 "backward_links": [{"id": "note_01C", "rel": "refines", "title": "Source title"}]}
```

`forward_links` and `backward_links` are arrays at the top level. Each link item has `id`, `rel`, `title`, and optionally `reason`.

### links

Shows all relationships for an object:

```json
{"id": "note_01A",
 "forward": [{"id": "note_01B", "rel": "supports", "type": "note", "title": "Target title"}],
 "backward": [{"id": "note_01C", "rel": "refines", "type": "note", "title": "Source title"}]}
```

Use `links` to explore the graph — follow connections to discover related knowledge.

### list

```json
{"type": "notes", "count": 42, "items": [{"id": "note_01A", "title": "..."}]}
```

With `--orphans`: `{"type": "notes", "filter": "orphans", "count": 5, "items": [{"id": "...", "title": "...", "preview": "..."}]}`

Flags: `--since 7d` (time filter: `Nd`/`Nw`/`Nm`/`Ny`), `--orphans` (notes only), `--limit N`, `--offset N` (pagination for orphans).

### context

Assembles a context pack with full note bodies. Use `context` when you need to **synthesize** across multiple notes (e.g., answering "what do I know about X"). Use `search` when you just need to **find** matching notes by title/keyword.

```json
{"query": "...", "timestamp": "...", "total_results": 5,
 "notes": [{"id": "note_01A", "title": "...", "source": "src_01B", "forward_links": [...], "body": "full body..."}]}
```

Always returns JSON (no `--json` flag needed). Only includes notes, not sources or tasks.

### digest

Groups recent notes by link type. Categorization: has `contradicts` link → **tensions**, has other links → **connected**, no links → **seeds**.

```json
{"period": "7d", "total": 12,
 "tensions": [{"id": "note_01A", "title": "...", "forward_links": [{"rel": "contradicts", "to": "note_01B"}]}],
 "connected": [{"id": "note_01C", "title": "...", "forward_links": [...]}],
 "seeds": [{"id": "note_01D", "title": "..."}]}
```

Accepts `week`/`month`/`year` or `--days N` (default: 1 day). Use to review recent work and spot contradictions worth exploring.

### status

```json
{"path": "/Users/.../.lens", "notes": 903, "sources": 99,
 "tasks": {"open": 2, "done": 0, "total": 2}, "total": 1004,
 "connectivity": {"orphan_count": 4, "orphan_rate": 0.4, "total_links": 2874, "cross_source_pct": 0},
 "link_types": {"related": 2457, "supports": 279, "refines": 136, "contradicts": 2}}
```

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

Link types: supports, contradicts (auto-bidirectional), refines, related.

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

**Body is free-form markdown.** Evidence, confidence, scope, perspective — all go in body, not frontmatter.

For full field reference, read [references/note-fields.md](references/note-fields.md).

## Combining Commands

Common multi-step workflows:

**Compile an article** (Capture → Collide → Crystallize):
1. `lens fetch <url> --save --json` → get source_id
2. `lens search "key concepts" --json` → find related notes
3. `lens show <id> --json` → read related notes, follow forward_links
4. `lens write --file batch.json --json` → create notes with links

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

## Common Pitfalls

1. **Link field naming is consistent.** `show` returns `forward_links[]` and `backward_links[]` (top-level arrays). `links` returns `forward[]` and `backward[]`. `search` and `list` return `forward_links[]`. Each link item uses `id`, `rel`, `title`.

2. **Never truncate note IDs.** IDs are exactly `prefix_` + 26 uppercase chars (ULID). `note_01KP2SFME1Z07MX` is too short and will be rejected. Always copy the full ID.

3. **`--stdin` is always JSON mode.** Do not add `--json` flag with `--stdin` — it's redundant (not harmful, but unnecessary).

4. **Curly quotes break JSON.** `"word"` (U+201C/U+201D) is not valid JSON. Use straight quotes `"word"`. When writing JSON files with special characters, use the Write tool — do not construct JSON strings by hand.

5. **Error responses are JSON too.** When a command fails, stdout is `{"error": {"code": "...", "message": "..."}}`. Always check exit code or parse the response before assuming success.

## Errors

`{"error": {"code": "...", "message": "...", "command": "..."}}`
