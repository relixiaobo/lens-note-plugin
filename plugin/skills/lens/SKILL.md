---
name: lens
description: Store, query, and link knowledge in a persistent knowledge graph. Use when the user wants to save a note, record knowledge, asks "save this", "remember this", references prior research, asks "what do I know about...", wants to compile an article, says "check lens", asks about tasks or TODOs ("what tasks", "check tasks"), or when the conversation topic may relate to previously compiled knowledge.
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
| **links[]** | Typed semantic edges: supports, contradicts, refines, related, indexes, continues. | "backlinks", "wikilinks" |
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

**Note on `continues`**: this rel is deprecated in the decision path. Luhmann's Folgezettel chain was a paper-era workaround for the absence of search and filtering. In a modern digital graph, what a "chain" expresses is reconstructible from temporal order + topic filter + existing structural links. Explicitly marking chain parents adds friction in the wrong place (creation intent is fragile data and shouldn't require manual tagging). The `continues` rel remains in the schema for backward compatibility but should not be used in new writes. See `docs/chain-model.md` in the lens repo for the full reasoning.

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

**Why this matters:** The most common source of graph quality degradation is bulk-import `supports` generated by topic proximity. A reason like "A and B both discuss distributed systems" is not evidence — it's a topic label. Every spurious `supports` link degrades the signal value of every real `supports` link in the graph.

The rule of thumb: **if the reason could apply to more than one target note, it's not a `supports` reason — it's a `related` reason.**

`related` can always be upgraded to `supports` after careful review. The reverse (downgrading spurious `supports`) requires a manual audit pass.

### Post-import lint habit

After any bulk import or reading session, always run:

```bash
lens lint --audit vague_reasons --json   # catch supports used as topic labels
lens lint --json                         # check supports_density and super_connectors
```

The `vague_reasons` audit now flags `supports` links whose reasons only describe topic proximity (pattern: "A and B" with no explanatory verb). These are the most common source of spurious hub notes.

After `vague_reasons` reports offenders, do not just bulk-downgrade. For any offender thesis with ≥ 5 incoming offenders, shift to **per-target audit** — see [references/curation.md](references/curation.md) "Per-Target Supports Audit". A single under-examined thesis often collects the majority of an import's bad supports, and a per-target decision tree (keep-with-reason / downgrade / unlink / retype-to-contradicts) gives better outcomes than bulk edit.

## Setup

Check: `which lens`. If missing: `npm install -g lens-note && lens init`

## User Context

Check context on first use: `lens lint --summary --json` — look for `context` field.

If missing, ask the user for **role**, **audience**, **language**, **style** and save via `lens config`:
```bash
printf '%s' '{"command":"config","input":{"action":"set","key":"context.role","value":"product manager"}}' | lens --stdin
```

When writing notes, adapt to the context:
- **Role** → infer the right angle from content (product → user impact; technical → trade-offs; personal → reflection)
- **Audience** → self: jargon OK, be direct. team: explain cross-domain terms. public: full context.
- **Language** → write in the specified language; keep foreign quotes in blockquotes
- **Style** → apply the user's stated principle literally (e.g., "be concise" → short sentences, no filler)

## Decide Your Mode

When lens is relevant, identify the mode before acting:

| User says | Mode | What to do |
|-----------|------|------------|
| "save this" / quick thought | **Capture** | Search if topic might exist → write → map candidate → link. |
| "compile" / "analyze this article" | **Compile** | Read [references/compilation.md](references/compilation.md) first. |
| "what do I know about X" | **Query** | Search → map top results → show key notes → synthesize. |
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
Done. Created note_01ABC... Added 3 related links.
Structure note [[note_01DEF...]] updated. Pending task deleted.
```

**Good** (knowledge report):
```
The three caching approaches now form a coherent cluster —

They split along a latency/safety axis: the first leans hard on
prefix caching for low latency, the second caches the full system
prompt and accepts staleness risk, the third caches only tool
definitions and keeps the rest cold. That lines up with your
earlier note that infra choices leak a team's risk appetite —
cache granularity turns out to be a direct read on it.

Filed under the cross-team comparison index; 5 links in place.
```

### Rules

1. **Lead with insight, not action.** First say what was learned or what's interesting. Put the mechanical operations ("created", "linked", "deleted") at the end, briefly.

2. **Name, don't number.** Refer to notes by title, not raw ID. IDs can go in parentheses if needed: "The negative cost of high-quality software (note_01ABC...)" — but never ID-only.

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

Run `lens schema --json` to get the full, always-current command catalog (inputs, output shapes, examples, readonly flags). Key commands to know:

| Intent | Command | When to use |
|--------|---------|-------------|
| Find notes | `lens search "<query>" --json` | Know what you're looking for |
| Read content | `lens show <id> --json` | Need the full text |
| See cluster structure | `lens map <id> --json` | Understand a topic area before diving in |
| Filter links precisely | `lens links <id> --rel --json` | Before retype/unlink operations |
| Find new connections | `lens discover <id> --json` | Spatial browsing, dedup, collision |
| Write/modify | `lens write --file <path> --json` | Create or change anything |
| Recent activity | `lens digest week --json` | "What happened this week" |
| Graph quality | `lens lint --json` | Check health, find issues |
| Keyword entry | `lens index --json` | "Where do I start" |

### Calling conventions

- **--stdin** (preferred for agents): `printf '%s' '{"command":"...","positional":[],"flags":{},"input":{}}' | lens --stdin`
- **--file** (preferred for Chinese/special chars): write JSON to temp file, then `lens write --file <path> --json`
- **Title resolution**: all write operations accept titles in place of IDs — no need to resolve first
- **Write-time suggestions**: note creation returns `suggestions[]` with unlinked-but-related notes

## Mode: Capture

For quick thoughts, observations, ideas. Fast, but not blind.

**Writing and placement are two steps, not one.** Finding candidates is the skill's job — removes scanning friction. **Choosing which to link and what rel is the user's judgment** — this is where the thinking happens. Never collapse them by auto-picking links for the user.

### Annotated sources

If the capture draws from a source that has `annotations[]`, those annotations are **strong must-capture signals** — the user has explicitly marked those passages. Extract the annotated content as the primary note(s). Never skip an annotation in favor of other content you find more interesting.

But annotations are **not a ceiling**. Highlights and comments mark what the user noticed in the moment; they do not mean nothing else in the source matters. Scan the rest of the source for additional thesis-grade claims and surface them to the user as candidates — e.g. "not annotated but looks important: …, want to capture it?" Do **not** autonomously write notes from un-annotated passages; always ask first.

### Step 1 — Search (when the topic may already exist)

If the thought touches something likely already in the graph:

```bash
lens search "key concepts" --json
```

If a tight match surfaces (very similar title/claim), check whether this is actually a merge or update before writing a new card.

### Step 2 — Write

Write the note. Include only links you are highly confident about (explicit `contradicts` stated by the user, quote from a named source, etc.). Leave the rest for Step 3.

```bash
printf '%s' '{"command":"write","input":{"type":"note","title":"Simple tools composed together beat complex frameworks"}}' | lens --stdin
```

The write response returns `suggestions[]` — up to 5 unlinked-but-related notes ranked by similarity. **Do not discard this field.**

### Step 3 — Placement (ask the user)

Surface `suggestions[]` to the user. Do NOT silently ignore them; do NOT autonomously link them.

**Presentation format: labeled candidates + shorthand response.**

Each candidate gets a short label (a / b / c for one note, or 1 / 2 / 3 for multi-note). Show the skill's recommendation inline (strength + suggested rel + why), but keep final judgment with the user. Example:

> Saved "Small composable tools beat monolithic frameworks" (note_01X…)
>
> Placement candidates:
>
>   a · [strong] refines  → Revisiting the Unix pipeline philosophy
>       why: new note is the Unix principle applied to agent tooling
>   b · [mid]    related  → Case study: a sub-agent system built from small tools
>       why: same thesis, different case — cross-source resonance
>   c · [weak]   related  → Orthogonality as a design heuristic
>       why: related principle but different abstraction level; may skip
>
> Reply: `a,b` / `a` / `a supports, b skip` / `all` / `skip`

**Accept shorthand responses.** Parse the reply as:
- Bare labels (`a`, `b,c`, `a,b`) → link those with the recommended rel
- Label + rel (`a supports`, `b refines`) → override the recommended rel
- `all` → link every candidate with recommended rel
- `skip` / `none` / no reply → link nothing
- Inline modifiers (`b skip`) → drop specific items from a broader selection

**Always include the skill's recommendation** (strong / mid / weak + suggested rel + one-line why) so the user can say `a,b` without re-reading every candidate in detail. Cost of response: one line.

Edge cases:
- Similarity > 0.85 on a candidate → propose **merge** instead of link (`lens write '{"type":"merge","from":"<new_id>","into":"<existing_id>"}' --json`)
- Similarity 0.5–0.85 → usually `refines` / `supports` / `contradicts`
- Similarity < 0.5 → usually `related` with explicit reason, or no link
- Similarity < 0.1 → don't show; surface the note as "new territory" instead (see Exceptions below)

**Never flood the user.** Cap at ~5 candidates per note. If more exist, show top 5 by relevance and note "(N more weaker matches not shown)".

### Step 4 — Close out the source (if capturing from inbox)

If the note you just wrote references an inbox source (via `[[src_...]]` in body or the `source` field), immediately flip the source out of inbox:

```bash
printf '%s' '{"command":"write","input":{"type":"update","id":"src_01K...","set":{"inbox":false}}}' | lens --stdin
```

`inbox: true` means "awaiting processing." Once a note captures content from the source, processing is done. Leaving it in inbox turns the queue into noise.

### Exceptions to Step 3

- User explicitly says "just save" / "quick save" → skip Step 3, write is done (still do Step 4 if inbox source)
- `suggestions[]` is empty → **new territory**. Say so to the user ("no close neighbors in the graph yet — this is a new direction") — don't treat it as "nothing to do"; it's a signal worth mentioning. Still do Step 4.
- All candidates' similarity < 0.1 → effectively new territory. Same treatment as above. Don't parade noise as candidates.
- Capture during bulk Compile/Import → follow that mode's rules, not Capture's

### Why this shape

Luhmann wrote each card first, then carried it to the box and looked for where it fit. The search ("find candidates") and the judgment ("decide placement") are distinct cognitive acts. AI is excellent at the first and should never pretend to do the second unless the user explicitly delegates it.

One rule: **one idea per note.** If the thought has multiple claims, split into separate notes.

### Dedup check with `lens discover --duplicates`

After creating a note, check for near-duplicates:

```bash
lens discover <id> --duplicates --json                # default threshold: 0.3
lens discover <id> --duplicates --threshold 0.5 --json  # stricter matching
```

To scan all notes at once and group duplicates:

```bash
lens discover --all --duplicates --json               # all groups above 0.3
lens discover --all --duplicates --threshold 0.8 --json  # only high-confidence duplicates
```

- Only works on notes (not sources or tasks)
- Uses TF-IDF + cosine similarity with ICU word segmentation — multilingual (CJK, Latin, Arabic, etc.)
- `--threshold`: 0–1 float. Default 0.3 catches loose duplicates; 0.5+ for stricter matching
- Single-note output: `{"id": "...", "count": N, "results": [{"id": "...", "title": "...", "similarity": 0.65}, ...]}`
- `--all` output: `{"count": N, "groups": [{"notes": [...], "pairs": [{"a": "...", "b": "...", "similarity": 0.9}]}]}`

If a near-duplicate is found (similarity > 0.5), merge them: `lens write '{"type":"merge","from":"dup_id","into":"keep_id"}' --json`. This redirects links, appends body, and rewrites `[[ID]]` refs in one step.

### Discovery modes

`discover` has three modes for different intents:

| Mode | Flag | What it finds | Use when |
|------|------|---------------|----------|
| Default | (none) | Unlinked-but-related notes | Filing a note — "what should I link this to?" |
| Collide | `--collide` | Cross-domain surprises | Collision Method — "surprise me with unexpected connections" |
| Duplicates | `--duplicates` | Near-duplicates for merge | Dedup — "is this already in the graph?" |

```bash
lens discover <id> --json                 # What's nearby but unlinked? (exclude 2-hop neighbors)
lens discover <id> --collide --json       # Cross-domain collision (exclude entire connected component)
lens discover <id> --duplicates --json    # Find duplicates (no exclusion)
```

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

**Close out the source when done.** If the source was in inbox, flip `inbox: false` once compilation is complete — regardless of how many notes were produced (including zero). The inbox queue tracks what still needs processing, not what was fruitful. If a source gets re-opened later, the user can manually re-flag it.

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

## API Quick Reference

For full details: [references/api.md](references/api.md). Essentials:

- **JSON envelope**: success → `{ok: true, schema_version: 1, data: {...}}`, error → `{ok: false, ..., hint: "..."}`. Always check `ok`; follow `hint`.
- **Title resolution**: write operations accept titles in place of IDs. Ambiguous → returns candidates.
- **Write-time suggestions**: note creation returns `suggestions[]` with unlinked-but-related notes.
- **ID or title**: `show`, `map`, `links`, `discover` all accept either. No need to resolve first.
- **Batch writes**: use `$0`/`$1` to reference earlier items' IDs. Links are idempotent. `contradicts` is auto-bidirectional.
- **Hub advisory**: when a link write exceeds 20 inbound, response includes advisory with `rel_breakdown` and `is_healthy_hub`. See [references/compilation.md](references/compilation.md) for repair pipeline.
- **Encoding**: curly quotes break JSON — use straight quotes only. For Chinese content, prefer `--file` over `--stdin`.
