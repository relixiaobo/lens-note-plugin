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

Crystallization takes many forms:

| What happened | What to do |
|--------------|------------|
| Genuinely new insight | Create a note with links to what it collided with |
| Strengthened existing understanding | Update: add evidence to body, strengthen the claim |
| Found a contradiction | Create a note with `contradicts` link. Explain the tension. |
| Two old notes are actually the same | Merge: update the better one, delete the other |
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
5. At each collision point: crystallize.
6. For existing notes that gain new evidence: write update via `lens --stdin`
7. For new insights: write note via `lens --stdin`
8. Zero new notes is acceptable. An article that only strengthens existing knowledge produces updates, not new notes.

## Anti-Patterns

- **Extraction**: creating a note for every paragraph of the article
- **Source-oriented notes**: notes that only make sense if you name the source
- **Linking because you can**: vague associations are noise, not structure
- **Always creating new**: an article that covers known ground should produce updates
- **Smoothing over contradictions**: tension is insight. Don't resolve it prematurely.
- **Pre-building structure**: structure notes are sparse post-hoc index entries, created after clusters form naturally. Never one per article.
