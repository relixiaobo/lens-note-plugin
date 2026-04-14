---
description: Reflect on recent learning — where you've been and where to go next
argument-hint: '[week|month|year]'
allowed-tools: Bash(lens:*)
---

Reflect on recent knowledge activity. Period: $ARGUMENTS (default: week if no argument provided)

Run in order:
1. `lens digest $ARGUMENTS --json` (substitute `week` if $ARGUMENTS is empty)
2. `lens tasks --json`
3. `lens list notes --orphans --limit 5 --json`

Write like you're catching up with a friend after a week away. Plain language, no academic framing. Say what you actually see.

**Lead with one observation.** Before any lists: what's the most interesting thing that happened in the user's thinking this period? One sentence, direct. Not "the knowledge graph shows increased connectivity" — something real, like "you've been circling around X and haven't landed yet."

Then present:

**Unresolved tensions** — for each note with a `contradicts` link, state the conflict in plain language: "You believe [A], but you also noted [B]. These pull in opposite directions." Do not just list titles. If no tensions, skip this section.

**Pending collides** — tasks titled "Collide: ..." created by `/lens:save`. These are articles waiting for your perspective. For each: show the article title and the "Your move" question from the task body. This is the most actionable section.

**Seeds wanting connection** — for each orphan note, don't just list it. Look at its content (use `lens show <id> --json`) and suggest one possible connection: "This might link to [existing note or theme] because..." Limit to 3.

**Open tasks** — list titles only, excluding Collide tasks already shown above.

Skip connected notes entirely — they are done, no action needed.

Close with one direction: what single thread is most worth following this week, and why?
