# cmux-todos

A Claude Code plugin that puts every Claude session's TODO list, and your own, in [cmux](https://cmux.com). Each session publishes its plan, you can edit any list by hand, and a board shows everything in one place.

```
Session todos                                         11:33:29

Personal                                                   1/2
  ✓ Book the design review
  ○ Email my manager about the release slip

◑ Add retry to the orders client                           2/4
  session 4f546b1a
  ✓ Read the existing retry logic
  ✓ Add exponential backoff to fetchOrders
  ▶ Cover the timeout path with tests
  ○ Update the client README

▾ Website launch                                           1/3
  ✳ Point the domain at the new host                       1/3
    session 9b21c0de
    ✓ Add the DNS records
    ▶ Wait for the certificate
    you
    ○ Tell marketing the site is live

5 workspaces without todos · w to show
space done  a add  e edit  d del  s start  ⏎ go  q quit
```

## Install

Requires [cmux](https://cmux.com) 0.64+, [Claude Code](https://claude.com/claude-code), and `python3` (stdlib only).

```bash
claude plugin marketplace add takutolehr/cmux-todos
claude plugin install cmux-todos@cmux-todos
```

Then, in any Claude session running inside cmux, type:

```
/cmux-todos:open
```

That's the whole setup. It opens the board in the right-sidebar Dock (or in a split if the Dock is turned off). No symlinks, `PATH` changes, or config files are needed: inside Claude sessions the plugin's `bin/` directory is already on `PATH`.

To update to the latest version:

```bash
claude plugin marketplace update cmux-todos
claude plugin update cmux-todos@cmux-todos
```

The skill declares `allowed-tools: Bash(cmux-session-todos *)`, but some modes (headless `claude -p`, at least) still ask before running it. To never be prompted, add the rule to `permissions.allow` in `~/.claude/settings.json`:

```json
{ "permissions": { "allow": ["Bash(cmux-session-todos *)"] } }
```

### Using the board outside Claude sessions

The installed plugin lives under a versioned path that changes with each update, so to run `cmux-session-todos` from your own terminals, the Dock config, or a split pane, clone the repo too:

```bash
git clone https://github.com/takutolehr/cmux-todos ~/cmux-todos
```

The examples below use `~/cmux-todos/bin/cmux-session-todos`. Add `~/cmux-todos/bin` to your `PATH` to shorten that to `cmux-session-todos`.

To try the plugin for one session without installing it: `claude --plugin-dir ~/cmux-todos`.

## Using it

### Claude keeps its own list

There's nothing to do. Every Claude session started inside cmux is told about the plugin, and for any task with 3 or more steps it publishes its plan before it starts, then republishes as items start and finish. Claude runs the update itself, so the list changes at those points, not continuously, and short tasks get no list.

Each session's list appears on its workspace's row in the cmux sidebar and on the board, grouped under `session <id>`.

To put something on your own list, ask any session: "put emailing my manager on my todos".

### Open the board

Type `/cmux-todos:open` in a Claude session, or ask it to "bring back the todo board". It shows the board if it's running, restarts it in its old tab if that tab is back at a shell prompt, and otherwise adds a new tab. It never types into a tab that's running something else. Each window has its own Dock, so do this once per window.

### Edit any list yourself

Everything is editable outside the session, and your edits win.

**On the board**, move with `↑`/`↓` (or `j`/`k`), jump to the next or previous list heading with `tab`/`shift-tab`, and press:

| Key | Action |
| --- | --- |
| `space` | check off / reopen |
| `s` | mark in progress |
| `a` | add an item to the selected list (Personal or a workspace) |
| `e` | edit the text |
| `d` | delete the item, or on a list heading, every item in that list (asks first) |
| `enter` | jump to that workspace |
| `c` / `w` | hide completed items / show workspaces without todos |
| `q` | quit |

In the text prompt, `enter` accepts, `esc` cancels, and `ctrl-u` / `ctrl-w` clear the line or the last word.

**From any terminal:**

```bash
cmux-session-todos personal add "Email my manager about the release slip"
cmux-session-todos personal               # numbered list
cmux-session-todos personal check 1       # also: uncheck, start, edit N "text", rm N, clear

cmux todo add "Tell QA the fix is in" --workspace workspace:3   # a workspace's checklist
```

**In cmux itself:** turn on `sidebar.beta.workspaceTodos.controls.enabled` in `~/.config/cmux/cmux.json` to get add and check controls on each workspace's checklist in the sidebar.

**What a session does with your edits:** when you check off, reopen, reword, or delete one of a session's items, its next `set` keeps your edit and reports it back to the agent, which then adopts it. This is a field-by-field merge against what the session last published. If the session also changed that field, your edit still wins, and the agent sees why in the output. Your own items (added on the board, with `cmux todo add`, or in the cmux sidebar) are never touched by a session.

## Where to show it

The board always covers every window and workspace group, wherever it runs. What changes is where you can see it from:

| Place | Visible from | What it shows |
| --- | --- | --- |
| **Right-sidebar Dock** (recommended) | every workspace in that window | the whole board: personal list, all windows and groups, editable |
| A split pane | only the workspace it's in | the whole board |
| cmux's own left sidebar | every workspace | each workspace's checklist on that workspace's row; no personal list, no combined view |
| A custom cmux sidebar | | not possible: custom sidebars receive no checklist data |

### Dock (recommended)

1. On cmux builds where the Dock is still a beta, turn on **Settings → Beta Features → Dock** (`rightSidebar.beta.dock.enabled`).
2. Run `/cmux-todos:open` in a Claude session in each window.

Optional: to have every new window's Dock start with the board, create `~/.config/cmux/dock.json`. The command needs the script's full path, so point it at your clone:

```json
{
  "controls": [
    {
      "id": "session-todos",
      "title": "Todos",
      "command": "~/cmux-todos/bin/cmux-session-todos board --watch 2"
    }
  ]
}
```

Outside Claude, `~/cmux-todos/bin/cmux-session-todos open` does what `/cmux-todos:open` does. Run it in a terminal inside the Dock and that terminal becomes the board.

`cmux todo open` is a different command: it opens cmux's own checklist pane for a single workspace. It isn't this board, and it fails in Dock terminals, which belong to the window rather than to a workspace.

### Split pane

```bash
cmux new-pane --direction right --command "~/cmux-todos/bin/cmux-session-todos board --watch 2"
```

Without `--watch`, `board` prints the board once.

### cmux's own sidebar

Each workspace row in cmux's left sidebar already shows that workspace's checklist, session items included, as a summary you can open. Turn on `sidebar.beta.workspaceTodos.controls.enabled` in `~/.config/cmux/cmux.json` to add and check items there.

## How it works

- **Skill `session-todos`** tells the agent to publish its plan with `cmux-session-todos set "[x] ..." "[>] ..." "[ ] ..."` when a task starts, and to republish whenever an item starts or finishes.
- **Slash command `/cmux-todos:open`** opens the board in the cmux Dock, or brings it back after you've closed it.
- **SessionStart hook** reminds every session started inside cmux that the skill exists. Outside cmux it prints nothing.
- **`bin/cmux-session-todos`** writes a session's items into cmux's own per-workspace checklist (`cmux todo`), tagged `origin=agent`. They show up wherever cmux shows that checklist: the sidebar row and the workspace todo pane (`cmux todo open`).
- **Personal list** holds your own tasks that don't belong to any session. It's stored in `~/.local/share/cmux-session-todos/personal.json` and shown at the top of the board.
- **`cmux-session-todos board`** shows the personal list and every workspace's checklist from every window, laid out like the cmux sidebar: window headings (when you have more than one window), workspace groups with their members indented, then each workspace's items grouped by session.

Each session only replaces its own items. Items you add yourself, and items from other live sessions in the same workspace, are left in place. Which item IDs belong to which session is recorded in `~/.local/state/cmux-session-todos/sessions/<session-id>.json`. When Claude is restarted in the same terminal, the new session takes over the previous session's items so they don't pile up as duplicates.

## Commands

| Command | What it does |
| --- | --- |
| `set ITEM...` | Replace this session's items. Each item is `"[ ] text"`, `"[>] text"` or `"[x] text"`. With no arguments it reads one item per line from stdin. Prints the result and any edits you made in cmux. |
| `show` | Print this session's items, including your edits. |
| `clear` | Remove this session's items. |
| `personal [list\|add\|check\|uncheck\|start\|edit\|rm\|clear]` | Your own list. |
| `board [--watch SECS] [--hide-done]` | The personal list and every workspace's items. Editable with `--watch`. |
| `open` | Show the board in this window's Dock, restarting it or adding a tab if needed. This is what `/cmux-todos:open` runs. |

`set`, `show` and `clear` need `CMUX_WORKSPACE_ID` and `CLAUDE_CODE_SESSION_ID` in the environment, which cmux and Claude Code set for the agent's shell. `personal` and `board` work from any terminal; `board` needs cmux running.

## Limits

- cmux caps each workspace checklist at 50 items; `set` refuses a list that would exceed it.
- cmux moves an item to the top when it's reopened and to the bottom when it's checked off, so items reorder as you work.
- A session's list stays after the session ends, so you can still see where it stopped. The next session in that terminal replaces it. To delete a leftover list, press `d` on its heading on the board.
- The personal list lives only on the board; cmux's sidebar shows only workspace checklists.
- A custom cmux sidebar (`~/.config/cmux/sidebars/`) can't show these lists. On cmux 0.64.25 its workspace data has no checklist fields, and it can't read files, so the board runs as a terminal in the Dock or a pane.
