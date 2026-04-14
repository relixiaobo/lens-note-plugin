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

> A new source `<source_id>` titled "<title>" was saved to lens. Find where this article meets the user's existing thinking — don't extract notes, just map the collision and leave one task.
>
> Steps:
> 1. Read the full source: `lens show <source_id> --json`
> 2. Pick out 2–4 things the article actually says (not themes — concrete claims)
> 3. Search for each in the user's notes: `lens search "<keywords>" --json`
> 4. Read the top results: `lens show <id> --json`
> 5. Find where they connect, clash, or push past what the user already thinks
>
> Create one task in lens. Write the body like you're leaving a note for a friend — plain language, short sentences, no academic framing. Say what you actually mean.
>
> Title: "Collide: <title>"
> Status: open
> Body:
>
> ## What this is about
> 2–3 plain sentences. What does the article say? Not what it "explores" or "examines" — what does it actually claim?
>
> ## Where it hits your notes
> - "[Note title]" (`note_ID`) — [one plain sentence: does this back it up, clash with it, or take it further? say which and say why]
> - "[Note title]" (`note_ID`) — [same]
> (2–4 total)
>
> ## Worth thinking about
> One question in plain language — something only you can answer from your own experience. Not "how does this relate to your epistemological framework" — something real, like "does this change how you think about X?"
>
> Use `lens write --file <tmp> --json` with `{"type":"task","title":"Collide: <title>","status":"open","body":"..."}` to create the task.
>
> Do not write any notes. Only this one task. The user decides what gets written after they engage with it.

Do not wait for the background agent to complete.
