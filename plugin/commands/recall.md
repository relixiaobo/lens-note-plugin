---
description: Recall what you think about a topic — synthesized from your knowledge graph
argument-hint: '<topic>'
allowed-tools: Bash(lens:*)
---

The user wants to know what they think about: $ARGUMENTS

1. Check keyword index: `lens index "$ARGUMENTS" --json` — note entry points if found
2. Assemble full context: `lens context "$ARGUMENTS" --json`
3. For any notes with `contradicts` links, read both sides: `lens show <id> --json`
4. Count how many notes matched; check how many are orphans (no links)

Do not output a list of notes. Write like a smart friend who has read all the user's notes and is giving them a straight answer. Plain language, short sentences. No academic framing.

**Your take on [topic]:** What does the user actually think? One paragraph, second person ("You think...", "Your position is..."). Be specific — not "you've explored this" but "you think X because Y." Cite note IDs as evidence.

**The unresolved bit:** If contradicting notes exist, say what the conflict is in plain terms. Which side is stronger? Why hasn't it settled?

**The loose end:** 1–2 ideas that haven't connected to anything yet — and for each, suggest a *specific* action: a note worth writing, a connection worth making, or a contradiction worth recording. Not just "this could connect to X" but "write a note about [concrete claim] and link it to [note_id]."

**Graph status for this topic:**
- N notes found, M are orphans (unlinked)
- If orphan rate is high: "Several notes on this topic are isolated — consider linking them."
- If note count > 10 and no keyword index entry exists: "This topic has enough depth for a keyword index entry. Pick the best starting-point note and run `lens index add`."
- If there are obvious clusters of related notes that aren't linked to each other: point them out.

**Next step:** One concrete `lens` command the user can run right now — the single most useful action based on graph state. Examples: `lens index add "topic" <best_entry_id>` if the topic has depth but no entry point; a specific `lens write` to create a missing connection; `lens list notes --orphans` if orphan rate is high.
