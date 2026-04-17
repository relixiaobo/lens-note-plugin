---
description: Discover hidden contradictions in your knowledge graph
argument-hint: '[topic]'
allowed-tools: Bash(lens:*)
---

Find contradictions in the knowledge graph. Topic focus: $ARGUMENTS

**Step 1 — Gather thesis-level notes:**

If a topic is provided:
```bash
lens search "$ARGUMENTS" --expand --json
```

If no topic (scan broadly):
```bash
lens list notes --min-links 3 --limit 30 --json
```

Then read the top notes:
```bash
lens show <id1> <id2> <id3> ... --json
```

Focus on notes whose titles make claims — assertions, prescriptions, or generalizations. Skip pure observations ("Manus does X") and focus on theses ("X is the right approach", "Y matters more than Z").

**Step 2 — Identify tension pairs:**

For each thesis note, ask: does any other note in this set argue the opposite, or make an incompatible assumption?

Look for:
- **Direct opposition**: "Use small context windows" vs "Large context enables better reasoning"
- **Incompatible assumptions**: One note assumes X is cheap, another assumes X is expensive
- **Boundary conditions**: Both are right, but in different contexts — the tension is about where the boundary lies
- **Evolution**: An earlier note says X, a later note says not-X — the user's thinking has shifted

Do NOT flag as tension:
- Notes that merely discuss different aspects of the same topic
- Notes where one is strictly more specific than the other (that's `refines`, not `contradicts`)
- Notes that could both be true simultaneously without conflict

**Step 3 — Present discoveries:**

For each genuine tension found (aim for 2-5):

**Tension: [one-sentence summary of the conflict]**
- Side A: [note title] — [what it claims, one sentence]
- Side B: [note title] — [what it claims, one sentence]
- The crux: [where exactly do they disagree? what assumption differs?]
- Already linked? Check `lens links <id> --json` — if `contradicts` already exists, skip.

**Step 4 — Check existing contradicts coverage:**

```bash
lens lint --json
```

Look at the `weak_contradicts` check. Report: "Your graph has N contradicts links across M notes. [healthy/sparse]."

**Step 5 — Offer to create links:**

For each new tension discovered, offer to create a `contradicts` link (auto-bidirectional). The reason should capture the crux, not just "they disagree."

Ask the user which ones to create before writing anything. Contradicts is the most valuable link type — it should be precise.
