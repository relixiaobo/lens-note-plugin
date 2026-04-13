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
- `to`: target note ID (required)
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
| `related` | Loose association (use sparingly) | No |

## Body (markdown after frontmatter)

Everything that isn't the title goes in the body:
- **Evidence**: use markdown blockquotes (`> "quote" — Source`)
- **Confidence**: state in prose ("**likely** based on 2 sources")
- **Scope**: mention if it's a big-picture principle or supporting detail
- **Perspective**: describe what this view sees and ignores
- **Questions**: pose open questions in the body text
- **Inline references**: mention other note IDs naturally in prose

The body is free-form markdown. No required structure. Write naturally.

## Source Fields

| Field | Required | Description |
|-------|----------|-------------|
| `id` | auto | Unique ID (src_ULID) |
| `type` | auto | Always "source" |
| `title` | yes | Article/document title |
| `source_type` | no | web_article, markdown, plain_text, manual_note |
| `url` | no | Original URL |
| `author` | no | Author name |
| `word_count` | auto | Word count |
| `created_at` | auto | ISO timestamp |

## Batch References

In batch writes (`[{...}, {...}]`), use `$0`, `$1` to reference the ID of earlier items by array index.
