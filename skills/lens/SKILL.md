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
lens health --json                 # Knowledge graph health metrics
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
lens health --json
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

## Workflows

### Compile an article into knowledge

1. `lens fetch <url> --save --json` — get source_id + markdown
2. `lens search "key topics" --json` — find existing related notes
3. For top results: `lens show <id> --json` — read full detail + links
4. Think: what's genuinely new? What supports or contradicts existing notes?
5. Batch write your notes:
```bash
echo '[
  {"type":"note", "text":"your insight", "role":"claim", "qualifier":"likely", "source":"src_ID", "supports":["note_ID"]},
  {"type":"note", "text":"another thought", "role":"observation", "source":"src_ID"}
]' | lens write --json
```

### Answer a question from knowledge

1. `lens search "<query>" --json` — find relevant notes
2. `lens show <id> --json` for top results — get full detail
3. Synthesize the answer from notes. Cite note IDs (e.g. "According to note_01ABC...").

### Curate orphan notes

1. `lens health --json` — check `connectivity.orphan_count`
2. For orphan notes: `lens show <id> --json` — read the note
3. `lens search "related keywords" --json` — find potential connections
4. `echo '{"type":"link","from":"orphan_id","rel":"supports","to":"target_id"}' | lens write --json`

## When to Use lens

- User references prior research ("what do I know about...", "my notes on...")
- User says "check lens", "ask lens", or "save to lens"
- You discover something relevant to existing notes
- User asks to compile, ingest, or save an article
- User wants to build persistent knowledge across sessions

## Error Format

All errors return: `{"error": {"code": "...", "message": "...", "command": "..."}}`

The message explains what went wrong and how to fix it.

## Tips

- Always use `--json` when consuming output
- Note IDs: `note_01ABC...`, source IDs: `src_01ABC...`
- Search supports Chinese, Japanese, Korean text natively
- Contradicts links are always bidirectional (lens enforces this)
- Batch `$N` references resolve to the ID of the Nth item in the same batch
- `lens health` shows graph quality — high orphan rate means notes need linking
