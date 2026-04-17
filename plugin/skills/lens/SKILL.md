---
name: lens
description: Store, query, and link knowledge in a persistent knowledge graph. Use when the user wants to save a note, record knowledge, asks "save this", "remember this", references prior research, asks "what do I know about...", wants to compile an article, says "check lens", asks about tasks or TODOs ("what tasks", "check tasks", "任务"), or when the conversation topic may relate to previously compiled knowledge.
allowed-tools: Bash(lens *) Bash(printf * | lens --stdin) Write Read
---

# lens — knowledge graph for agents

lens stores, queries, and links structured knowledge. You do the thinking; lens does the storage.

**lens vs auto memory**: Knowledge, ideas, insights, notes → store in lens. Only personal preferences and work habits → auto memory. When unsure, prefer lens.

**When in doubt, search.** If a note touches a topic that might already exist in the graph, run `lens search` before writing. This is how connections are discovered. Don't skip the search just because you're in a hurry — but don't search mechanically for every trivial capture either.

**Before creating any synthesis node** (L2, thesis, or structure note), search the graph for existing nodes covering the same sub-focus, including disconnected ones. Use `lens search --expand` AND `lens list notes --max-links 2` — thesis nodes with no inbound `supports` are invisible to ordinary keyword search. See [references/compilation.md](references/compilation.md) Cluster Check Step 0.

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

### Note title discipline

Two kinds of notes require different title patterns:

| Kind | When to use | Title pattern |
|------|-------------|---------------|
| **Observation** | Specific finding from one source/case | `[Source]: [what they do/say]` — e.g., "Manus sub-agent design: context isolation over role simulation" |
| **Thesis** | Generalized claim you believe holds broadly | Direct assertion — e.g., "Context isolation is the core constraint in sub-agent design" |

**Never promote an observation to a thesis in the title.** If Manus uses context isolation, the title should say "Manus uses X", not "X is the nature of multi-agent systems." Overstated thesis titles attract spurious `supports` links from anything on the same topic.

If an observation has been confirmed by multiple independent sources, you may write a separate thesis note linking back to them with `supports`.

### Choosing link types (rel decision tree)

When linking two notes, work through this order:

1. **A opposes B?** → `contradicts` (most valuable — tensions are where knowledge grows)
2. **A is a concrete version/implementation/case of B?** → `refines`
3. **A strengthens or provides evidence for B?** → `supports`
4. **A indexes/organizes B?** → `indexes`
5. **None of the above?** → `related` (requires a `reason` explaining HOW — topic overlap alone is not enough)

**`related` is the last resort, not the default.** The CLI rejects `related` without a reason. Run `lens lint --json` to check graph health.

### `supports` quality rule

`supports` means: **this note provides specific evidence for the thesis in the target note.** It does NOT mean "both notes are about the same topic."

Before creating a `supports` link, ask: *"How does [source] provide evidence for the specific claim in [target]?"* If the honest answer is "both mention multi-agent systems" or "both are about context", use `related` instead.

**Red flags for a `supports` reason:**
- "Protocol X and System Y architecture" — two topic labels joined with "and", no evidential explanation
- "Article A and concept B" — structural label, not causal
- Reason restates that both notes share a topic, without saying how one provides evidence for the other

**Good `supports` reasons explain the mechanism:**
- "Benchmark shows accuracy drops 37% when context exceeds 32k tokens, directly supporting the 'smaller context = clearer' claim"
- "Case study confirms the thesis: each worker agent gets a clean context window with no shared state between runs"
- "Counter-example: the Room+@mention architecture uses shared state — shows the principle is design-specific, not universal"

### Import link discipline

During bulk ingest (Mode: Import or Compile), **do NOT create `supports` links**. Only create:

- `related` — for notes that share context or topic
- `refines` — when one note is a concrete case or implementation of another

Reserve `supports` for manually verified evidence relationships. Before creating a `supports` link, you must have read both notes and confirmed: the source note provides *specific evidence* for the *thesis* in the target note — not just that both cover the same topic.

**Why this matters:** The most common source of graph quality degradation is bulk-import `supports` generated by topic proximity. A reason like "A与B都关注分布式系统" (A and B both focus on distributed systems) is not evidence — it's a topic label. Every spurious `supports` link degrades the signal value of every real `supports` link in the graph.

The rule of thumb: **if the reason could apply to more than one target note, it's not a `supports` reason — it's a `related` reason.**

`related` can always be upgraded to `supports` after careful review. The reverse (downgrading spurious `supports`) requires a manual audit pass.

### Post-import lint habit

After any bulk import or reading session, always run:

```bash
lens lint --audit vague_reasons --json   # catch supports used as topic labels
lens lint --json                         # check supports_density and super_connectors
```

The `vague_reasons` audit now flags `supports` links whose reasons only describe topic proximity (pattern: "A与B" with no explanatory verb). These are the most common source of spurious hub notes.

After `vague_reasons` reports offenders, do not just bulk-downgrade. For any offender thesis with ≥ 5 incoming offenders, shift to **per-target audit** — see [references/curation.md](references/curation.md) "Per-Target Supports Audit". A single under-examined thesis often collects the majority of an import's bad supports, and a per-target decision tree (keep-with-reason / downgrade / unlink / retype-to-contradicts) gives better outcomes than bulk edit.

## Setup

Check: `which lens`. If missing: `npm install -g lens-note && lens init`

## User Context

Before first use, check if the user has configured their context:

```bash
printf '%s' '{"command":"lint","flags":{"summary":true}}' | lens --stdin    # look for "context" field
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

Read the user's context from `lens lint --summary --json` and adapt every note you write:

**Role** — a description of who the user is, not a single label. People have multiple facets. Read the role as background context, then **infer the right angle from the content itself**:
- Writing about product decisions? Emphasize user impact and strategic implications.
- Writing about technical architecture? Emphasize trade-offs and design rationale.
- Writing about personal experience? Emphasize reflection and transferable insight.
- Writing about a concept or theory? Emphasize clarity and connections to other ideas.

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
| "what tasks" / "check tasks" / list open work | **Task list** | `lens list tasks --status open --json` → show open tasks. `lens list tasks --json` for all. |
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
lens search "<query>" --expand --json     # Search with full bodies + links
lens show <id> [id2...] --json            # Show object(s) with body + links (batch supported)
lens links <id> --json                    # All relationships (forward + backward)
lens links <id> --rel related --json      # Filter by relationship type
lens links <id> --direction forward --json # Only outgoing links

# Write
lens write --file <path> --json           # Write note/source/task/link/unlink/update/delete/retype/merge/batch
lens fetch <url> [--save] --json          # Extract web content (--save to create source)

# Browse
lens list notes --json                    # List all notes
lens list notes --orphans --json          # List unlinked notes (+ --limit/--offset)
lens list notes --since 7d --json         # List notes from last 7 days (7d/2w/1m/1y)
lens list notes --min-links 10 --json     # Hub notes by link count
lens list notes --max-links 2 --json      # Orphan-ish notes (useful for finding disconnected thesis nodes before creating synthesis)
lens list sources --source-type book --json # Filter by source type
lens list sources --inbox --json          # Sources awaiting agent processing (set by clippers)
lens list tasks --status open --json      # Tasks by status (open/done)

# Analyze
lens similar <id|title> --json            # Find near-duplicates (+ --threshold)
lens similar --all --json                 # Scan all notes, group duplicates
lens digest [week|month|year] --json      # Recent insights grouped by type
lens lint --json                          # Graph quality checks (9 checks) with offender IDs
lens lint --audit <check> --json          # Full offender export with context for batch fixing
lens lint --check --json                  # Same + exit code 1 on failures (for CI)
lens lint --summary --json                # Stats + graph health + user context

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
lens schema --json                        # Machine-readable command catalog (preferred for self-discovery)
lens doctor --json                        # Self-diagnostic (paths, git, DB integrity, schema version)
lens init                                 # First-time setup; re-run to repair a half-init
```

**Agents**: prefer `lens schema --json` over hard-coding this command list. It returns every command's inputs, output shape, examples, and `readonly` flag — always in sync with the installed lens version.

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

If a near-duplicate is found (similarity > 0.5), merge them: `lens write '{"type":"merge","from":"dup_id","into":"keep_id"}' --json`. This redirects links, appends body, and rewrites `[[ID]]` refs in one step.

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

- **All JSON output uses an envelope** (lens v1.21.0+): success → `{ok: true, schema_version: 1, data: {...}}`, error → `{ok: false, schema_version: 1, error: {code, message}, hint?: "..."}`. Always check `ok` before reading `data`; follow `hint` to decide the next action.
- **Readonly-safe commands**: `search`, `show`, `links`, `list`, `similar`, `lint --summary`, `schema`, `doctor` work when LENS_HOME is read-only (CI, sandboxes, mounted caches). Writes require a writable LENS_HOME.
- `show`, `links`, `similar` accept ID or title — no need to resolve first. If ambiguous, returns candidates.
- `show` supports batch: `lens show id1 id2 id3 --json` returns `{count, items}`.
- `show` returns full `forward_links[]` and `backward_links[]` arrays. `links` returns `forward[]` and `backward[]`.
- `links --rel related` filters by type. `links --direction forward` filters by direction. Combine both.
- `search --expand` returns full note bodies — use it for synthesis. Plain `search` returns titles only — use it for finding.
- `list tasks --status open` replaces the old `tasks` command. `list notes --min-links 10` finds hub notes.
- `lint --summary` replaces the old `status` command (includes user context).
- Write operations include `retype` (atomic link type change, inherits reason if not specified) and `merge` (atomic note merge with link redirect and [[ID]] rewrite).
- Batch writes use `$0`/`$1` to reference earlier items' IDs.
- Links are idempotent. `contradicts` is auto-bidirectional.
- `lint --audit <check>` returns all offenders with full context (titles, reasons) for batch fixing. Available checks: `supports_density` (evidence backbone health), `super_connectors` (notes with >30 inbound — apply chain topology to repair), `related_dominance` (audit related links), `missing_reasons`, `vague_reasons`, `duplicate_links`, `thin_notes`, `superseded_alive`.
- **Hub advisory in write response:** when a `{"type": "link"}` write causes a target to exceed 20 inbound links, the response includes `advisory.warning_code == "approaching_super_connector"` with `target_id`, `rel_breakdown`, and `is_healthy_hub`. In batch writes, advisories are aggregated in a top-level `warnings[]` array (keyed by `target_id`). Only explicit `link` items trigger the advisory — inline `links[]` on note/task/update writes do not. `is_healthy_hub` is `true` only when the target has inbound `indexes` links and `indexes >= supports`; otherwise apply chain topology: new notes → L2 synthesis → `refines` → master. See [references/api.md](references/api.md) and [references/compilation.md](references/compilation.md) for the repair pipeline.
- `indexes` links are exempt from reason requirements (structural, not semantic).
- Never truncate IDs. Always copy the full `prefix_` + 26-char ULID.
- Curly quotes break JSON — use straight quotes only.
