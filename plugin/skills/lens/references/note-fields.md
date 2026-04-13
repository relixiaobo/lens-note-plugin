# Note Fields Reference

Complete reference for all fields when creating or updating notes.

## Required

- `text`: The thought itself. Must be an independent idea, understandable without context.

## Role (soft hint for display)

| Role | When to use | Key fields |
|------|------------|------------|
| `claim` | A substantiated assertion with evidence | evidence[], qualifier |
| `frame` | A perspective that reveals and hides | sees, ignores, assumptions[] |
| `question` | An unresolved inquiry | question_status |
| `observation` | A thought without evidence (default) | — |
| `connection` | Bridges two existing notes | bridges[] |
| `structure_note` | Sparse index entry point | entries[] |

Role is a display hint, not a constraint. A note can have evidence (claim) AND sees/ignores (frame) simultaneously.

## Claim Fields (Toulmin structure)

- `evidence[]`: Array of `{text, source, locator}`. Verbatim supporting quotes.
  - `text`: The exact quote
  - `source`: Source ID (e.g. `src_01ABC`)
  - `locator`: Optional position reference (page, section)
- `qualifier`: Confidence level
  - `certain`: Multiple independent sources confirm
  - `likely`: Good evidence, some room for doubt
  - `presumably`: Reasonable inference, limited evidence
  - `tentative`: Speculative, needs more evidence
- `voice`: How the note was produced
  - `synthesized`: Your own thinking (preferred)
  - `restated`: Rephrased from source
  - `extracted`: Direct from source (use sparingly)

## Frame Fields

- `sees`: What this perspective reveals
- `ignores`: What this perspective overlooks
- `assumptions[]`: What it takes for granted

## Question Fields

- `question_status`: `open` | `tentative_answer` | `resolved` | `superseded`

## Hierarchy (Reif/Miller)

- `scope`: `big_picture` (core insight) | `detail` (supporting point)
  - Big-picture notes linked to detail notes form natural hierarchy through supports links
  - No folders or containers needed

## Link Fields

- `supports[]`: Note IDs this note strengthens
- `contradicts[]`: Note IDs this note conflicts with (auto-bidirectional)
- `refines[]`: Note IDs this note is a more precise version of
- `related[]`: Array of `{id, note}` for loose associations (use sparingly)
- `bridges[]`: Note IDs being connected (for connection notes)
- `entries[]`: Note IDs serving as entry points (for structure notes)

## Provenance

- `source`: Source ID this note was compiled from
- `status`: `active` | `superseded`

## Batch References

In batch writes (`[{...}, {...}]`), use `$0`, `$1` to reference the ID of earlier items by array index.
