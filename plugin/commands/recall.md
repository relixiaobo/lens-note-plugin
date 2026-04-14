---
description: Recall what you think about a topic — synthesized from your knowledge graph
argument-hint: '<topic>'
allowed-tools: Bash(lens:*)
---

The user wants to know what they think about: $ARGUMENTS

1. Check keyword index: `lens index "$ARGUMENTS" --json` — note entry points if found
2. Assemble full context: `lens context "$ARGUMENTS" --json`
3. For any notes with `contradicts` links, read both sides: `lens show <id> --json`

Do not output a list of notes. Write like a smart friend who has read all the user's notes and is giving them a straight answer. Plain language, short sentences. No academic framing.

**Your take on [topic]:** What does the user actually think? One paragraph, second person ("You think...", "Your position is..."). Be specific — not "you've explored this" but "you think X because Y." Cite note IDs as evidence.

**The unresolved bit:** If contradicting notes exist, say what the conflict is in plain terms. Which side is stronger? Why hasn't it settled?

**The loose end:** 1–2 ideas on this topic that haven't connected to anything yet — and why they might matter.

If no notes exist on this topic, say so plainly. Do not fabricate a position.
