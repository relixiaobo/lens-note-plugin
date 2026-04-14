---
description: Save a URL to your knowledge graph and process it in the background
argument-hint: '<url>'
allowed-tools: Bash(lens:*), Agent
---

Save and process: $ARGUMENTS

**Step 1 — Save** (foreground):

Run: `lens fetch "$ARGUMENTS" --save --json`

If fetch fails (network error, invalid URL, unsupported site): report the error clearly and stop.

If successful: tell the user the article **title**, **word count**, and **source ID**. Then say: "Saved. Processing in the background — it will be ready in your knowledge graph shortly."

**Step 2 — Compile** (background):

Spawn a background Agent with the following instruction (substitute actual `<source_id>` and `<title>` from Step 1 output):

> Process the lens source `<source_id>` titled "<title>" using the Collision Method.
>
> Steps:
> 1. Read the full source body: `lens show <source_id> --json`
> 2. Search for related notes using 2–3 key concepts extracted from the title and content
> 3. Follow links from the top results — look for connections, contradictions, refinements
> 4. Extract 3–7 key insights from the article as distinct notes
> 5. Write notes with links to each other and back to the source, using `lens write --file <tmp> --json`
> 6. After writing, check for near-duplicates: `lens similar --all --json` — merge if similarity > 0.5
>
> Prefer updating existing notes over creating new ones. Zero new notes is acceptable if the content largely confirms what is already in the graph. Do not summarize verbosely — one crisp claim per note.

Do not wait for the background agent to complete.
