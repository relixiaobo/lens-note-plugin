---
description: Save a URL and process it into your knowledge graph in the background
argument-hint: '<url>'
allowed-tools: Bash(lens:*), Agent
---

Save and process: $ARGUMENTS

**Step 1 — Save** (foreground):

Run: `lens fetch "$ARGUMENTS" --save --json`

If fetch fails: report the error clearly and stop.

If successful: tell the user the article **title** and **word count**. Say: "Saved. Processing in the background."

**Step 2 — Process** (background):

Spawn a background Agent with the following instruction (substitute actual `<source_id>` and `<title>` from Step 1):

> Process the source `<source_id>` titled "<title>" into the user's knowledge graph.
>
> **Your job**: read the article, find where it meets the user's existing thinking, and write what actually emerged — not what the article said, but what's new or different when it meets what's already there.
>
> Steps:
> 1. Read the full source: `lens show <source_id> --json`
> 2. Pick out 2–4 concrete claims from the article (not themes — specific things it actually says)
> 3. For each claim, search the user's notes: `lens search "<keywords>" --json`
> 4. Read the top results: `lens show <id> --json`
> 5. See what happens when the article meets the existing notes:
>    - Pulls in a different direction from an existing note → `contradicts` link
>    - Backs up something the user already thinks → `supports` link
>    - Adds precision to an existing note → `refines` link
>    - Genuinely new ground with no connections yet → create as a seed note, no links for now
>    - Basically confirms what's already there → update the existing note, don't create a new one
>
> Write the notes that actually emerged from this. Use `lens write --file <tmp> --json` for batch writes. Link each note back to the source.
>
> **Plain language only.** Each note title is one clear claim. The body is evidence, context, or the reason the tension exists — written like you're explaining it to a friend, not summarizing an article.
>
> Do not extract notes mechanically — one per paragraph, one per idea from the article. Only write what's genuinely new or interesting when it meets the existing graph. Zero notes is a valid outcome if the article mostly confirms what's already there.

Do not wait for the background agent to complete.
