---
description: Save a URL and prepare a collision for your review
argument-hint: '<url>'
allowed-tools: Bash(lens:*), Agent
---

Save and prepare: $ARGUMENTS

**Step 1 — Save** (foreground):

Run: `lens fetch "$ARGUMENTS" --save --json`

If fetch fails: report the error clearly and stop.

If successful: tell the user the article **title** and **word count**. Say: "Saved. Preparing your collision in the background."

**Step 2 — Prepare collision** (background):

Spawn a background Agent with the following instruction (substitute actual `<source_id>` and `<title>` from Step 1):

> A new source `<source_id>` titled "<title>" was saved to lens. Your job is to prepare a collision — not to extract notes mechanically, but to find where this article meets the user's existing thinking.
>
> Steps:
> 1. Read the full source: `lens show <source_id> --json`
> 2. Identify 2–4 key claims or arguments from the article
> 3. For each claim, search the user's knowledge graph: `lens search "<claim keywords>" --json`
> 4. Read top results: `lens show <id> --json`
> 5. Find the most interesting collision points — where the article supports, contradicts, or extends what the user already thinks
>
> Then create a single task in lens:
>
> Title: "Collide: <title>"
> Status: open
> Body (write in markdown):
>
> ## Core argument
> 2–3 sentences: what does this article actually claim?
>
> ## Where it meets your thinking
> - `note_ID` "Note title" — how the article relates: supports / contradicts / extends, and why it matters
> - `note_ID` "Note title" — (add 2–4 collision points total)
>
> ## Your move
> One question only the user can answer — something that requires their perspective and experience, not just the article's content. Make it specific and provocative.
>
> Use `lens write --file <tmp> --json` with `{"type":"task","title":"Collide: <title>","status":"open","body":"..."}` to create the task.
>
> Do not write any notes. The user will guide what gets crystallized after they engage with this task.

Do not wait for the background agent to complete.
