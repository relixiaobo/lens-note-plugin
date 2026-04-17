---
description: Discover hidden contradictions in your knowledge graph
argument-hint: '[topic]'
allowed-tools: Bash(lens:*)
---

Find contradictions in the knowledge graph. Topic: $ARGUMENTS

**Goal**: `contradicts` links are the most valuable edges in the graph but the hardest to find. Surface thesis pairs that make incompatible claims.

**Gather**: If a topic is given, search with `--expand`. Otherwise, list thesis-level notes (`--min-links 3`). Read the top candidates — focus on notes with assertion titles, not observations.

**What counts as a tension**:
- Direct opposition: "use small context" vs "large context enables better reasoning"
- Incompatible assumptions: one assumes X is cheap, another assumes X is expensive
- Boundary conditions: both right in different contexts — the tension is about where the line falls
- Evolution: earlier note says X, later note says not-X — thinking has shifted

**What does NOT count**:
- Different aspects of the same topic (not contradictory)
- One is more specific than the other (that's `refines`)
- Both can be simultaneously true (no real conflict)

**Present**: For each genuine tension (aim for 2-5):
- One-sentence summary of the conflict
- Side A: [title] — what it claims
- Side B: [title] — what it claims
- The crux: where exactly they disagree

Check `lens links` first — skip pairs that already have a `contradicts` link.

**Link**: Offer to create `contradicts` links. The reason should capture the crux, not just "they disagree." Ask before writing.
