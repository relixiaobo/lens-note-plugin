---
description: Find surprising cross-domain connections using LLM judgment
argument-hint: '<note title or id>'
allowed-tools: Bash(lens:*)
---

Find surprising connections for: $ARGUMENTS

**Goal**: Go beyond word overlap. The TF-IDF candidates from `discover --collide` share rare terms but miss structural isomorphisms — your job is to read both sides and judge whether a genuine cross-domain connection exists.

**Gather**: Resolve the target, then get collision candidates (up to 10). Read both notes for each candidate pair.

**Judge each pair on**:
- **Structural isomorphism**: same underlying mechanism in different domains?
- **Shared constraint**: same fundamental trade-off or limitation?
- **Transferable solution**: one domain's answer applies to the other's problem?

Skip surface-level topic overlap. If the only connection is "both mention X," it's not a collision.

**Present**: For each genuine connection (aim for 1-3):
- What they share — one sentence on the structural link, not topic overlap
- Why it matters — what new insight emerges from seeing this connection

If no genuine connections found, say so honestly.

**Link**: Offer to create `related` links with the structural insight as reason. Do not create without user confirmation — the surprise needs human judgment.
