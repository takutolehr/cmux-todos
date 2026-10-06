---
name: session-todos
description: Publish this session's TODO list to cmux so the user can follow every session's progress from the cmux sidebar and the all-sessions board. Use when running inside cmux (CMUX_WORKSPACE_ID is set) at the start of any task with 3 or more steps, and again whenever an item starts, finishes, or the plan changes. Also use when the user asks you to add something to their own todo list (for example "put emailing my manager on my todos").
allowed-tools: Bash(cmux-session-todos *)
---

# Session TODOs in cmux

The user runs many Claude sessions side by side in cmux and follows each one's plan from cmux instead of reading every terminal. Your list appears in this workspace's sidebar checklist and on the all-sessions board. Keep it current.

## Publish

Pass the whole list, one argument per item, on one line. It replaces whatever this session published before:

```bash
cmux-session-todos set "[x] Read the existing retry logic" "[>] Add exponential backoff to fetchOrders" "[ ] Cover the timeout path with tests"
```

`[ ]` pending, `[>]` in progress, `[x]` done. The command echoes the list back with a done count.

## When to publish

- Once you understand the task: publish the plan before you start editing.
- When an item starts or finishes: publish again with updated marks. Keep exactly one item `[>]` while you work.
- When the plan changes: add, drop, or reword items and publish.
- When the task is finished: publish with every item `[x]`. Leave it there; the next task's list replaces it.

## Writing items

- 3 to 10 items, each an outcome the user would recognize ("Add backoff to fetchOrders"), not a tool step ("grep for fetchOrders").
- Keep wording stable between updates so cmux keeps the same item instead of creating a new one.
- If you also track tasks with a built-in task tool, keep this list in step with it.

## When the user edits your list

The user can check off, reopen, reword, or delete your items in cmux. `set` keeps their edits and lists them under "The user changed these in cmux". Each one is the user's decision. Adopt it in your next `set`: mark the item the way they did, use their wording, and leave out items they deleted. Don't redo work they checked off. If an item later needs to change again (say the work turned out unfinished), publish the new value as usual.

## The user's personal list

The user keeps their own tasks, such as "Email my manager about the delay", on a personal list that isn't tied to any session. Add to it only when the user asks you to note something for them to do:

```bash
cmux-session-todos personal add "Email my manager about the delay"
```

Never put your own work on it. `cmux-session-todos personal` prints it.

## Other commands

- `cmux-session-todos show`: print the items this session owns, in the same `[ ]` format, including the user's edits. Run it before republishing after a long gap or a context compaction.
- `cmux-session-todos clear`: remove this session's items. Only when the user asks.
- `cmux-session-todos open`: when the user asks to see or bring back the todo board, show it in the cmux Dock. It reuses the existing Dock tab, restarts the board there if it was quit, or adds a new tab.

## Rules

- Change the checklist only through `cmux-session-todos`. Do not call `cmux todo` directly: the checklist also holds the user's own items and other sessions' items, and this script leaves those alone.
- Never edit, check off, or remove the user's own items, on the workspace checklist or the personal list, unless they ask.
- If the command says it is not running inside cmux, skip this skill for the rest of the session.
