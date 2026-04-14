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
   - `.md` files (folder) → **Obsidian vault**
   - `.jsonl` → **Pre-cleaned JSONL** (skip to Step 3)
   - `.csv` / `.txt` / other → **Generic** (write a small cleaning script)
3. Report to the user: format detected, file size, rough item count

## Step 2 — Clean

Parse the raw format into `{title, body}` pairs. Use pre-built parsers when available:

- **Tana**: `npx tsx spike/tana-clean.ts "$ARGUMENTS" /tmp/tana-clean.jsonl`
- **Roam**: `npx tsx spike/roam-clean.ts "$ARGUMENTS" /tmp/roam-clean.jsonl`
- **Obsidian**: Read `.md` files directly — no parser needed. Apply the filter heuristics from import.md.
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
2. **Is it distinct?** `lens search "key terms" --json` — skip if 3+ existing notes cover this.
3. **Is it worth colliding?** Would this change or refine something already in the graph?

Work in chunks of ~20 candidates. For each chunk, present a summary:
- How many to import, how many to skip, how many need user decision
- List the "import" candidates with one-line titles
- List any borderline cases for user input

## Step 5 — Write

Collect approved items into a JSON array file. Keep batches under 50 items.

```bash
lens write --file /tmp/import-batch-N.json --json
```

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
