---
name: open
description: Show the todo board (every session's TODO list plus the user's personal list) in the cmux Dock, reopening or restarting it if it was closed.
disable-model-invocation: true
allowed-tools: Bash(cmux-session-todos open)
---

!`cmux-session-todos open 2>&1`

Relay the line above to the user, nothing more. If it failed because the session is not running inside cmux, say the board only works in cmux.
