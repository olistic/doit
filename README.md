# doit

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) plugin that turns a markdown checklist into an executable prompt queue. Define your TODOs, pick one, and let Claude do it.

## How it works

1. Create a `.claude/todos.md` file in your project with a markdown checklist:

   ```md
   - [ ] refactor the auth middleware to use the new token format
   - [x] add error handling to the payment webhook
   - [ ] update the user schema to include `preferences`
         make sure to add a migration and update the seed script
   ```

2. Run `/doit` in Claude Code. It lists your pending items:

   ```
   Pending TODOs:
     1. refactor the auth middleware to use the new token format
     2. update the user schema to include `preferences`
        make sure to add a migration and update the seed script

   2 pending, 1 completed
   ```

3. Pick a number. Claude executes the prompt and marks it `[x]` when done.

## Install

In Claude Code, run:

```
/plugin marketplace add https://github.com/olistic/doit.git
/plugin install doit
```

## File format

The file is a standard markdown checklist (`.claude/todos.md`):

- `- [ ]` marks a pending item
- `- [x]` marks a completed item
- Indented lines after a checklist item are part of that prompt
- Add it to `.gitignore` if your TODOs are personal

## License

MIT
