---
description: Review recent learnings — tensions, connections, and seeds
argument-hint: '[week|month|year]'
allowed-tools: Bash(lens:*)
---

Review recent knowledge activity. Period: $ARGUMENTS (use `week` if no argument provided)

Run these in order:

1. Digest: `lens digest $ARGUMENTS --json` (substitute `week` if $ARGUMENTS is empty)
2. Open tasks: `lens tasks --json`
3. Orphan seeds: `lens list notes --orphans --limit 5 --json`

Present as a structured review:

**Tensions** (N) — notes with `contradicts` links. These are the most interesting: state each conflict in one sentence.
**Connected** (N) — notes that found their place in the graph.
**Seeds** (N) — unlinked notes. List 3–5 with titles — potential collisions waiting to happen.
**Open tasks** (N) — list titles if any exist.

Close with one synthesis observation: the single most interesting tension or seed worth following up on, and why.
