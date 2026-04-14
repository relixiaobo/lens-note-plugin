# Importing from Other Knowledge Bases

How to migrate notes from Tana, Roam Research, Obsidian, or any other tool into lens.

## Contents

- [The Core Principle](#the-core-principle)
- [Universal Flow](#universal-flow)
- [Format-Specific Guidance](#format-specific-guidance)
- [Decision Framework](#decision-framework)
- [Writing the Batch](#writing-the-batch)
- [Adding Links During Import](#adding-links-during-import)
- [Post-Import Audit](#post-import-audit)
- [What Not to Import](#what-not-to-import)

## The Core Principle

Most knowledge base exports are 80% noise: daily logs, stubs, task lists, half-finished
thoughts, system nodes. The goal of import is not to copy everything — it is to find
which notes are worth bringing in and letting them meet your existing knowledge.

A 90% reduction is normal and healthy. Import selectively.

## Universal Flow

```
1. Assess    → format, size, rough quality signal
2. Clean     → parse raw format, extract {title, body} pairs, filter noise
3. Preview   → show user: how many candidates, sample titles, breakdown by category
4. Decide    → per item: import as note, import as source, or skip
5. Write     → lens write --file <batch.json> --json (batches of ≤50)
6. Curate    → post-import audit (see curation.md)
```

## Format-Specific Guidance

### Tana (JSON export)

Tana exports one large JSON file. Categories by tag:

| Tag / condition | Import as |
|---|---|
| `#article` or `#source` (has URL) | Source via `lens fetch <url> --save` |
| `#article` or `#source` (no URL) | Source manually |
| `#highlight` | Note (evidence/quote) |
| Node with 2+ children | Note (reflection) |
| Short standalone node | Skip |

The raw format is complex (docs + datoms + tuple types). Use the pre-built parser
(located in the lens project root — run from the lens repo directory):

```bash
npx tsx spike/tana-clean.ts <export.json> /tmp/tana-clean.jsonl
```

Output: one JSON object per line — `{title, body, tana_id, category, word_count, url?}`.

### Roam Research (EDN backup)

Roam stores a Datascript database as EDN. Use the pre-built parser
(located in the lens project root — run from the lens repo directory):

```bash
npx tsx spike/roam-clean.ts <backup.edn> /tmp/roam-clean.jsonl
```

Output: `{title, body, roam_uid, category, word_count}`. Categories: `reflection`, `thought`, `daily`.

Skip `daily` notes unless `word_count > 100` and the body contains a real claim.

### Obsidian / markdown files

Plain markdown — no parser needed. Read directly. Supports both a single `.md` file
and a vault directory:

```bash
# Single file
cat <file.md>

# Vault directory (recursive — Obsidian vaults have subfolders)
find <vault-directory> -name '*.md' -type f
```

Filter heuristics:
- Skip files under 50 words
- Skip files where every line starts with `- [ ]` (pure task lists)
- Skip templates, MOC files, and system notes (often named with `_` prefix)
- Wikilinks `[[note title]]` → convert to plain text or strip

### Generic text / CSV / other

For any format: extract title and body per entry, apply the decision framework below.
If the format is complex, write a small cleaning script first — output JSONL with at
minimum `{title, body}`.

## Decision Framework

For each candidate, ask:

1. **Is it a claim?** Does the title state something specific and falsifiable?
   — "Complexity is the enemy of reliability" → yes.
   — "Notes from June 2022" → no, skip.

2. **Is it distinct?** Does lens already have this?
   ```bash
   # First: exact title match
   lens search "exact note title" --resolve --json
   # Then: keyword search for similar territory
   lens search "key terms from the note" --json
   ```
   If exact title match found → skip (duplicate). If 3+ existing notes cover this territory → skip or merge.

3. **Is it worth colliding?** Would this change, challenge, or refine something already in the graph?
   — Contradicts an existing note → import, add `contradicts` link after writing.
   — Adds a new dimension to a topic → import.
   — Confirms what's already there → skip.

**Decision table:**

| Condition | Action |
|---|---|
| Has URL + is about a specific source | `lens fetch <url> --save --json` (don't write manually) |
| Substantive claim, passes 3 questions | Import as Note |
| Short insight, no connections yet | Import as Note (seed — no links yet) |
| Mostly confirms existing notes | Skip |
| Duplicate / very similar to existing | Skip (or merge after import) |

## Writing the Batch

**URL-backed sources go through `lens fetch`, not `lens write`:**

```bash
# Sources with URLs — fetch extracts and normalizes the content
lens fetch "https://example.com/article" --save --json
```

**Everything else goes through batch write:**

```json
[
  {"type": "note", "title": "Complexity is the enemy of reliability", "body": "Evidence from..."},
  {"type": "note", "title": "Another insight", "body": "..."},
  {"type": "source", "title": "Manual source without URL", "source_type": "manual_note"}
]
```

Submit:
```bash
lens write --file /tmp/import-batch.json --json
```

**Check results.** Batch writes are partial-success — some items may fail while others
succeed. Always inspect the `{results:[...]}` output. If any item has `"action":"error"`,
investigate before continuing. Do not re-run the entire batch (this creates duplicates);
retry only failed items.

**Keep batches under 50 items.** For larger imports, pause between batches:
```bash
lens similar --all --threshold 0.8 --json   # check for duplicates introduced so far
```

## Adding Links During Import

Don't try to link everything during the initial write pass. Instead:

1. Write all notes first (get IDs from batch output)
2. For notes that clearly `contradicts` or `refines` something you already know, search and link after writing:
   ```bash
   lens search "related topic" --json
   lens show <existing_note_id> --json
   ```
3. Leave seeds (new territory with no connections) alone — they'll find connections during future compilation.

## Post-Import Audit

After all batches are written, run the Post-Import Audit from [curation.md](curation.md):

1. Check for lateral connections between imported notes (they may reference each other)
2. Dedup scan: `lens similar --all --threshold 0.8 --json`
3. Dense note check: any new note with `forward_links.length > 8`?

## What Not to Import

- Daily journal entries (unless a specific insight is embedded — extract just that)
- Task lists and TODO items
- Highlights without your own commentary
- Anything that's just a summary of a source — import the source instead via `lens fetch`
- Duplicates of what's already in the graph
