# lens plugin

Knowledge graph plugin for AI agents. Gives your agent persistent memory across sessions.

## Install

### Claude Code

```
/plugin marketplace add relixiaobo/lens-note-plugin
/plugin install lens
```

### Other agents (Codex CLI, Gemini CLI, Cursor)

Copy the skill file to your agent's skills directory:

```bash
# Universal (works for Claude Code, Codex CLI, Gemini CLI)
mkdir -p ~/.agents/skills/lens
cp skills/lens/SKILL.md ~/.agents/skills/lens/

# Or agent-specific:
# Claude Code:  ~/.claude/skills/lens/SKILL.md
# Codex CLI:    ~/.codex/skills/lens/SKILL.md
# Gemini CLI:   ~/.gemini/skills/lens/SKILL.md
# Cursor:       .cursor/skills/lens/SKILL.md
```

### Prerequisites

lens CLI must be installed:

```bash
npm install -g lens-note
lens init
```

## What it does

lens stores, queries, and links structured knowledge. Your agent uses it to:

- **Compile articles** — fetch content, think about connections, write linked notes
- **Answer from knowledge** — search notes, synthesize answers with evidence
- **Maintain the graph** — find orphan notes, add missing links

5 core commands: `search`, `show`, `write`, `fetch`, `health`. All support `--json`.

Zero LLM dependency. Zero API keys. The agent does the thinking; lens does the storage.

## Slash commands (Claude Code)

Three user-invocable commands for direct interaction:

| Command | What it does |
|---|---|
| `/lens:save <url>` | Save a URL and process it into your knowledge graph in the background |
| `/lens:recall <topic>` | Recall what you think about a topic — synthesized from your notes |
| `/lens:review [week\|month\|year]` | Reflect on recent learning — where you've been and where to go next |

## Links

- [lens-note on npm](https://www.npmjs.com/package/lens-note)
- [lens-note on GitHub](https://github.com/relixiaobo/lens-note)

## License

MIT
