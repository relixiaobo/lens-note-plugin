# Curation Methodology

How to maintain knowledge graph health. Based on Karpathy's "lint" concept and Luhmann's link discipline.

## When to Curate

- User explicitly asks ("clean up my notes", "fix orphans", "整理笔记")
- `lens status --json` shows high orphan rate (>10%)
- After a batch of compilations, as a follow-up pass

## Process

### 1. Check health

```bash
lens status --json
```

Look at `connectivity.orphan_count`. Orphans are notes with zero links to other notes.

### 2. For each orphan note

```bash
lens show <orphan_id> --json
lens search "keywords from the note" --json
```

### 3. Decide the relationship

- **supports**: this note strengthens another note's claim
- **contradicts**: this note conflicts with another (auto-bidirectional)
- **refines**: this note is a more precise version of another
- **related**: weak association — use sparingly, only when the relationship is real but doesn't fit the above types

### 4. Add the link (only if justified)

```bash
echo '{"type":"link","from":"orphan_id","rel":"supports","to":"target_id"}' | lens write --json
```

**No link is better than a weak link.** If you can't articulate why two notes are connected, don't link them. The orphan is a seed — it will find its connections when related knowledge arrives in the future.

## Merge and Supersede

**Merge**: when two notes say essentially the same thing, keep the better one. Update it with any unique evidence from the other. Delete the duplicate.

```bash
echo '{"type":"update","id":"note_KEEP","add":{"evidence":[...]}}' | lens write --json
echo '{"type":"delete","id":"note_DUPLICATE"}' | lens write --json
```

**Supersede**: when understanding has evolved and an old note is no longer accurate. Don't delete — mark as superseded so the history is preserved.

```bash
echo '{"type":"update","id":"note_OLD","set":{"status":"superseded"}}' | lens write --json
```

## Structure Notes

Structure notes (`role: structure_note`) are sparse index entry points. Rules:

- Create them only AFTER a cluster of related notes has formed naturally
- They point to 3-8 entry-point notes via `entries[]`
- They are navigational aids, not categories
- Most knowledge graphs need very few structure notes
- Never create one per article or per topic automatically

## Anti-Patterns

- Linking every orphan to the nearest vaguely-related note (noise)
- Creating structure notes proactively before clusters form
- Ignoring orphans entirely (they should be revisited when new related knowledge arrives)
- Deleting notes instead of superseding them
