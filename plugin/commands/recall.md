---
description: Search your knowledge graph and surface relevant notes
argument-hint: '<query>'
allowed-tools: Bash(lens:*)
---

Search the user's knowledge graph for: $ARGUMENTS

1. Check keyword index first: `lens index "$ARGUMENTS" --json`
   - If entries exist, note those IDs — they are the best entry points for this topic
2. Search: `lens search "$ARGUMENTS" --json`
3. For the top 3 results, read each: `lens show <id> --json`
4. Present results in a compact, scannable format:
   - **Title** (`id`) — one-line summary from body, link count
   - List key connections (what it supports / contradicts / refines)
5. If results include contradicting notes, highlight the tension explicitly.
6. If no results found, say so clearly. Do not suggest creating notes unprompted.
