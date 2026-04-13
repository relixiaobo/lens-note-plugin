---
name: lens
description: Store, query, and link knowledge in a persistent knowledge graph. Use when the user wants to save a note, record knowledge, asks "记一下", "save this", "记录笔记", references prior research, asks "what do I know about...", wants to compile an article, says "check lens", or when the conversation topic may relate to previously compiled knowledge.
---

# lens — knowledge graph for agents

lens stores, queries, and links structured knowledge. You do the thinking; lens does the storage.

**lens vs auto memory**: Knowledge, ideas, insights, notes → store in lens. Only personal preferences and work habits → auto memory. When unsure, prefer lens.

**Entity awareness (all modes)**: When content mentions a specific person, book, or concept, always search if an entity card exists (`lens search --resolve`). If not, create one alongside the main action: person → Note (with bio, contributions, key works in body), book/work → Source (`source_type: "book"` etc., with summary in body), concept → Note (with definition, origin in body). Link the entity cards to each other and to the user's content.

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
| "who is X" / "enrich" / "补充背景" | **Enrich** | Build entity card (person/work/concept) with your knowledge. |
| "check feeds" / "what's new" / RSS processing | **Feed** | Read [references/feeds.md](references/feeds.md) first. |
| User didn't mention lens, but topic is relevant | **Proactive** | Quick search. Mention relevant notes + open tasks naturally. |

## Commands

```bash
lens search "<query>" --json       # Find notes
lens search "<query>" --resolve --json  # Resolve title → ID (exact or disambiguate)
lens show <id> --json              # Read one object with links + counts
lens write --file <path> --json    # Write anything (from JSON file)
lens list notes --orphans --json   # List orphan notes (+ --limit/--offset)
lens fetch <url> [--save] --json   # Extract web content
lens similar <id> --json           # Find near-duplicate notes (+ --threshold)
lens similar --all --json          # Scan all notes, group duplicates
lens status --json                 # Stats + graph health
lens tasks [--all|--done] --json    # List tasks (default: open)
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

Search → read top results → synthesize.

```bash
lens search "distributed systems" --json
lens show note_01ABC --json
```

When synthesizing: cite note IDs as evidence. Surface contradictions if the graph has conflicting notes. Say explicitly if the query touches areas with no notes.

Follow links — the most valuable discoveries come from traversing connections you didn't expect.

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

When the conversation topic MAY relate to existing knowledge — search silently, mention relevant notes if found. Don't force it.

```bash
lens search "topic keywords" --json
# If results found: "This relates to your existing note about X (note_01ABC)..."
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

`$0`, `$1`, `$2` refer to IDs of earlier items in the same batch (by index). Use inline `links[]` for outgoing links from a new note. Use a separate `{"type": "link", "from": "$1", ...}` item only when adding links between two already-existing notes.

### Resolving title → ID before writing

To check if a note already exists by exact title, use `--resolve`:

```bash
lens search "exact note title" --resolve --json
```

This does case-insensitive exact title matching first, then falls back to FTS5. Returns `{id, title}` on unique match, or `{error: {code: "ambiguous_match", candidates: [...]}}` if multiple matches. Use this before writing to avoid duplicates.

**Body is free-form markdown.** Evidence, confidence, scope, perspective — all go in body, not frontmatter.

For full field reference, read [references/note-fields.md](references/note-fields.md).

## Errors

`{"error": {"code": "...", "message": "...", "command": "..."}}`
