---
description: Reflect on recent learning — where you've been and where to go next
argument-hint: '[week|month|year]'
allowed-tools: Bash(lens:*)
---

Reflect on recent knowledge activity. Period: $ARGUMENTS (default: week if empty)

**Gather**: Run digest and check for orphans and open tasks.

```bash
lens digest $ARGUMENTS --json
lens list notes --orphans --limit 5 --json
lens list tasks --status open --json
```

For any orphan notes, read them (`lens show`) to understand what they're about.

**Synthesize**: Write like you're catching up with a friend. Say what you actually see.

- **Lead with one observation** — the most interesting thing that happened in the user's thinking this period. Not "the graph shows increased connectivity" but something real.
- **Tensions** — for each `contradicts` note in the digest, state the conflict in plain language. Skip if none.
- **Seeds wanting connection** — for each orphan, suggest one specific connection based on its content. Limit to 3.
- **Gained evidence** — if the digest shows existing notes that received new `supports`, highlight the strongest (their position is now better-supported).
- **Open tasks** — list titles if any exist. Skip if none.
- **Direction** — what single thread is most worth following next, and why?
