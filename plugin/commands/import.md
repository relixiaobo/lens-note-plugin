---
description: Import notes from another knowledge base (Tana, Roam, Obsidian, or any export file)
argument-hint: '<file-path>'
allowed-tools: Bash(lens:*), Bash(npx:*), Bash(ls:*), Bash(wc:*), Bash(cat:*), Read, Write, Glob, Grep, Agent
---

Import knowledge from an external file: $ARGUMENTS

Read [references/import.md](../skills/lens/references/import.md) first — it contains the full methodology, format-specific guidance, and decision framework. Follow it precisely.

## Step 1 — Assess

Determine what you're working with:

1. Read the file at `$ARGUMENTS` (or list the directory if it's a folder)
2. Detect the format:
   - `.json` with `docs` array → **Tana export**
   - `.edn` → **Roam Research backup**
   - `.md` (single file) → **Plain markdown** (read directly)
   - Directory with `.md` files → **Obsidian vault** (use recursive glob `**/*.md`)
   - `.jsonl` → **Pre-cleaned JSONL** (skip to Step 3)
   - `.csv` / `.txt` / other → **Generic** (write a JS/TS cleaning script, run via `npx tsx`)
3. Report to the user: format detected, file size, rough item count

## Step 2 — Clean

Parse the raw format into `{title, body}` pairs. Use pre-built parsers when available:

- **Tana**: `npx tsx spike/tana-clean.ts "$ARGUMENTS" /tmp/tana-clean.jsonl` (spike scripts are in the lens project root — run from the lens repo directory, or use the absolute path)
- **Roam**: `npx tsx spike/roam-clean.ts "$ARGUMENTS" /tmp/roam-clean.jsonl` (same — in lens project root)
- **Obsidian**: Read `.md` files directly — no parser needed. Use recursive glob for vault directories. Apply the filter heuristics from import.md.
- **Generic**: Write a small cleaning script to produce JSONL with `{title, body}` per line.

## Step 3 — Preview

Show the user what you found. Present:

- Total candidate count
- Breakdown by category (if applicable)
- 5-10 sample titles (pick a representative mix, not just the first few)
- Word count distribution (min / median / max)

Ask the user: "Ready to proceed with filtering, or do you want to adjust?"

Wait for user confirmation before continuing.

## Step 4 — Decide

For each candidate, apply the decision framework from import.md:

1. **Is it a claim?** Does the title state something specific?
2. **Is it distinct?** First check exact title: `lens search "title" --resolve --json`. If match found → skip. Then keyword search: `lens search "key terms" --json` — skip if 3+ existing notes cover this.
3. **Is it worth colliding?** Would this change or refine something already in the graph?

Work in chunks of ~20 candidates. For each chunk, present a summary:
- How many to import, how many to skip, how many need user decision
- List the "import" candidates with one-line titles
- List any borderline cases for user input

## Step 5 — Write

Present the final list to the user and ask for confirmation before writing anything.

**URL-backed sources go through `lens fetch`** (not batch write):
```bash
lens fetch "<url>" --save --json
```

**Everything else** — collect into a JSON array file. Keep batches under 50 items:
```bash
lens write --file /tmp/import-batch-N.json --json
```

**Check results after each batch.** Batch writes are partial-success. Inspect `{results:[...]}` — if any item has `"action":"error"`, investigate and retry only failed items. Do not re-run the entire batch.

Between batches, check for duplicates introduced so far:
```bash
lens similar --all --threshold 0.8 --json
```

If duplicates found, pause and resolve before continuing.

## Step 6 — Post-Import Audit

After all batches are written:

1. Check for lateral connections between imported notes
2. Run dedup scan: `lens similar --all --threshold 0.8 --json`
3. Report final stats: total imported, total skipped, any duplicates found

Do NOT try to link everything during import. Seeds (new territory with no connections) are fine — they'll find connections during future compilation.
