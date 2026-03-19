# doit Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a Claude Code plugin that provides `/doit` — a slash command that turns `.claude/todos.md` into an executable prompt queue.

**Architecture:** Single-skill Claude Code plugin. One SKILL.md file contains all instructions for Claude. No runtime code — the skill is pure markdown that tells Claude how to parse, display, and execute TODOs.

**Tech Stack:** Claude Code plugin system (YAML frontmatter + Markdown)

**Spec:** `docs/superpowers/specs/2026-03-19-doit-design.md`

---

## File Structure

```
doit/
├── .claude-plugin/
│   └── plugin.json          # Plugin metadata (name, version, description)
├── skills/
│   └── doit/
│       └── SKILL.md         # The /doit slash command (core artifact)
├── .gitignore
├── CLAUDE.md                # (exists) Project instructions for contributors
├── README.md                # (exists) User-facing docs
├── LICENSE                  # MIT license
└── docs/                    # (exists) Specs and plans
```

---

### Task 1: Initialize git repo and project scaffolding

**Files:**
- Create: `.gitignore`
- Create: `LICENSE`
- Create: `.claude-plugin/plugin.json`
- Pre-existing: `CLAUDE.md`, `README.md`, `docs/` (created during brainstorming)

- [ ] **Step 1: Initialize git repo**

```bash
cd /Users/olistic/Projects/doit
git init
```

- [ ] **Step 2: Create .gitignore**

```
.DS_Store
```

- [ ] **Step 3: Create LICENSE**

MIT license with copyright `2026 Matias Olivera`.

- [ ] **Step 4: Create plugin.json**

```json
{
  "name": "doit",
  "version": "0.1.0",
  "description": "Turn a markdown checklist into an executable prompt queue"
}
```

- [ ] **Step 5: Commit scaffolding**

```bash
git add .gitignore LICENSE .claude-plugin/plugin.json CLAUDE.md README.md docs/
git commit -m "init: project scaffolding"
```

---

### Task 2: Write the SKILL.md

This is the core artifact. Follow the writing-skills skill for authoring guidance.

**Files:**
- Create: `skills/doit/SKILL.md`

**Reference:** Spec sections "File format", "Command: /doit", and "SKILL.md" define the exact behavior.

- [ ] **Step 1: Write SKILL.md with frontmatter and body**

Frontmatter:

```yaml
---
name: doit
description: Use when the user invokes /doit to list and execute pending TODOs from .claude/todos.md
user-invocable: true
allowed-tools: Read, Edit, Write
---
```

Body must instruct Claude to:

1. Read `.claude/todos.md` from the current project directory.
2. If the file doesn't exist, offer to create it with a starter example:
   ```md
   - [ ] your first TODO here
   ```
   If the user accepts, create the file using the Write tool, then tell them to add their TODOs and run `/doit` again. Stop here.
3. Parse the file according to these rules:
   - `- [ ] ` starts a pending item
   - `- [x] ` or `- [X] ` starts a completed item
   - Lines indented by 2+ spaces (or tabs) that don't start a new item are continuations
   - Blank lines between items do not break continuation
   - Near-miss syntax (e.g., `- [] no space`, `- ( ) parens`) is ignored — not items, no warning
   - All other lines (headings, plain text, plain list items without checkboxes) are ignored
4. If no pending items, display "No pending TODOs." with the completed count. Stop here.
5. Display pending items numbered sequentially with a summary:
   ```
   Pending TODOs:
     1. first item text
     2. second item text
        with continuation lines

   N pending, M completed
   ```
6. Ask the user to pick a number.
7. After the user replies with a number, treat the full text of the selected item (including continuation lines) as an instruction and act on it immediately.
8. After completing the work, use the Edit tool to replace `- [ ]` with `- [x]` on the item's first line in `.claude/todos.md`.

Keep the SKILL.md concise — under 200 words if possible. Claude doesn't need verbose instructions for a straightforward flow.

- [ ] **Step 2: Commit SKILL.md**

```bash
git add skills/doit/SKILL.md
git commit -m "feat: add /doit skill"
```

---

### Task 3: Test locally

- [ ] **Step 1: Create a temporary test project**

```bash
mkdir -p /tmp/doit-test
cd /tmp/doit-test
```

All testing happens in this directory, not in the doit repo.

- [ ] **Step 2: Test with no todos.md file**

```bash
cd /tmp/doit-test
claude --plugin-dir /Users/olistic/Projects/doit
```

Inside Claude Code, run `/doit`. Expected: Claude offers to create `.claude/todos.md`.

Decline the offer. Verify Claude stops gracefully.

- [ ] **Step 3: Create a test todos.md and test listing**

Create `/tmp/doit-test/.claude/todos.md` with:

```md
- [ ] say hello
- [x] this was already done
- [ ] say goodbye
      and wish them well
```

Run `/doit` in the same Claude Code session. Expected:

```
Pending TODOs:
  1. say hello
  2. say goodbye
     and wish them well

2 pending, 1 completed
```

- [ ] **Step 4: Test execution and mark-complete**

Reply with "1". Expected: Claude says hello and marks the first item `[x]` in the file.

Verify the file now shows `- [x] say hello`.

- [ ] **Step 5: Test edge cases**

- Reply with an invalid number (e.g., "5") — should ask for valid number
- Test with all items completed — should show "No pending TODOs."
- Test multi-line item execution — reply with "2" and verify continuation text is included

- [ ] **Step 6: Fix any issues found during testing**

If the SKILL.md needs adjustments based on test results, edit and re-test.

- [ ] **Step 7: Commit any fixes**

```bash
git add skills/doit/SKILL.md
git commit -m "fix: adjust skill based on testing"
```

(Skip if no fixes needed.)

---

### Task 4: Polish and publish

- [ ] **Step 1: Review README.md for accuracy**

Verify the README matches the final implementation. Update install command, file format examples, and behavior description if anything changed during testing.

- [ ] **Step 2: Create GitHub repo and push**

```bash
gh repo create olistic/doit --public --source=. --push
```

- [ ] **Step 3: Verify plugin install works**

From a different directory:

```bash
claude plugin install olistic/doit
```

Run `/doit` to verify it works when installed as a plugin (not just with `--plugin-dir`).
