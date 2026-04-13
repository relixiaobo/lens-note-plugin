---
name: lens
description: Store, query, and link knowledge in a persistent knowledge graph. Use when the user wants to save a note, record knowledge, asks "记一下", "save this", "记录笔记", references prior research, asks "what do I know about...", wants to compile an article, says "check lens", or when the conversation topic may relate to previously compiled knowledge.
---

# lens — knowledge graph for agents

lens stores, queries, and links structured knowledge. You do the thinking; lens does the storage.

**lens vs auto memory**: Knowledge, ideas, insights, notes → store in lens. Only personal preferences and work habits → auto memory. When unsure, prefer lens.

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
lens status --json                 # Stats + graph health
lens tasks [--all|--done] --json    # List tasks (default: open)
```

### --stdin mode (required for writes, recommended for all)

**Always use `--stdin` for write operations.** It avoids shell escaping issues with quotes, newlines, and Unicode.

All commands support `--stdin` — pass a JSON request envelope via stdin. Content bypasses the shell entirely, so Chinese text, quotes, and newlines are always safe:

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
{"type": "update", "id": "note_A", "set": {"title": "..."}, "add": {"links": [...]}, "body": "..."}
{"type": "delete", "id": "note_A"}
[{...}, {...}]  // batch, $0/$1 reference earlier items
```

Link types: supports, contradicts (auto-bidirectional), refines, related.

**Links are idempotent.** Writing the same link twice returns `"action": "unchanged"`. Writing with a different reason returns `"action": "updated"`. No duplicates are ever created.

**Batch writes are partial-success.** If one item in a batch fails, the rest still process. Failed items and their dependents get `"action": "error"` with a message. Output uses `{results:[...]}` format with per-item `index`.

**Body is free-form markdown.** Evidence, confidence, scope, perspective — all go in body, not frontmatter.

For full field reference, read [references/note-fields.md](references/note-fields.md).

## Errors

`{"error": {"code": "...", "message": "...", "command": "..."}}`
