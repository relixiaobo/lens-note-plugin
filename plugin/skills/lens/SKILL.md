---
name: lens
description: Store, query, and link knowledge in a persistent knowledge graph. Use when the user references prior research, asks "what do I know about...", wants to compile an article, or says "check lens". Install via npm install -g lens-note.
---

# lens — knowledge graph for agents

lens is a CLI that stores, queries, and links structured knowledge. Like Git for code, lens is for knowledge. You do the thinking; lens does the storage.

**Zero LLM dependency. Zero API keys.** lens never calls an LLM — you are the intelligence.

## Setup

Check if lens is installed: `which lens`

If not installed: `npm install -g lens-note` then `lens init`

## 5 Core Commands

```bash
lens search "<query>" --json       # Find notes (multilingual, CJK-aware)
lens show <id> --json              # Read one object with full detail + links
echo '<json>' | lens write --json  # Write anything: note, source, link, batch
lens fetch <url> [--save] --json   # Extract web content as clean markdown
lens status --json                 # Knowledge graph health metrics
```

## Reading Knowledge

```bash
# Search
lens search "agent architecture" --json
# → {query, count, results: [{id, type, text, role, qualifier, scope}]}

# Read full object
lens show note_01ABC --json
# → full object with all fields, evidence, metadata

# Graph health
lens status --json
# → {total_notes, connectivity: {orphan_count, ...}, link_types, ...}
```

## Writing Knowledge

Pipe JSON to `lens write --json`. The `type` field determines the operation.

### Create a note

```bash
echo '{
  "type": "note",
  "text": "High quality software is cheaper in the medium term",
  "role": "claim",
  "qualifier": "certain",
  "scope": "big_picture",
  "source": "src_01ABC",
  "supports": ["note_01DEF"],
  "evidence": [{"text": "the track overtakes within weeks", "source": "src_01ABC"}]
}' | lens write --json
# → {"id": "note_01XYZ", "type": "note", "action": "created"}
```

Note fields:
- `text` (required): the thought itself
- `role`: claim / frame / question / observation / connection / structure_note
- `qualifier`: certain / likely / presumably / tentative
- `scope`: big_picture / detail
- `voice`: extracted / restated / synthesized
- `source`: source ID this note comes from
- `supports`, `contradicts`, `refines`: arrays of note IDs
- `evidence`: array of `{text, source, locator}`
- `sees`, `ignores`: for frame notes
- `question_status`: open / tentative_answer / resolved
- `bridges`: array of note IDs (for connection notes)

### Create a source

```bash
echo '{"type": "source", "title": "Article Title", "url": "https://...", "source_type": "web_article"}' | lens write --json
```

### Add a link

```bash
echo '{"type": "link", "from": "note_01A", "rel": "supports", "to": "note_01B"}' | lens write --json
```

Contradicts links are automatically bidirectional.

### Update a note

```bash
echo '{"type": "update", "id": "note_01A", "set": {"qualifier": "certain"}, "add": {"supports": ["note_01B"]}}' | lens write --json
```

### Batch write

```bash
echo '[
  {"type": "source", "title": "Article X", "url": "https://..."},
  {"type": "note", "text": "Key insight", "role": "claim", "source": "$0"},
  {"type": "note", "text": "Supports first note", "supports": ["$1"], "source": "$0"}
]' | lens write --json
```

Use `$0`, `$1` to reference earlier items in the batch by index.

## Fetching Web Content

```bash
# Extract only
lens fetch https://example.com/article --json
# → {title, author, url, word_count, markdown}

# Extract + save as Source
lens fetch https://example.com/article --save --json
# → {title, author, url, word_count, markdown, source_id}
```

## How to Think (Read This First)

lens follows the Zettelkasten method. These principles determine the quality of everything you store:

**1. You are a thinker, not an extractor.** Don't summarize what the article says. Write YOUR thoughts triggered by the reading. "The article says X" is useless. "X contradicts what we know about Y, which suggests Z" is valuable.

**2. Search BEFORE you write.** Always search lens for existing knowledge on the topic before creating notes. The value is in connections — a note that links to existing knowledge is worth 10x more than an isolated one.

**3. Write concept-oriented notes, not source-oriented notes.** Bad: "Notes from Martin Fowler's article." Good: "High internal quality has negative cost in software development." The note should stand alone as a thought, independent of which article triggered it.

**4. Rewrite in your own words.** Don't copy quotes. Reformulate the idea. This forced rewriting is what makes knowledge stick. Use `voice: "synthesized"` for your own thinking, `"extracted"` only for verbatim evidence quotes.

**5. Contradictions and tensions are the most valuable links.** `contradicts` and `refines` create more knowledge value than `supports`. When you find a tension between a new idea and an existing note, that IS the insight.

**6. Update existing notes, don't just create new ones.** If a new article provides evidence for an existing claim, update it (`type: "update"`, add evidence, strengthen qualifier). If understanding evolves, update the note rather than creating a duplicate.

**7. Every link needs a reason.** Don't link just because topics are vaguely related. Ask: "why does this note specifically support/contradict/refine that one?" If you can't articulate it, don't link.

**8. Fewer, better notes.** An article that mostly covers known ground might produce 1 new note and 3 updates to existing notes. A breakthrough article might produce 5 genuine insights. The number follows from thinking, not from a target.

## Workflows

### Compile an article into knowledge

1. `lens fetch <url> --save --json` — get source_id + markdown
2. **Read the article.** Identify 2-3 key themes.
3. `lens search "theme keywords" --json` — find what the knowledge graph already knows
4. For relevant results: `lens show <id> --json` — read full detail + links
5. **Think**: What's genuinely new? What contradicts existing notes? What strengthens them?
6. For existing notes that get new evidence: use `type: "update"` to add evidence or strengthen qualifier
7. For genuinely new insights: create notes with links to existing ones
```bash
echo '[
  {"type":"update", "id":"note_01EXISTING", "add":{"evidence":[{"text":"new supporting quote","source":"src_NEW"}]}},
  {"type":"note", "text":"Your genuine new insight", "role":"claim", "qualifier":"likely", "source":"src_NEW", "contradicts":["note_01OTHER"]}
]' | lens write --json
```

### Answer a question from knowledge

1. `lens search "<query>" --json` — find relevant notes
2. `lens show <id> --json` for top results — read full detail + links
3. Synthesize the answer from notes. Cite note IDs as evidence.
4. Surface contradictions: if the knowledge graph has conflicting notes, present both sides.
5. Identify gaps: if the query touches areas with no notes, say so explicitly.

### Curate orphan notes

1. `lens status --json` — check `connectivity.orphan_count`
2. For orphan notes: `lens show <id> --json` — read the note
3. `lens search "related keywords" --json` — find potential connections
4. Only add links you can justify: `echo '{"type":"link","from":"orphan_id","rel":"supports","to":"target_id"}' | lens write --json`

## When to Use lens

- User references prior research ("what do I know about...", "my notes on...")
- User says "check lens", "ask lens", or "save to lens"
- User asks to compile, ingest, or save an article

## What IS Worth Storing

- An insight that connects to ideas already in the knowledge graph
- A principle you'd apply in a different context in the future
- A tension or contradiction between two ideas that's unresolved
- A genuinely new perspective that none of the existing notes express

**The test**: would this note surprise and help you if you found it 3 months from now while working on something unrelated? If not, don't store it.

## What is NOT Worth Storing

- Summaries of what the article says (store your thinking, not its content)
- Debug solutions and tool-specific workarounds (they expire)
- Common knowledge (if everyone already knows it, it won't surprise you)
- Process logs (what you did today, steps taken)
- Facts without interpretation (store the insight, not the data)

## Error Format

All errors return: `{"error": {"code": "...", "message": "...", "command": "..."}}`

The message explains what went wrong and how to fix it.

## Tips

- Always use `--json` when consuming output
- Note IDs: `note_01ABC...`, source IDs: `src_01ABC...`
- Search supports Chinese, Japanese, Korean text natively
- Contradicts links are always bidirectional (lens enforces this)
- Batch `$N` references resolve to the ID of the Nth item in the same batch
- `lens status` shows graph quality — high orphan rate means notes need linking
