# The Collision Method

Knowledge grows through collision, not through collection.

## The Cycle: Spark → Collide → Crystallize

### Spark

A thought arrives. It can come from anywhere:
- An article you're reading
- A conversation you're having
- A sudden insight
- Reviewing existing notes
- Practicing something (writing code, running an experiment)

The source doesn't matter. What matters is: you now have a thought that wants to meet your existing knowledge.

### Collide

Bring your thought to the knowledge graph. This is where value is created.

```bash
lens search "key concepts from your thought" --json
```

When you find related notes, **don't just read them — wander.** Look at their links. Follow them. See what THOSE notes connect to.

```bash
lens show note_01ABC --json    # Read it. See its links.
lens show note_01DEF --json    # Follow a link. What's here?
```

This wandering is not aimless. You're carrying your thought through the graph, watching what happens when it meets different existing ideas. The graph talks back.

What to notice during collision:
- **Agreement**: your thought strengthens something you already know
- **Tension**: your thought conflicts with an existing note — THIS is the most valuable moment
- **Gap**: your thought enters territory with no existing notes
- **Surprise**: you discover a connection between your thought and a note you didn't expect

### Crystallize

Write what emerged from the collision. Not what the article said. Not what you already knew. The NEW understanding that neither had alone.

**Cluster Check — before linking multiple notes to the same target:**

If ≥ 3 notes from this session would link to the same existing target node, AND the target already has > 20 inbound links (check the `advisory` on a `{"type": "link"}` write response, or run `lens links <id> --direction backward --json` to count) — **pause before linking**. Track this yourself across your writes in the session; the CLI only warns at the individual link level.

Ask: do these new notes share a thematic sub-focus that isn't already captured in the graph?

| If yes — use chain topology | If no — link directly |
|-----------------------------|----------------------|
| Create an L2 synthesis note for the sub-focus | Each note evidences the target thesis directly — proceed |
| New notes → `supports` → L2 | |
| L2 → `refines` → master | |

**Example:** reading a book on leadership generates 12 notes all targeting "Leadership and Decision-Making" — 8 are about how decisions get made under uncertainty, 4 about how leaders communicate rationale to teams. Create "Decision-Making Under Uncertainty" (L2), redirect the 8 decision notes there, L2 `refines` "Leadership and Decision-Making".

**When to skip the L2:** target is a narrow specific thesis (≤ 15 inbound), or you can't name the sub-focus in a single phrase.

**If the write response contains `advisory.warning_code == "approaching_super_connector"`:** check `advisory.is_healthy_hub`. If `false` and `advisory.target_inbound_count` is approaching 30 — apply chain topology for remaining links.

---

**Before linking, classify the collision:**

| Collision type | What happened | Link to create |
|---------------|---------------|----------------|
| **Evidential** | This note proves or demonstrates the thesis in an existing note | `supports` |
| **Hierarchical** | This note is a concrete case or implementation of an existing note | `refines` |
| **Topical** | Both notes cover the same territory, but neither proves the other | `related` (write a reason explaining HOW they connect) |
| **Contradictory** | This note conflicts with an existing note | `contradicts` |
| **Weak** | Connection is interesting but you can't articulate why | No link — leave as seed |

**The test for `supports`:** Can you honestly complete this sentence? *"[This note] provides specific evidence that [target thesis] because..."* If the honest ending is "both are about X" or "both mention Y" — that's `related`, not `supports`.

**Not linking is valid.** A note that enters new territory with no strong connections is a seed. Seeds find connections in future collisions when more context exists. A well-reasoned `related` link is better than a vague `supports`.

Crystallization takes many forms:

| What happened | What to do |
|--------------|------------|
| Genuinely new insight | Classify the collision type first, then create the note with the right link |
| Strengthened existing understanding | Update: add evidence to body, strengthen the claim |
| Found a contradiction | Create a note with `contradicts` link. Explain the tension. |
| Two old notes are actually the same | Merge: `write {"type":"merge","from":"dup","into":"keep"}` |
| Old understanding superseded | Update body to mark as superseded, link to the newer note |
| Multiple insights emerged | Multiple notes + links between them |
| Nothing new after collision | Do nothing. This is a valid outcome. |

## When Collision Isn't Deep Enough

Sometimes you collide and nothing happens — the encounter is too surface-level. Four natural moves to go deeper:

**Break apart** — The concept is too big or vague. Split it. Look at it from different angles. What does it assume? What would it look like from the opposite perspective?

**Drill down** — You have a "so what?" feeling. Keep asking: why is this true? What's underneath? What would break if this were false? Don't stop until you reach something solid.

**Reduce** — Too many dimensions, too much complexity. Strip it down: what are the 2-3 things that actually matter? Everything else follows from those.

**Debate** — Your thought and an existing note genuinely disagree. Don't smooth it over. Let both sides speak. The tension IS the insight. Write it as a `contradicts` link with your analysis of where the disagreement lives.

## The Shape of Good Crystallization

Each crystal (note) should be:

- **One idea** — If it contains multiple claims, split it. A clear thought holds one thing.
- **Independent** — Understandable without knowing which article triggered it. "High internal quality has negative cost in software" not "Fowler's article argues that..."
- **In your words** — Reformulated, not copied. You discover what you think by trying to express it.
- **Placed** — Linked to the notes it collided with. The link IS the context. Without it, the note is an orphan — a seed waiting for a future collision.

## What Goes in the Body?

The body is free-form markdown. Use it for everything beyond the title:

- **Evidence**: use blockquotes — `> "quote" — Source`
- **Confidence**: state in prose — "**likely** based on 2 sources"
- **Scope**: mention if it's a big-picture principle or supporting detail
- **Perspective/frames**: describe what this view sees and what it ignores. Frames are among the most valuable notes — they don't add facts, they change how you interpret facts.
- **Questions**: pose open questions naturally
- **Inline references**: use `[[note_ID]]` to reference other notes (resolved to titles on read)

## Process for Deep Compilation (Articles)

When the spark is a full article:

1. `lens fetch <url> --save --json` — Get the content and register the source.
2. Read the article fully. Don't start writing yet.
3. Identify your strongest reactions — what surprised you, what you disagreed with, what connected to something you know.
4. For each reaction: carry it into the graph. `lens search` → `lens show` → follow links → wander.
5. At each collision point: classify the collision type, then crystallize.
6. For existing notes that gain new evidence: write update via `lens --stdin`
7. For new insights: write note via `lens --stdin`
8. Zero new notes is acceptable. An article that only strengthens existing knowledge produces updates, not new notes.

**Two-pass for large sessions (>4 new notes):** When an article generates many notes, split the work into two passes:

- **Pass 1 — Write:** Create all notes, linking each only to the source. No inter-note links yet.
- **Pass 2 — Connect:** Once all notes exist, review them together. With the full session picture visible, classify each collision and create inter-note links.

This prevents the common failure mode of creating topic-proximity `supports` links during writing, when you have the least context about how the notes relate to each other.

## Anti-Patterns

- **Extraction**: creating a note for every paragraph of the article
- **Source-oriented notes**: notes that only make sense if you name the source
- **Linking because you can**: vague associations are noise, not structure. Not linking is a valid outcome.
- **Always creating new**: an article that covers known ground should produce updates
- **Smoothing over contradictions**: tension is insight. Don't resolve it prematurely.
- **Pre-building structure**: structure notes are sparse post-hoc index entries, created after clusters form naturally. Never one per article.
- **Linking during the writing pass**: for sessions producing many notes, creating inter-note links while writing risks topic-proximity `supports` links. Write first, connect after — you'll have full context in Pass 2.
- **Star topology**: linking all new notes directly to one master synthesis node instead of creating thematic L2 nodes first. A write-time `advisory` fires at > 20 inbound (on `{"type": "link"}` writes); `lens lint` warns at > 30 (`super_connectors` check). A target is only a legitimate hub when `is_healthy_hub: true` — it has inbound `indexes` links and `indexes >= supports`. Otherwise apply chain topology (Cluster Check above) when multiple notes share a sub-focus.
