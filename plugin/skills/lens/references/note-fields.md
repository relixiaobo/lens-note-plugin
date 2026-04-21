# Note Fields Reference

## Frontmatter (7 fields)

| Field | Required | Description |
|-------|----------|-------------|
| `id` | auto | Unique ID (note_ULID) |
| `type` | auto | Always "note" |
| `title` | yes | The thought in one sentence |
| `source` | no | Source ID this note comes from |
| `links` | no | Array of relationships (see below) |
| `created_at` | auto | ISO timestamp |
| `updated_at` | auto | ISO timestamp |

## Links

Each link has:
- `to`: target object ID — any `note_`, `src_`, or `task_` prefix (required)
- `rel`: relationship type (required)
- `reason`: why this link exists (optional but recommended)

```json
{"to": "note_01ABC", "rel": "contradicts", "reason": "AI changes the cost equation"}
```

### Link types

| Rel | Meaning | Auto-bidirectional? |
|-----|---------|---------------------|
| `supports` | Strengthens the target note | No |
| `contradicts` | Conflicts with the target note | Yes |
| `refines` | More precise version of the target | No |
| `continues` | Next step in the target's line of thought (Folgezettel chain) | No |
| `related` | Loose association — **requires reason** (use as last resort) | No |

**Choosing a rel:** Try `contradicts` → `refines` → `supports` → `continues` first. Only use `related` when none fit, and always provide a `reason` that explains HOW (not just topic overlap). `related` without a reason is rejected by the CLI. For topic organization (MOC-style grouping), use a whiteboard (`lens board`) instead of a typed link.

## Body (markdown after frontmatter)

Everything that isn't the title goes in the body:
- **Evidence**: use markdown blockquotes (`> "quote" — Source`)
- **Confidence**: state in prose ("**likely** based on 2 sources")
- **Scope**: mention if it's a big-picture principle or supporting detail
- **Perspective**: describe what this view sees and ignores
- **Questions**: pose open questions in the body text
- **Inline references**: use `[[note_ID]]`, `[[src_ID]]`, or `[[task_ID]]` to reference objects in prose

The body is free-form markdown. No required structure. Write naturally.

**Inline reference convention**: Use `[[note_01ABC]]`, `[[src_01ABC]]`, or `[[task_01ABC]]` in body text. On read (`lens show`, `search --expand`), JSON output returns body unchanged plus `body_refs: [{id, title}]` with resolved titles. Text output resolves `[[ID]]` → `[Title](ID)` inline. This is for readability only — it does not create a graph edge. To create an actual connection, add an entry to `links[]`.

## Source Fields

| Field | Required | Description |
|-------|----------|-------------|
| `id` | auto | Unique ID (src_ULID) |
| `type` | auto | Always "source" |
| `title` | yes | Article/document title |
| `source_type` | no | book, paper, report, video, podcast, course, web_article, newsletter, social_post, conversation, manual_note, note_batch, markdown, plain_text |
| `url` | no | Original URL |
| `author` | no | Author name |
| `word_count` | auto | Word count |
| `created_at` | auto | ISO timestamp |

## Task Fields

| Field | Required | Description |
|-------|----------|-------------|
| `id` | auto | Unique ID (task_ULID) |
| `type` | auto | Always "task" |
| `title` | yes | What needs to be done |
| `status` | no | `open` (default) or `done` |
| `source` | no | Note or source ID that prompted this task |
| `links` | no | Array of relationships |
| `created_at` | auto | ISO timestamp |
| `updated_at` | auto | ISO timestamp |

## Batch References

In batch writes (`[{...}, {...}]`), use `$0`, `$1` to reference the ID of earlier items by array index.
