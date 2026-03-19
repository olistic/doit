---
name: doit
description: Use when the user invokes /doit to list and execute pending TODOs from .claude/todos.md
user-invocable: true
allowed-tools: Read, Edit, Write
---

Read `.claude/todos.md` from the project root.

If the file doesn't exist, offer to create it with this starter content:

```md
- [ ] your first TODO here
```

If the user accepts, create it with the Write tool, then tell them to add their TODOs and run `/doit` again. Stop.

Parse the file:
- `- [ ] ` — pending item
- `- [x] ` or `- [X] ` — completed item
- Lines indented 2+ spaces or a tab (that don't start a new item) continue the previous item
- Blank lines between items do not break continuation
- Ignore all other lines (headings, plain text, plain list items, near-miss syntax like `- []` or `- ( )`)

If no pending items, display "No pending TODOs." with the completed count. Stop.

Display pending items:

```
Pending TODOs:
  1. first item text
  2. second item text
     with continuation

N pending, M completed
```

Ask the user to pick a number. If they reply with an invalid number, re-display the list and ask again.

When they reply with a valid number, treat the full text of that item (including continuations) as an instruction and execute it immediately.

After the work is done, use Edit to replace `- [ ]` with `- [x]` on the item's first line in `.claude/todos.md`.
