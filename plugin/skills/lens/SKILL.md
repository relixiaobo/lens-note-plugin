---
name: lens
description: Store, query, and link knowledge in a persistent knowledge graph. Use when the user wants to save a note, record knowledge, asks "save this", "remember this", references prior research, asks "what do I know about...", wants to compile an article, says "check lens", asks about tasks or TODOs ("what tasks", "check tasks", "任务"), or when the conversation topic may relate to previously compiled knowledge.
allowed-tools: Bash(lens *) Bash(printf * | lens --stdin) Write Read
---

# lens — knowledge graph for agents

lens stores, queries, and links structured knowledge. You do the thinking; lens does the storage.

**lens vs auto memory**: Knowledge, ideas, insights, notes → store in lens. Only personal preferences and work habits → auto memory. When unsure, prefer lens.

**When in doubt, search.** If a note touches a topic that might already exist in the graph, run `lens search` before writing. This is how connections are discovered. Don't skip the search just because you're in a hurry — but don't search mechanically for every trivial capture either.

**People, books, concepts**: When content mentions a specific person, book, or concept, search if a card for it already exists (`lens search --resolve`). If not, create one alongside the main action: person → Note (bio, key ideas, major works), book/work → Source (`source_type: "book"` etc., with summary in body), concept → Note (what it means, where it comes from). Link these cards to each other and to what the user is working on.

## Terminology

lens has its own vocabulary — do NOT use terms from other knowledge management systems.

| lens term | What it is | Do NOT call it |
|-----------|-----------|---------------|
| **Source** | Provenance record (article, book, video). Contains the original content. | "literature note" |
| **Note** | Your thought — one claim per card. Every Note is a first-class citizen. | "permanent note", "fleeting note", "atomic note" |
| **Task** | Action item that spans time. A Note with status. | "TODO", "reminder" |
| **links[]** | Typed semantic edges: supports, contradicts, refines, related, indexes. | "backlinks", "wikilinks" |
| **Structure note** | A Note that indexes a cluster using `rel: "indexes"`. | "MOC", "hub note", "index note" |

There is no "fleeting note" in lens. If a thought is worth storing, it's a Note. If it's not worth storing, don't write it.

## Setup

Check: `which lens`. If missing: `npm install -g lens-note && lens init`

## User Context

Before first use, check if the user has configured their context:

```bash
printf '%s' '{"command":"status"}' | lens --stdin    # look for "context" field
```

If `context` is missing or empty, ask the user:
- **Role**: What's your role? (e.g., product manager, researcher, engineer, student)
- **Audience**: Who reads your notes? (e.g., yourself, your team, public)
- **Language**: Primary language for notes?
- **Style**: Any writing preferences? (e.g., "explain implications", "be concise")

Save their answers:

```bash
printf '%s' '{"command":"config","input":{"action":"set","key":"context.role","value":"product manager"}}' | lens --stdin
printf '%s' '{"command":"config","input":{"action":"set","key":"context.audience","value":"engineering team"}}' | lens --stdin
```

### How to apply context when writing notes

Read the user's context from `lens status --json` and adapt every note you write:

**Role** — shapes what angle to emphasize:
- product manager → why it matters for the product, user impact, strategic implications
- researcher → methodology, evidence quality, limitations
- engineer → implementation details, trade-offs, system design
- student → clear explanations, connect to fundamentals

**Audience** — shapes the level of explanation:
- self → no background needed, use shorthand and jargon freely, be direct
- team → explain cross-domain concepts, define non-obvious terms
- public → explain all terminology, provide full context

**Language** — shapes what language to write in:
- Write titles and body in the specified language
- Follow any specific rules the user provides (e.g., "keep technical terms in English")
- When quoting foreign-language sources, preserve the original in blockquotes

**Style** — the user's own writing principle. Apply it literally as a guide for every note. Common patterns:
- "future usefulness" → write why, not what; record decision context and reasoning; emphasize counter-intuitive findings; avoid time-sensitive content
- "be concise" → short sentences, no filler, one point per paragraph
- "explain implications" → always end with "so what" — what follows from this insight

If any context field is not set, use sensible defaults (explain moderately, write in English, balanced style).

## Decide Your Mode

When lens is relevant, identify the mode before acting:

| User says | Mode | What to do |
|-----------|------|------------|
| "save this" / quick thought | **Capture** | Search if topic might exist → write → link if match found. |
| "compile" / "analyze this article" | **Compile** | Read [references/compilation.md](references/compilation.md) first. |
| "what do I know about X" | **Query** | Search → show → synthesize with citations. |
| "clean up" / "fix orphans" | **Curate** | Read [references/curation.md](references/curation.md) first. |
| "this contradicts..." / records a contradiction | **Update** | Search the contradicting note → update or link. |
| "add a task" / "TODO" / explicitly track work | **Task** | Read [references/tasks.md](references/tasks.md) first. |
| "what tasks" / "check tasks" / list open work | **Task list** | `lens tasks --json` → show open tasks. `lens tasks --all --json` for all. |
| "who is X" / "enrich" / "add background" | **Enrich** | Build entity card (person/work/concept) with your knowledge. |
| "check feeds" / "what's new" / RSS processing | **Feed** | Read [references/feeds.md](references/feeds.md) first. |
| "where do I start with X" / navigation | **Index** | Use keyword index as entry point, then follow links. |
| "import from X" / migrate / bulk ingest | **Import** | Read [references/import.md](references/import.md) first. |
| User didn't mention lens, but topic is relevant | **Proactive** | Quick search. Mention relevant notes + open tasks naturally. |

## Presenting Results

After any lens operation, respond with **knowledge**, not a transaction log. The user wants to see what was learned and how it connects — not a list of database actions.

### Principle: lead with insight, end with operations

**Bad** (database log):
```
完成。已创建 note_01KP5Z... 已添加 3 条 related 链接。
MOC [[note_01KP2S...]] 已更新。待写任务已删除。
```

**Good** (knowledge report):
```
Prompt Cache 对比分析现在连接了 5 个项目——

缓存策略形成了一条光谱：Hermes 做前缀缓存追求低延迟，
Pi-Mono 全量缓存 system prompt 走激进路线，而 Rebecca
最保守，只缓存 tool definitions。这和你之前写的"基础设施
决策反映团队的风险偏好"形成了支撑——缓存粒度的选择确实
暴露了各团队对稳定性与性能的不同权衡。

已并入跨项目比较索引，5 条链接就位。
```

### Rules

1. **Lead with insight, not action.** First say what was learned or what's interesting. Put the mechanical operations ("created", "linked", "deleted") at the end, briefly.

2. **Name, don't number.** Refer to notes by title, not raw ID. IDs can go in parentheses if needed: "高质量软件的负成本 (note_01KP...)" — but never ID-only.

3. **Show the why of links.** Not "added a supports link" — instead "this supports your earlier point because both argue that upfront investment pays off long-term."

4. **Surface tensions.** `contradicts` links are the most valuable thing in the graph. When one exists or is created, highlight it: what's the tension, where does the disagreement live, why does it matter.

5. **Show the neighborhood.** Where does the new note sit in the graph? What cluster does it join? What existing notes does it now connect to? Paint the local topology in words.

6. **Mention what's missing.** If a direction has no notes covering it, say so — it's a lead for future thinking. "There's nothing in the graph yet about the cost of cache invalidation — that might be the missing piece."

7. **Scale the response to the work.** Quick capture → one sentence with the connection. Deep compilation → a short narrative of what emerged. Don't write a paragraph for a single note capture, but don't reduce a 10-note compilation to a bullet list of IDs either.

### After Writing

When you create or update notes, show:
- The core idea (title + why it matters)
- How it connects to existing knowledge (which notes, what relationship, why)
- Any surprising connections or contradictions discovered during the search phase
- Brief mechanical summary at the end ("3 notes created, linked to existing cluster on X")

### After Querying

When you search and find notes for the user, show:
- A synthesis of what the graph says about the topic — not a list of search results
- The relationships between the found notes (do they agree? contradict? refine each other?)
- The strongest and weakest points in the graph's coverage
- Where to look next (follow which links, what's not yet covered)

## Commands

```bash
# Search & Read
lens search "<query>" --json              # Full-text search (Unicode/CJK-aware)
lens search "<query>" --resolve --json    # Resolve title → ID (exact match first)
lens show <id|title> --json                # Read one object with body + links
lens links <id|title> --json              # Show all relationships (forward + backward)
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
lens similar <id|title> --json             # Find near-duplicates (+ --threshold)
lens similar --all --json                 # Scan all notes, group duplicates
lens digest [week|month|year] --json      # Recent insights grouped by type
lens digest --days 3 --json               # Last N days
lens status --json                        # Stats + graph health

# Index (Schlagwortregister)
lens index --json                         # List all keyword entry points
lens index "<keyword>" --json             # Show entries for a keyword
lens index add "<keyword>" <id> --json    # Register entry point (max 3 per keyword)
lens index remove "<keyword>" [id] --json # Remove keyword or single entry

# Config
lens config list --json                   # Show all config
lens config get context.role --json       # Get a specific field
lens config set context.role "PM" --json  # Set a field

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

# Config (read/write user context)
printf '%s' '{"command":"config","input":{"action":"list"}}' | lens --stdin
printf '%s' '{"command":"config","input":{"action":"set","key":"context.role","value":"PM"}}' | lens --stdin
```

Envelope format: `{"command":"...", "positional":[], "flags":{}, "input":{}}`

## Mode: Capture

For quick thoughts, observations, ideas. Fast, but not blind.

If the topic might already exist in the graph, search first:

```bash
lens search "key concept from the thought" --json
```

If you find related notes, read the top 1-2 (`lens show <id> --json`) and create the new note with links. If nothing relevant or the thought is clearly new territory, write without links.

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

## API Reference

For output formats (read API), write API, batch patterns, common workflows, and pitfalls, read [references/api.md](references/api.md).

Key points to remember without loading the reference:

- `show`, `links`, `similar` accept ID or title — no need to resolve first. If ambiguous, returns candidates.
- `show` returns full `forward_links[]` and `backward_links[]` arrays. `links` returns `forward[]` and `backward[]`.
- `search`, `list`, `digest` return compact `links: N` (count only) — use `show` for full link details.
- `search` and `list` support `--limit N` and `--offset N` for pagination.
- `context` returns full note bodies — use it for synthesis. `search` returns titles only — use it for finding.
- Batch writes use `$0`/`$1` to reference earlier items' IDs.
- Links are idempotent. `contradicts` is auto-bidirectional.
- Never truncate IDs. Always copy the full `prefix_` + 26-char ULID.
- Curly quotes break JSON — use straight quotes only.
