---
name: lens
description: Store, query, and link knowledge in a persistent knowledge graph. Use when the user references prior research, asks "what do I know about...", wants to compile an article, or says "check lens". Install via npm install -g lens-note.
---

# lens — knowledge graph for agents

lens stores, queries, and links structured knowledge. You do the thinking; lens does the storage.

## Setup

Check if lens is installed: `which lens`
If not installed: `npm install -g lens-note` then `lens init`

## How You Must Think

These rules determine quality. Violating them produces a noisy, useless knowledge graph.

**1. One note = one independent idea.** If a note contains multiple claims, split it before writing. Each note must be understandable without its source.

**2. You are a thinker, not an extractor.** "The article says X" is forbidden. "X contradicts what we know about Y, which suggests Z" is valuable. If your note can only be described as "from article X," reject or rewrite it as a concept-oriented claim.

**3. Search BEFORE you write.** Always `lens search` for existing knowledge before creating notes. A note that links to existing knowledge is worth 10x more than an isolated one.

**4. Rewrite in your own words.** Don't copy quotes. Reformulate the idea. Use `voice: "synthesized"` for your own thinking, `"extracted"` only for verbatim evidence inside the `evidence` array.

**5. Contradictions and tensions are the most valuable.** `contradicts` and `refines` create more knowledge value than `supports`. Actively look for tensions between new ideas and existing notes.

**6. Update before create.** If a new article provides evidence for an existing claim, use `type: "update"` to add evidence or strengthen qualifier. If understanding evolves, update the existing note. Only create a new note for a genuinely new idea.

**7. Every link needs a reason.** Don't link because topics are vaguely related. Automatic backlinks and loose associations are harmful noise. Ask: "why does this note SPECIFICALLY support/contradict/refine that one?"

**8. Zero new notes is acceptable.** An article that only strengthens existing knowledge should produce updates, not new notes. The number of notes follows from thinking, not from a target.

**9. Structure notes are sparse and post-hoc.** Don't create `structure_note` entries for every topic. They are index entry points created AFTER a cluster of linked notes has formed naturally. Never create one per article.

**10. Merge, evolve, supersede.** When two notes say essentially the same thing, merge them. When confidence changes, evolve the qualifier. When a note is replaced by better understanding, supersede it (`status: "superseded"`). Notes have lifecycles, not just births.

## When to Use lens

- User references prior research ("what do I know about...", "my notes on...")
- User says "check lens", "ask lens", or "save to lens"
- User asks to compile, ingest, or save an article

## What IS Worth Storing

- An insight that connects to ideas already in the knowledge graph
- A principle you'd apply in a different context in the future
- A tension between two ideas that's unresolved
- A genuinely new perspective that none of the existing notes express

**The test**: would this note surprise you 3 months from now while working on something unrelated? If not, don't store it.

## What is NOT Worth Storing

- Summaries of what the article says (store your thinking, not its content)
- Debug solutions and tool-specific workarounds (they expire)
- Common knowledge (if everyone knows it, it won't surprise you)
- Process logs (what you did today)
- Facts without interpretation
- A note that cannot stand without naming its source article

## Workflows

### Compile an article

1. `lens fetch <url> --save --json` → get source_id + markdown
2. Read the article. Rewrite candidate ideas in your own words.
3. `lens search "concept keywords" --json` → find what lens already knows
4. For relevant results: `lens show <id> --json` → read detail + links
5. Test each candidate idea against existing notes:
   - Strengthens an existing note? → `type: "update"`, add evidence
   - Contradicts an existing note? → create new note with `contradicts` link
   - Genuinely new? → create new note with links to related existing notes
   - Already covered? → skip
6. 0 new notes is a valid outcome.

### Answer a question from knowledge

1. `lens search "<query>" --json` → find relevant notes
2. `lens show <id> --json` for top results → full detail + links
3. Synthesize. Cite note IDs as evidence.
4. Surface contradictions: present both sides if the graph has conflicting notes.
5. Identify gaps: say explicitly if the query touches areas with no notes.

### Curate orphan notes

1. `lens status --json` → check `connectivity.orphan_count`
2. `lens show <orphan_id> --json` → read the note
3. `lens search "related keywords" --json` → find connections
4. Only add links you can justify. No link is better than a weak link.

## Command Reference

### 5 Core Commands

```bash
lens search "<query>" --json       # Find notes (multilingual)
lens show <id> --json              # Read one object with detail + links
echo '<json>' | lens write --json  # Write anything (see below)
lens fetch <url> [--save] --json   # Extract web content
lens status --json                 # Stats + graph health
```

### Write API

Pipe JSON to `lens write --json`. The `type` field routes the operation:

```json
{"type": "note", "text": "...", "role": "claim", "qualifier": "likely", "supports": ["note_ID"]}
{"type": "source", "title": "...", "url": "...", "source_type": "web_article"}
{"type": "link", "from": "note_A", "rel": "supports", "to": "note_B"}
{"type": "update", "id": "note_A", "set": {"qualifier": "certain"}, "add": {"evidence": [...]}}
{"type": "delete", "id": "note_A"}
[{...}, {...}]  // batch, use $0/$1 to reference earlier items
```

Note fields: `text` (required), `role` (claim/frame/question/observation/connection/structure_note), `qualifier` (certain/likely/presumably/tentative), `scope` (big_picture/detail), `voice` (extracted/restated/synthesized), `source`, `supports[]`, `contradicts[]`, `refines[]`, `evidence[]`, `sees`, `ignores`, `question_status`, `bridges[]`.

Contradicts links are automatically bidirectional.

### Error Format

`{"error": {"code": "...", "message": "...", "command": "..."}}`
