# Task Management

Tasks are the collaboration protocol between you and the user. Knowledge inspires action, action produces knowledge.

```
Knowledge (note) → inspires → Task → doing → produces → Knowledge (note)
```

## When to Create Tasks

- User explicitly asks you to do something: "帮我调研一下 X"
- You discover something to investigate during compilation: a contradiction to verify, a gap to fill
- User says "记一下要做的事" / "TODO" / "add a task"

## Creating a Task

```bash
printf '%s' '{"command":"write","input":{"type":"task","title":"调研 MCP 协议的设计理念","status":"open","body":"重点关注与 CLI 的关系","links":[{"to":"note_01ABC","rel":"related","reason":"来自这条笔记的启发"}]}}' | lens --stdin
```

- `title`: what needs doing (imperative, one sentence)
- `status`: always `open` when creating
- `body`: context, breakdown, notes on approach
- `links`: connect to the note/source that prompted this task

## Checking Tasks

```bash
lens tasks --json              # open tasks
lens tasks --done --json       # completed tasks
lens tasks --all --json        # everything
```

When the conversation topic relates to an open task, mention it naturally: "This relates to your open task about X (task_01ABC)."

## Completing a Task

When a task is done, mark it and capture what you learned:

```bash
# Complete + reflect (batch write)
printf '%s' '{"command":"write","input":[
  {"type":"update","id":"task_01ABC","set":{"status":"done"}},
  {"type":"note","title":"Lesson from completing task","body":"What I learned...","links":[{"to":"$0","rel":"related","reason":"reflection after completing this task"}]}
]}' | lens --stdin
```

The reflection note is optional but valuable — it closes the knowledge loop. If the task didn't produce new insight, just mark it done without a note.

## Updating Progress

For long-running tasks, update the body with progress:

```bash
printf '%s' '{"command":"write","input":{"type":"update","id":"task_01ABC","body":"Updated progress:\n- Step 1 done\n- Step 2 in progress"}}' | lens --stdin
```

## Rules

1. **One task, one action.** If it has multiple steps, track them in the body, not as separate tasks.
2. **Link to knowledge.** A task without links is an orphan — connect it to the note or source that prompted it.
3. **Reflect on completion.** The lesson from doing something is often more valuable than the doing itself.
4. **Don't over-create.** Not every user request is a task. Quick questions, simple lookups — just do them. Tasks are for work that spans time or produces knowledge.
