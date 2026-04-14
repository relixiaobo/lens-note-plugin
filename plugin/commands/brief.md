---
description: Assemble a knowledge briefing on a topic before deep work
argument-hint: '<topic>'
allowed-tools: Bash(lens:*)
---

Prepare a knowledge briefing on: $ARGUMENTS

1. Check keyword index: `lens index "$ARGUMENTS" --json`
   - If entries found, start from those IDs — they are the most connected entry points
2. Assemble full context: `lens context "$ARGUMENTS" --json`
3. For notes with contradictions (`contradicts` links), read both sides: `lens show <id> --json`

Present as a structured briefing:

**Core ideas** — the most connected notes on this topic (brief summary each)
**Tensions** — contradicting notes, if any (state the conflict clearly)
**Open threads** — seeds with no links yet (ideas waiting to collide)

End with: "N notes found. Key entry point: <title> (`<id>`)."

Do not create or modify any notes. This is read-only.
