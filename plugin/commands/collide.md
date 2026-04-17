---
description: Find surprising cross-domain connections using LLM judgment
argument-hint: '<note title or id>'
allowed-tools: Bash(lens:*)
---

Find surprising connections for: $ARGUMENTS

**Step 1 — Resolve the target note:**

```bash
lens search "$ARGUMENTS" --resolve --json
```

If ambiguous, pick the best match. Show the user which note you're working with.

**Step 2 — Get collision candidates:**

```bash
lens discover <id> --collide --count 10 --json
```

This returns notes from outside the target's connected component, ranked by TF-IDF similarity. The scores will be low (5-15%) — that's expected. Your job is to find the ones with genuine structural connections that word overlap alone can't detect.

**Step 3 — Deep assessment (the LLM step):**

For each candidate (up to 10), read both notes:

```bash
lens show <target_id> <candidate_id> --json
```

For each pair, assess:
- **Structural isomorphism**: Do they describe the same underlying mechanism in different domains? (e.g., "progressive disclosure in UI" and "support vector machines" both separate complexity by boundaries)
- **Shared constraint**: Do they face the same fundamental trade-off or limitation?
- **Transferable solution**: Does one domain's solution apply to the other's problem?

Skip candidates where the only connection is surface-level topic overlap.

**Step 4 — Present results:**

For each genuine connection found (aim for 1-3), present:

**[Target title] x [Candidate title]**
What they share: one sentence explaining the structural connection — not "both are about X" but "both solve Y by doing Z."
Why it matters: what new insight or question emerges from seeing this connection.

If no genuine connections are found among the 10 candidates, say so honestly — "The graph's disconnected notes don't share structural patterns with this one. This might mean the note is in a well-explored domain, or that the interesting connections require a different starting point."

**Step 5 — Offer to link:**

For each genuine connection, offer to create a `related` link with the structural insight as the reason. Ask the user which ones to create.

```bash
lens write --file /tmp/collide-links.json --json
```

Do not create links without user confirmation — the value of collide is in the surprise, and the user needs to judge whether the connection is genuine.
