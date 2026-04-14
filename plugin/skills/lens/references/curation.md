# Keeping the Graph Healthy

How to maintain knowledge graph health — find orphans, make connections, clean up what's stale.

## When to Curate

- User explicitly asks ("clean up my notes", "fix orphans")
- `lens status --json` shows high orphan rate (>10%)
- After a batch of compilations, as a follow-up pass

## Process

### 1. Check health

```bash
lens status --json
```

Look at `connectivity.orphan_count`. Orphans are notes with zero links to other notes. Note: `status` only reports the count — use `lens list notes --orphans --json` to get actual orphan IDs and previews.

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

- **supports**: this note strengthens another note's claim
- **contradicts**: this note conflicts with another (auto-bidirectional)
- **refines**: this note is a more precise version of another
- **related**: weak association — use sparingly, only when the relationship is real but doesn't fit the above types
- **indexes**: this note organizes/indexes the target note (for MOC/structure notes)

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

## Merge and Supersede

**Merge**: when two notes say essentially the same thing, keep the better one. Update it with any unique content from the other. Delete the duplicate.

```bash
printf '%s' '{"command":"write","input":{"type":"update","id":"note_KEEP","body":"merged body..."}}' | lens --stdin
printf '%s' '{"command":"write","input":{"type":"delete","id":"note_DUPLICATE"}}' | lens --stdin
```

**Supersede**: when understanding has evolved and an old note is no longer accurate. Don't delete — update the body to note it's superseded, and link to the newer note.

```bash
printf '%s' '{"command":"write","input":{"type":"update","id":"note_OLD","body":"**Superseded** by note_NEW.","add":{"links":[{"to":"note_NEW","rel":"related","reason":"superseded by newer understanding"}]}}}' | lens --stdin
```

## Structure Notes and the Keyword Index

Two complementary navigation tools — don't confuse them:

- **Structure note**: a regular note whose body uses `[[note_ID]]` to reference 3–8 entry-point notes on a topic, with `rel: "indexes"` in `links[]`. Created manually after a cluster forms naturally.
- **Keyword index** (`lens index`): a sparse registry mapping a keyword string to 1–3 note IDs. No body, no reasoning — just a pointer. Used for quick navigation entry across the whole graph.

Use them together: the keyword index gets you into a cluster; the structure note shows the cluster's shape.

Rules for structure notes:

- Create them only AFTER a cluster of related notes has formed naturally
- Link to child notes via `links[]` with `rel: "indexes"` — this distinguishes "organizes" from "semantically relates"
- Use `[[note_ID]]` in the body for inline references (resolved to `[Title](ID)` on read)
- They are navigational aids, not categories
- Most knowledge graphs need very few structure notes
- Never create one per article or per topic automatically

### Dense Note Audit

A note with more than 8 forward links is worth inspecting — not automatically wrong, but suspicious. Ask: can you articulate in one sentence why each specific link exists? If not, the link is noise.

```bash
lens show <id> --json    # count forward_links.length
```

Common causes and fixes:

- **Compilation linked everything to one "hub" note** → remove links you can't justify
- **One note doing two separate jobs** → split it; connect the two halves with `refines` or `related`

A carefully curated structure note with 8 links is fine. A compilation note with 15 loose `related` links is not.

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
lens similar --all --threshold 0.8 --json    # high-confidence duplicates only
```

If two new notes are near-duplicates (similarity > 0.8), merge them: update the better one with any unique content, then delete the weaker one.

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
- Creating structure notes proactively before clusters form
- Ignoring orphans entirely (they should be revisited when new related knowledge arrives)
- Deleting notes instead of superseding them
