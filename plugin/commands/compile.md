---
description: Compile an argument outline from your knowledge graph
argument-hint: '<thesis or topic>'
allowed-tools: Bash(lens:*), Write, Read
---

Compile an argument from the knowledge graph about: $ARGUMENTS

**Step 1 — Gather material:**

Search broadly for related notes:
```bash
lens search "$ARGUMENTS" --expand --json
```

Also check the keyword index:
```bash
lens index "$ARGUMENTS" --json
```

For each relevant note found, follow its links to discover the full cluster:
```bash
lens links <id> --json
lens show <linked_ids> --json
```

Continue following links until you have a comprehensive view of what the graph says about this topic. Read the full body of each relevant note (`lens show` returns bodies). Aim for 10-30 notes.

**Step 2 — Identify the argument structure:**

From the gathered notes, identify:
- **The central thesis**: What is the user's main claim about $ARGUMENTS? If the notes don't converge on a single thesis, identify the 2-3 competing positions.
- **Supporting evidence**: Notes that provide specific evidence, examples, or case studies.
- **Tensions and nuances**: `contradicts` links, edge cases, boundary conditions.
- **The gap**: What's NOT in the graph that the argument needs? Missing evidence, unexplored angles, unresolved tensions.

**Step 3 — Propose the outline:**

Present a numbered outline of 5-15 points. Each point is:

1. **[Claim in one sentence]** — from [note title(s)]
   Brief note on what evidence supports this and what transitions to the next point.

The outline should read as a coherent argument, not a list of notes. Order matters — each point should build on the previous one. Use the `continues` relationship logic: each step follows from the last.

Mark any points where the evidence is thin or the transition is weak. Mark any points where a genuine tension exists (cite both sides).

**Step 4 — Ask for approval:**

Present the outline and ask:
- Does this capture what you want to argue?
- Should any points be reordered, removed, or split?
- Are the marked gaps acceptable, or should we find more material first?

**Step 5 — Save as a continues chain (only after user approval):**

Create a new note for the outline itself, then link the source notes in order using `continues`:

Write a JSON batch to a temp file and execute:

```bash
lens write --file /tmp/compile-chain.json --json
```

The batch should:
1. Create an outline note with the full argument structure in the body
2. Link each source note to the outline with `supports` (they provide evidence)
3. Between consecutive argument steps, if no `continues` link exists, create one

Register the outline note as a keyword index entry if the topic doesn't have one:
```bash
lens index add "$ARGUMENTS" <outline_id> --json
```

Report what was created: the outline note, how many continues links, and any gaps that remain for future work.
