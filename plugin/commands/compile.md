---
description: Compile an argument outline from your knowledge graph
argument-hint: '<thesis or topic>'
allowed-tools: Bash(lens:*), Write, Read
---

Compile an argument from the knowledge graph about: $ARGUMENTS

**Goal**: Turn scattered atomic notes into a coherent argument. Gather what the graph says, identify the structure, propose an ordered outline, and (after approval) save as a `continues` chain.

**Gather**: Search broadly (`--expand`), check the keyword index, and follow links from top hits. Read full bodies. Aim for 10-30 relevant notes. Don't stop at first results — follow links to discover the full cluster.

**Identify**:
- The central thesis (or 2-3 competing positions if notes don't converge)
- Supporting evidence — specific cases, examples, data
- Tensions and nuances — `contradicts` links, edge cases
- The gap — what's NOT in the graph that the argument needs

**Propose**: A numbered outline of 5-15 points. Each point:
1. **[Claim]** — from [note title(s)]
   Evidence summary + transition to next point.

Order matters — each point builds on the previous. Mark where evidence is thin or tensions exist.

**Approve**: Present the outline and ask:
- Does this capture what you want to argue?
- Should anything be reordered, removed, or added?
- Are the gaps acceptable?

**Save** (only after approval):
- Create an outline note with the argument structure in the body
- Link source notes with `supports`
- Create `continues` links between consecutive argument steps where they don't exist
- Register as keyword index entry if the topic lacks one
