# Feed Processing

How to handle new articles from RSS feeds. The key decision: is this worth compiling, or just saving?

## The Flow

```
lens feed check --json     → list of new articles (title + URL)
        ↓
lens fetch <url> --save    → save each as Source (cheap: HTTP + extraction)
        ↓
SCENT scan                 → score each Source against the graph
        ↓
High score → compile       → Collision Method → Notes
Low score  → skip (Source stays for future reference)
```

Fetching is cheap. Compilation is expensive. The decision point is "compile or not", not "fetch or not".

## Deciding What to Compile

After fetching, read each source and ask four questions:

| Question | How to check |
|----------|--------------|
| Does it **surprise** you — contradict or complicate something you already think? | `lens search "key claim"` — do results disagree? |
| Does it **connect** to 2+ existing notes, or bridge separate areas? | Search multiple key terms — do results span different topics? |
| Is it **substantial** — dense enough to compress into real insights? | Is it substantive or mostly filler? |
| Does it cover **new ground** you don't have yet? | `lens search "new concept"` — zero results means new territory |

### Decision

- **Surprise + Connection** → Compile. That's where the collision is.
- **Just familiar ground presented well** → Skip. Confirmation doesn't produce new knowledge.
- **Genuinely new topic but nothing to connect to yet** → Save as a seed. It may matter when related knowledge arrives.
- **Confirms what you already know** → Skip.

## Compilation

When you decide to compile, follow the [Collision Method](compilation.md):

1. Read the full Source body.
2. Identify your strongest reactions — surprise, disagreement, connection to something you know.
3. For each reaction: `lens search` → `lens show` → follow links → wander.
4. At each collision point: crystallize a Note.
5. Link new Notes to the Source and to existing Notes they collided with.

## Practical Example

```bash
# 1. Check feeds
lens feed check --json

# 2. Fetch all new articles as Sources
# (for each article URL from step 1)
lens fetch "https://example.com/article" --save --json

# 3. Read the Source, scan for SCENT
lens show src_01ABC --json
# → Read the body. Note key claims and concepts.

# 4. Search the graph for collision potential
printf '%s' '{"command":"search","positional":["key concept from article"]}' | lens --stdin
# → High Surprise or Connection? → Compile.
# → Nothing interesting? → Move to next article.

# 5. Compile (if worth it)
printf '%s' '{"command":"write","input":{"type":"note","title":"Insight from collision","source":"src_01ABC","body":"...","links":[{"to":"note_01DEF","rel":"contradicts","reason":"..."}]}}' | lens --stdin
```

## Rules

1. **Fetch everything, compile selectively.** Sources are cheap storage. Notes require thought.
2. **The graph is the filter.** An article's value depends on what you already know, not on the article alone.
3. **Surprise over confirmation.** Articles that challenge existing notes are more valuable than those that agree.
4. **Zero compilation is valid.** An article that adds nothing new to the graph should produce zero notes.
