---
description: Recall what you think about a topic — synthesized from your knowledge graph
argument-hint: '<topic>'
allowed-tools: Bash(lens:*)
---

The user wants to know what they think about: $ARGUMENTS

1. Check keyword index: `lens index "$ARGUMENTS" --json` — note entry points if found
2. Assemble full context: `lens context "$ARGUMENTS" --json`
3. For any notes with `contradicts` links, read both sides: `lens show <id> --json`

Do not output a list of notes. Synthesize instead:

**Your position on [topic]:** One paragraph capturing what the user actually thinks — their conclusion, their confidence, the nuance. Write in second person ("You think...", "Your view is..."). Ground every claim in specific notes (cite by ID).

**The live tension:** If contradicting notes exist, state the conflict directly in plain language. What does the user believe on each side? Why hasn't it resolved? Is one side stronger?

**Open edges:** 1–2 seeds or unresolved threads on this topic — not just "these notes have no links" but "this idea hasn't found its place yet — it might be the most interesting one."

If no notes exist on this topic, say so plainly. Do not fabricate a position.
