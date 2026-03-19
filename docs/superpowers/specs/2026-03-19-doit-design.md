# doit — Design Spec

**Date:** 2026-03-19
**Repo:** olistic/doit
**Distribution:** Claude Code plugin (Git-based marketplace)

## Problem

When working with Claude Code on a project, developers often maintain a TODO list of prompts in an unsaved editor tab. This creates friction:

- An unsaved "ghost" tab that clutters the IDE and can't safely be closed
- Context-switching between the editor and terminal
- Copy-paste tedium to send each prompt to Claude Code

## Solution

A Claude Code plugin that provides a single slash command (`/doit`) backed by a markdown checklist file (`.claude/todos.md`). The developer manages the file in their editor; Claude Code reads it, presents pending items, and executes the selected one.

## File format

Path: `.claude/todos.md` (inside the project's `.claude/` directory).

Standard markdown checklist with multi-line support:

```md
- [ ] refactor the auth middleware to use the new token format
- [x] add error handling to the payment webhook
- [ ] update the user schema to include `preferences`
      make sure to add a migration and update the seed script
```

### Parsing rules

- A line matching `- [ ] ` (with a space inside brackets) starts a new pending item.
- A line matching `- [x] ` or `- [X] ` starts a new completed item.
- Any line indented by 2 or more spaces that does not start a new checklist item is a continuation of the preceding item. Tabs count as indentation.
- Blank lines between items are allowed and do not break continuation.
- All other lines (headings, plain text, plain list items without checkboxes, HTML comments) are ignored.
- Near-miss syntax (e.g., `- [] no space`, `- ( ) parens`) is ignored — not treated as items, no warning.

## Command: `/doit`

### Interaction model

The command is a single-turn skill. When invoked, it outputs the numbered list and asks the user to reply with a number. The user's reply is a normal conversation turn — Claude reads the number, executes the corresponding prompt, and marks it complete. No special multi-turn mechanism is needed; it's just a natural two-message conversation flow.

### Execution semantics

"Execute the selected item" means Claude treats the item's full text (including continuation lines) as its next instruction and acts on it immediately within the same conversation. It is equivalent to the user having typed that text as a message. Claude uses all tools available in the session (not limited to the skill's `allowed-tools`) when executing the prompt.

### Behavior

1. Read `.claude/todos.md` from the current project.
2. Parse all checklist items, separating pending from completed.
3. Display pending items with sequential numbers:
   ```
   Pending TODOs:
     1. refactor the auth middleware to use the new token format
     2. update the user schema to include `preferences`
        make sure to add a migration and update the seed script

   2 pending, 1 completed
   ```
4. Ask the user to reply with a number.
5. When the user replies, treat the selected item's full text as an instruction and act on it.
6. After completing the work, mark the item done by replacing `- [ ]` with `- [x]` on the corresponding line using the Edit tool.

### Edge cases

- **File doesn't exist:** Offer to create `.claude/todos.md` with a starter example. If the user accepts, create the file and tell them to add their TODOs.
- **No pending items:** Display "No pending TODOs." with the completed count.
- **Invalid selection:** Ask the user to pick a valid number from the list.

## Plugin structure

```
doit/
├── .claude-plugin/
│   └── plugin.json
├── skills/
│   └── doit/
│       └── SKILL.md
├── CLAUDE.md
├── README.md
└── LICENSE
```

### plugin.json

```json
{
  "name": "doit",
  "version": "0.1.0",
  "description": "Turn a markdown checklist into an executable prompt queue"
}
```

### SKILL.md

The skill file contains YAML frontmatter and a markdown body with the full instructions for Claude.

Frontmatter:

```yaml
---
name: doit
description: Use when the user invokes /doit to list and execute pending TODOs from .claude/todos.md
user-invocable: true
allowed-tools: Read, Edit, Write
---
```

Body: The markdown body instructs Claude to:

1. Use the Read tool to read `.claude/todos.md`.
2. Parse the file according to the parsing rules defined above.
3. Output the numbered list of pending items with the summary line.
4. Ask the user to pick a number.
5. After the user replies, execute the selected item's text as if the user had typed it.
6. After completing the work, use the Edit tool to replace `- [ ]` with `- [x]` on the item's first line.
7. Handle edge cases (missing file, no pending items, invalid selection) as specified above.

## Non-goals

- No priority system or ordering beyond file order
- No due dates or metadata
- No integration with external task trackers
- No automatic prompt generation
- No multi-file support (single `todos.md` per project)
- No npm package (plugin is Git-based; npm adds maintenance overhead with no clear benefit)
