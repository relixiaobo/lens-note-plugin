---
description: Recall what you think about a topic — synthesized from your knowledge graph
argument-hint: '<topic>'
allowed-tools: Bash(lens:*)
---

The user wants to know what they think about: $ARGUMENTS

**Gather**: Search the graph from multiple angles — keyword index, full-text search with bodies, and link-walking from top hits. Read enough notes to form a real picture, not just list titles.

```bash
lens index "$ARGUMENTS" --json
lens search "$ARGUMENTS" --expand --json
```

For the most relevant hits, follow their links to see the full neighborhood.

**Synthesize**: Write like a smart friend who has read all the user's notes. Plain language, short sentences.

- **Your take on [topic]**: What does the user actually think? Be specific — not "you've explored this" but "you think X because Y." Cite by note title.
- **The unresolved bit**: If contradicting notes exist, state the conflict plainly. Which side is stronger?
- **The loose end**: 1-2 ideas that haven't connected to anything yet — with a specific action for each (a note worth writing, a connection worth making).
- **Graph status**: N notes found, M orphans, whether a keyword index entry exists. If depth is sufficient but no index entry: suggest one.
- **Next step**: One concrete `lens` command — the single most useful action based on graph state.
