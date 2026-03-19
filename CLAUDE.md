# doit

A Claude Code plugin that turns a markdown checklist into an executable prompt queue.

## Project structure

This is a Claude Code plugin. The main skill is defined in `skills/doit/SKILL.md`.

```
doit/
├── .claude-plugin/
│   └── plugin.json          # Plugin metadata
├── skills/
│   └── doit/
│       └── SKILL.md         # The /doit slash command
├── CLAUDE.md
├── README.md
└── LICENSE
```

## Key concepts

- **File:** `.claude/todos.md` in the user's project — a markdown checklist of prompts
- **Command:** `/doit` — lists pending TODOs, user picks one, Claude executes it
- **Completion:** After executing a TODO, the item is marked `[x]` in the file

## Development

Test locally with:

```sh
claude --plugin-dir .
```

Reload after changes with `/reload-plugins` inside Claude Code.

## Conventions

- Use [Conventional Commits](https://www.conventionalcommits.org) with scope when applicable
- Keep descriptions under 72 characters, imperative mood
