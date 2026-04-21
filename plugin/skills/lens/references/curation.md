# Keeping the Graph Healthy

How to maintain knowledge graph health — find orphans, make connections, clean up what's stale.

## Contents

- [When to Curate](#when-to-curate)
- [Process](#process)
- [Merge and Supersede](#merge-and-supersede)
- [Structure Notes and the Keyword Index](#structure-notes-and-the-keyword-index)
- [After Bulk Compilation](#after-bulk-compilation)
- [Dead Link Cleanup](#dead-link-cleanup)
- [Anti-Patterns](#anti-patterns)

## When to Curate

- User explicitly asks ("clean up my notes", "fix orphans")
- `lens lint --summary --json` shows high orphan rate (>10%)
- After a batch of compilations, as a follow-up pass

## Process

### 1. Check health

```bash
lens lint --summary --json    # quick stats + orphan count
lens lint --json              # full quality report with offender IDs
```

Look at `connectivity.orphan_count`. Orphans are notes with zero links to other notes. Use `lens list notes --orphans --json` to get actual orphan IDs.

### 2. Get orphan details

For small counts (<20 orphans), process them directly:

```bash
lens list notes --orphans --json
```

For large counts (20+ orphans), use pagination to work in batches:

```bash
lens list notes --orphans --limit 10 --json          # first 10
lens list notes --orphans --limit 10 --offset 10 --json  # next 10
```

### 3. For each orphan note

```bash
lens show <orphan_id> --json
lens search "keywords from the note" --json
```

### 4. Decide the relationship

Work through this decision tree in order:

1. **Does A oppose or challenge B?** → `contradicts` (auto-bidirectional)
2. **Is A a more specific version, concrete implementation, or case study of B?** → `refines`
3. **Does A strengthen, provide evidence for, or validate B?** → `supports`
4. **None of the above, but genuinely connected?** → `related` (must provide a reason explaining HOW — "related to same topic" is not enough)

For topic grouping or MOC-style organization, use a whiteboard (`lens board`) instead of a typed link.

`related` requires a `reason` field. If you cannot articulate the relationship direction, the link is probably not worth creating.

Run `lens lint --json` to check graph health — 9 checks including related dominance, missing/vague reasons, thin notes, superseded-but-referenced notes, super-connectors, duplicate links, and dead links. Use `lens lint --check --json` for CI (exit code 1 on failures).

### 5. Add the link (only if justified)

```bash
printf '%s' '{"command":"write","input":{"type":"link","from":"orphan_id","rel":"supports","to":"target_id"}}' | lens --stdin
```

**No link is better than a weak link.** If you can't articulate why two notes are connected, don't link them. The orphan is a seed — it will find its connections when related knowledge arrives in the future.

### Remove incorrect links

If a link was added by mistake or no longer makes sense:

```bash
printf '%s' '{"command":"write","input":{"type":"unlink","from":"note_A","rel":"supports","to":"note_B"}}' | lens --stdin
```

`unlink` requires all three fields: `from`, `to`, `rel`. For `contradicts` links, the reverse direction is also removed automatically.

## Link Quality Audit

### Per-note audit

Use `links --rel` to review links by type on a single note. Forward links include `reason`, so you can assess without follow-up `show` calls:

```bash
lens links <id> --rel related --direction forward --json
```

### Graph-wide audit

Use `lint --audit` to export ALL offenders for a check with full context. Process the list, decide new rels, and batch retype:

```bash
# Step 1: Get all related links with titles and reasons
lens lint --audit related_dominance --json

# Step 2: Review offenders, generate batch retype
# (agent processes the list and decides supports/refines/keep)

# Step 3: Batch retype
printf '%s' '{"command":"write","input":[
  {"type":"retype","from":"note_A","to":"note_B","old_rel":"related","new_rel":"supports"},
  {"type":"retype","from":"note_C","to":"note_D","old_rel":"related","new_rel":"refines"}
]}' | lens --stdin

# Step 4: Verify
lens lint --json
```

`retype` inherits the existing reason when none is specified — no data loss on rel changes. It also handles bidirectional invariants: leaving `contradicts` removes the reverse link; entering `contradicts` creates it.

### Rel strength for duplicate cleanup

When `duplicate_links` reports a pair with multiple rels, keep the strongest: `contradicts`/`continues` > `refines` > `supports` > `related`. `lint --audit duplicate_links` suggests which to keep.

### Reason requirements

- `related`: **required** (CLI rejects without one)
- `supports`, `contradicts`, `refines`, `continues`: recommended (lint checks)

## Per-Target Supports Audit

Graph-wide `lint --audit missing_reasons` / `vague_reasons` produces a flat list of offending edges. That's correct for bulk fixing, but many low-quality `supports` clusters are **target-shaped**, not edge-shaped: one thesis accumulates N unverified inbound because the original import built it by adjacency. Audit those thesis-by-thesis.

### When to run

- After any bulk import (Tana, Roam, Obsidian migration)
- On every thesis you touch during Compile that has ≥ 5 inbound `supports`
- When `lint --summary` shows `supports_density` unusually high and you suspect topic-proximity links are inflating it

### Find candidates

```bash
# Thesis nodes with many inbound supports — audit targets, ranked by total link count
lens list notes --min-links 8 --json

# Scope the audit to one target — returns only links indicating this thesis with missing reasons
lens lint --audit missing_reasons --target <thesis_id> --json

# Or inspect inbound supports directly (reasons included in backward view as of v1.18):
lens links <thesis_id> --rel supports --direction backward --json
```

### Decision tree — for each inbound supports with null or vague reason

Read the source note's body. Then classify:

| Test on source body vs target thesis | Action | CLI |
|--------------------------------------|--------|-----|
| Body presents specific evidence, mechanism, case, or counter-case for the target thesis (you can honestly write *"This proves the thesis because…"*) | **Keep + annotate.** `retype` same-rel with a real reason. | `{"type":"retype","from":"<src>","to":"<thesis>","old_rel":"supports","new_rel":"supports","reason":"<mechanism>"}` |
| Body shares topic/domain but offers no mechanism (e.g. quote about the same person/field, no claim about the target's specific thesis) | **Downgrade to related.** Write a reason naming the topical connection honestly. | `{"type":"retype","from":"<src>","to":"<thesis>","old_rel":"supports","new_rel":"related","reason":"<shared topic>"}` |
| Body is unrelated — link is pure import residue | **Unlink.** Source is a seed that will find its real connections elsewhere. | `{"type":"unlink","from":"<src>","rel":"supports","to":"<thesis>"}` |
| Body actually *contradicts* the thesis | **Retype to contradicts.** Write a reason locating the tension. | `{"type":"retype","from":"<src>","to":"<thesis>","old_rel":"supports","new_rel":"contradicts","reason":"<tension>"}` |

Batch the decisions and write in one shot:

```bash
printf '%s' '{"command":"write","input":[
  {"type":"retype","from":"note_A","to":"note_T","old_rel":"supports","new_rel":"supports","reason":"body shows X directly proving thesis"},
  {"type":"retype","from":"note_B","to":"note_T","old_rel":"supports","new_rel":"related","reason":"both discuss Y, no evidential link"},
  {"type":"unlink","from":"note_C","rel":"supports","to":"note_T"},
  {"type":"retype","from":"note_D","to":"note_T","old_rel":"supports","new_rel":"related","reason":"adjacent framing, no mechanism"}
]}' | lens --stdin
```

### Heuristics — when you don't need to read every body

Skim the source titles against the thesis first. A typical 9-inbound thesis splits roughly:
- 1-2 real evidence (keep)
- 3-5 topic proximity (downgrade to related)
- 2-4 pure residue (unlink)

If titles alone make the call obvious (e.g. an observational title "Iwata says X" supporting a thesis about decision-making, where X is about product, not decision-making), skip the body read.

### Verify

```bash
lens links <thesis_id> --rel supports --direction backward --json   # remaining supports, all with real reasons
lens lint --audit vague_reasons --json                              # should not list this thesis anymore
lens lint --audit missing_reasons --target <thesis_id> --json       # should return 0 offenders
```

## Merge and Supersede

**Merge**: when two notes say essentially the same thing, use atomic merge. It redirects all inbound links, carries forward links, appends body, and rewrites `[[ID]]` references across the graph — all in one step.

```bash
# Find duplicates
lens discover <id> --duplicates --json

# Compare content
lens show <dup_a> <dup_b> --json

# Merge (deletes source, redirects links, appends body)
printf '%s' '{"command":"write","input":{"type":"merge","from":"note_DUPLICATE","into":"note_KEEP"}}' | lens --stdin
```

Merge is idempotent — retrying after the source is deleted returns `"action": "unchanged"`. Only notes can be merged.

**Supersede**: when understanding has evolved and an old note is no longer accurate. Don't delete — update the body to note it's superseded, and link to the newer note.

```bash
printf '%s' '{"command":"write","input":{"type":"update","id":"note_OLD","body":"**Superseded** by note_NEW.","add":{"links":[{"to":"note_NEW","rel":"related","reason":"superseded by newer understanding"}]}}}' | lens --stdin
```

## Whiteboards and the Keyword Index

Two complementary navigation tools — don't confuse them:

- **Whiteboard** (`lens board`): a spatial workspace that aggregates 3–30 related Notes/Sources/Tasks for topic research. Independent from the graph — adding a card does NOT create a link. Created manually after a cluster forms naturally.
- **Keyword index** (`lens index`): a sparse registry mapping a keyword string to 1–3 note IDs. No body, no reasoning — just a pointer. Used for quick navigation entry across the whole graph.

Use them together: the keyword index gets you into a cluster; the whiteboard shows the cluster's shape.

Rules for whiteboards:

- Create them only AFTER a cluster of related notes has formed naturally
- `lens board create --title "..."` then `lens board add <wb-id> <card-id>...` to populate
- Use `lens board show <wb-id>` to review members; `lens board find <card-id>` to see which boards contain a card
- They are workspace aggregations, not categories or taxonomy
- Most knowledge graphs need very few whiteboards

### Dense Note Audit

A note with more than 8 forward links is worth inspecting — not automatically wrong, but suspicious. Ask: can you articulate in one sentence why each specific link exists? If not, the link is noise.

```bash
lens show <id> --json    # count forward_links.length
```

Common causes and fixes:

- **Compilation linked everything to one "hub" note** → remove links you can't justify
- **One note doing two separate jobs** → split it; connect the two halves with `refines` or `related`

A compilation note with 15 loose `related` links is always noise — if those notes really belong together, they belong on a whiteboard, not chained off a single hub.

## After Bulk Compilation

After creating 5+ notes from a single source, do a brief audit before moving on.

### 1. Check for lateral connections

Read each new note. Do any of them collide with each other — not just with the source? Notes linked only to a source form a star topology, not a graph.

```bash
lens show <note_id> --json    # read each new note, check forward_links
```

Add note-to-note links where a genuine collision happened. If none do, that's fine — not every batch produces note-to-note edges.

### 2. Check for near-duplicates

```bash
lens discover --all --duplicates --threshold 0.8 --json    # high-confidence duplicates only
```

If two new notes are near-duplicates (similarity > 0.8), merge them with `{"type":"merge","from":"weaker","into":"better"}`.

### 3. Dense note check

For each new note, check `forward_links.length`. Anything above 8 → apply the Dense Note Audit above.

## Dead Link Cleanup

A dead link is a forward link in a note's `links[]` that points to an ID that no longer exists. The CLI cleans SQLite when a note is deleted, but it does not update the YAML frontmatter of notes that link to it. Dead links persist in frontmatter and reappear in SQLite after `rebuild-index`.

**Symptom**: `lens show note_A --json` shows a forward link to `note_B`, but `lens show note_B --json` returns a not-found error.

**Fix**: explicitly unlink the dead reference:

```bash
printf '%s' '{"command":"write","input":{"type":"unlink","from":"note_A","rel":"supports","to":"note_B"}}' | lens --stdin
```

Then run `lens rebuild-index --json` to resync SQLite with the cleaned frontmatter.

Note: if you don't know the `rel` type of the dead link, check `lens show note_A --json` — the `forward_links` array includes the `rel` field even for dead links (it's read from YAML).

## Anti-Patterns

- Linking every orphan to the nearest vaguely-related note (noise)
- Creating whiteboards proactively before clusters form
- Ignoring orphans entirely (they should be revisited when new related knowledge arrives)
- Deleting notes instead of superseding them
