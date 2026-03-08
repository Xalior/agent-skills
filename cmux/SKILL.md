---
name: cmux
description: "Control the cmux terminal multiplexer. Use this skill when the user asks to manage terminal panes, workspaces, windows, browser splits, or interact with cmux in any way. Also use when you need to read other terminal screens, send commands to other panes, create splits, or manage the terminal layout."
---

This skill provides complete knowledge of the `cmux` CLI so you can control the cmux terminal multiplexer without needing to rediscover its interface.

## Binary Location

`/Applications/cmux.app/Contents/Resources/bin/cmux`

Usually on PATH as `cmux`.

## Global Options

```
cmux [--socket PATH] [--window WINDOW] [--password PASSWORD] [--json] [--id-format refs|uuids|both] [--version] <command> [options]
```

- `--json`: Output JSON (use this when you need to parse results programmatically)
- `--id-format refs|uuids|both`: Control ID format in output (defaults to refs)
- `--socket PATH`: Override Unix socket path
- `--password PASSWORD`: Socket auth password

## Environment Variables

- `CMUX_WORKSPACE_ID` - Auto-set in cmux terminals. Default `--workspace` for all commands.
- `CMUX_TAB_ID` - Optional alias for `tab-action`/`rename-tab` default `--tab`.
- `CMUX_SURFACE_ID` - Auto-set in cmux terminals. Default `--surface`.
- `CMUX_SOCKET_PATH` - Override the default Unix socket path (`/tmp/cmux.sock`).

**Important**: Because `CMUX_WORKSPACE_ID` and `CMUX_SURFACE_ID` are auto-set, most commands work without explicit `--workspace`/`--surface` flags when run from within a cmux terminal. They default to the caller's context.

## ID/Ref System

Most commands accept IDs in these formats:
- **UUIDs**: Full unique identifiers
- **Short refs**: `window:1`, `workspace:2`, `pane:3`, `surface:4`
- **Indexes**: Numeric indexes

`tab-action` also accepts `tab:<n>` in addition to `surface:<n>`.

## Commands Reference

### Discovery & Status
| Command | Description |
|---------|-------------|
| `version` | Show cmux version |
| `ping` | Check if cmux is running |
| `capabilities` | List supported capabilities |
| `identify [--workspace] [--surface] [--no-caller]` | Identify current context |

### Windows
| Command | Description |
|---------|-------------|
| `list-windows` | List all windows |
| `current-window` | Get the current window |
| `new-window` | Create a new window |
| `focus-window --window <id>` | Focus a window |
| `close-window --window <id>` | Close a window |
| `rename-window [--workspace] <title>` | Rename a window |
| `next-window` / `previous-window` / `last-window` | Navigate windows |

### Workspaces (tabs in the UI)
| Command | Description |
|---------|-------------|
| `list-workspaces` | List all workspaces |
| `current-workspace` | Get current workspace |
| `new-workspace [--command <text>]` | Create workspace, optionally run a command |
| `select-workspace --workspace <id\|ref>` | Switch to workspace |
| `close-workspace --workspace <id\|ref>` | Close workspace |
| `rename-workspace [--workspace] <title>` | Rename workspace |
| `workspace-action --action <name> [--workspace] [--title]` | Trigger workspace action |
| `move-workspace-to-window --workspace <id\|ref> --window <id\|ref>` | Move workspace to another window |
| `reorder-workspace --workspace <id\|ref\|index> (--index <n> \| --before <id> \| --after <id>) [--window]` | Reorder workspace |

### Panes & Surfaces
| Command | Description |
|---------|-------------|
| `list-panes [--workspace]` | List panes in workspace |
| `list-pane-surfaces [--workspace] [--pane]` | List surfaces in a pane |
| `focus-pane --pane <id\|ref> [--workspace]` | Focus a pane |
| `new-pane [--type terminal\|browser] [--direction left\|right\|up\|down] [--workspace] [--url]` | Create a new pane |
| `new-split <left\|right\|up\|down> [--workspace] [--surface] [--panel]` | Split current pane |
| `new-surface [--type terminal\|browser] [--pane] [--workspace] [--url]` | Add surface (tab) to pane |
| `close-surface [--surface] [--workspace]` | Close a surface |
| `move-surface --surface <id> [--pane] [--workspace] [--window] [--before] [--after] [--index] [--focus]` | Move surface between panes |
| `reorder-surface --surface <id> (--index <n> \| --before <id> \| --after <id>)` | Reorder surface within pane |
| `drag-surface-to-split --surface <id\|ref> <left\|right\|up\|down>` | Drag surface into a new split |
| `resize-pane --pane <id\|ref> [--workspace] (-L\|-R\|-U\|-D) [--amount <n>]` | Resize pane |
| `swap-pane --pane <id\|ref> --target-pane <id\|ref> [--workspace]` | Swap two panes |
| `break-pane [--workspace] [--pane] [--surface] [--no-focus]` | Break pane into new workspace |
| `join-pane --target-pane <id\|ref> [--workspace] [--pane] [--surface] [--no-focus]` | Join pane into another |
| `last-pane [--workspace]` | Focus previous pane |
| `refresh-surfaces` | Refresh all surfaces |
| `surface-health [--workspace]` | Check surface health |
| `respawn-pane [--workspace] [--surface] [--command <cmd>]` | Respawn a pane |

### Tabs (Surface-level)
| Command | Description |
|---------|-------------|
| `tab-action --action <name> [--tab] [--surface] [--workspace] [--title] [--url]` | Trigger tab action |
| `rename-tab [--workspace] [--tab] [--surface] <title>` | Rename a tab |
| `trigger-flash [--workspace] [--surface]` | Flash a surface |

### Panels
| Command | Description |
|---------|-------------|
| `list-panels [--workspace]` | List panels |
| `focus-panel --panel <id\|ref> [--workspace]` | Focus a panel |
| `send-panel --panel <id\|ref> [--workspace] <text>` | Send text to panel |
| `send-key-panel --panel <id\|ref> [--workspace] <key>` | Send keypress to panel |

### Reading & Sending to Terminals
| Command | Description |
|---------|-------------|
| `read-screen [--workspace] [--surface] [--scrollback] [--lines <n>]` | Read terminal screen content |
| `capture-pane [--workspace] [--surface] [--scrollback] [--lines <n>]` | Alias for read-screen |
| `send [--workspace] [--surface] <text>` | Send text to a terminal |
| `send-key [--workspace] [--surface] <key>` | Send a keypress to a terminal |
| `pipe-pane --command <shell-command> [--workspace] [--surface]` | Pipe pane output to a command |
| `clear-history [--workspace] [--surface]` | Clear scrollback history |
| `find-window [--content] [--select] <query>` | Find window by content or title |

### Clipboard / Buffers
| Command | Description |
|---------|-------------|
| `set-buffer [--name <name>] <text>` | Store text in a named buffer |
| `list-buffers` | List all buffers |
| `paste-buffer [--name <name>] [--workspace] [--surface]` | Paste buffer into surface |

### Notifications
| Command | Description |
|---------|-------------|
| `notify --title <text> [--subtitle <text>] [--body <text>] [--workspace] [--surface]` | Send a notification |
| `list-notifications` | List notifications |
| `clear-notifications` | Clear notifications |

### Sidebar Metadata
| Command | Description |
|---------|-------------|
| `set-status <key> <value> [--icon <name>] [--color <#hex>] [--workspace]` | Set sidebar status |
| `clear-status <key> [--workspace]` | Clear a status key |
| `list-status [--workspace]` | List all status entries |
| `set-progress <0.0-1.0> [--label <text>] [--workspace]` | Set progress bar |
| `clear-progress [--workspace]` | Clear progress bar |
| `log [--level <level>] [--source <name>] [--workspace] [--] <message>` | Write to sidebar log |
| `clear-log [--workspace]` | Clear sidebar log |
| `list-log [--limit <n>] [--workspace]` | List log entries |
| `sidebar-state [--workspace]` | Get sidebar state |

### Browser Control

Create and control browser panes within cmux.

**Opening browsers:**
```
cmux browser open [url]              # Open browser in caller's workspace
cmux browser open-split [url]        # Open as a split
cmux new-pane --type browser [--url <url>] [--direction left|right|up|down]
```

**Navigation:**
```
cmux browser goto|navigate <url> [--snapshot-after]
cmux browser back|forward|reload [--snapshot-after]
cmux browser url|get-url
```

**Reading page content:**
```
cmux browser snapshot [--interactive|-i] [--cursor] [--compact] [--max-depth <n>] [--selector <css>]
cmux browser get <url|title|text|html|value|attr|count|box|styles> [...]
cmux browser console <list|clear>
cmux browser errors <list|clear>
```

**Interacting with elements:**
```
cmux browser click|dblclick|hover|focus|check|uncheck|scroll-into-view <selector> [--snapshot-after]
cmux browser type <selector> <text> [--snapshot-after]
cmux browser fill <selector> [text] [--snapshot-after]    # empty text clears input
cmux browser press|keydown|keyup <key> [--snapshot-after]
cmux browser select <selector> <value> [--snapshot-after]
cmux browser scroll [--selector <css>] [--dx <n>] [--dy <n>] [--snapshot-after]
```

**Querying elements:**
```
cmux browser is <visible|enabled|checked> <selector>
cmux browser find <role|text|label|placeholder|alt|title|testid|first|last|nth> ...
cmux browser highlight <selector>
```

**Waiting:**
```
cmux browser wait [--selector <css>] [--text <text>] [--url-contains <text>] [--load-state <interactive|complete>] [--function <js>] [--timeout-ms <ms>]
```

**JavaScript execution:**
```
cmux browser eval <script>
```

**Page manipulation:**
```
cmux browser frame <selector|main>
cmux browser dialog <accept|dismiss> [text]
cmux browser viewport <width> <height>
cmux browser geolocation|geo <lat> <lon>
cmux browser offline <true|false>
cmux browser addscript <script>
cmux browser addstyle <css>
cmux browser addinitscript <script>
```

**State & network:**
```
cmux browser state <save|load> <path>
cmux browser cookies <get|set|clear> [...]
cmux browser storage <local|session> <get|set|clear> [...]
cmux browser download [wait] [--path <path>] [--timeout-ms <ms>]
cmux browser network <route|unroute|requests> [...]
cmux browser trace <start|stop> [path]
cmux browser screencast <start|stop>
cmux browser input <mouse|keyboard|touch>
```

**Browser tabs:**
```
cmux browser tab <new|list|switch|close|<index>> [...]
```

**Browser identity:**
```
cmux browser identify [--surface <id|ref|index>]
```

### Hooks & Signals
| Command | Description |
|---------|-------------|
| `set-hook [--list] [--unset <event>] \| <event> <command>` | Set/list/unset hooks |
| `wait-for [-S\|--signal] <name> [--timeout <seconds>]` | Wait for or send a signal |

### Other
| Command | Description |
|---------|-------------|
| `set-app-focus <active\|inactive\|clear>` | Set app focus state |
| `simulate-app-active` | Simulate app becoming active |
| `claude-hook <session-start\|stop\|notification> [--workspace] [--surface]` | Claude session hooks |
| `display-message [-p\|--print] <text>` | Display a message |
| `bind-key` / `unbind-key` / `copy-mode` | Key binding controls |
| `popup` | Show popup |

## Common Patterns

### Discover current layout
```bash
cmux identify                    # Who am I?
cmux list-workspaces --json      # All workspaces
cmux list-panes --json           # All panes in current workspace
```

### Run a command in a new split
```bash
cmux new-split right             # Create split to the right
# Then send a command to it (need to target the new surface)
cmux send --surface <new-surface-ref> "npm run dev"
cmux send-key --surface <new-surface-ref> Enter
```

### Read what's on another terminal
```bash
cmux read-screen --surface <ref>           # Current visible content
cmux read-screen --surface <ref> --scrollback --lines 200  # With scrollback
```

### Open a browser alongside terminal
```bash
cmux browser open http://localhost:3000
# or as a directional split:
cmux new-pane --type browser --direction right --url http://localhost:3000
```

### Show progress in sidebar
```bash
cmux set-progress 0.5 --label "Building..."
cmux log --level info --source claude -- "Step 3/5 complete"
cmux set-status task "Running tests" --icon checkmark --color "#00ff00"
```

## Tips

- Always use `--json` when you need to parse output programmatically.
- When creating splits/panes, capture the output to get the new surface/pane ID for subsequent commands.
- `read-screen` is invaluable for checking what's happening in other terminals.
- `send` sends raw text; remember to follow with `send-key Enter` to execute a command.
- Browser `snapshot` gives you a DOM snapshot — use `--compact` for a smaller representation.
- Use `--snapshot-after` on browser interaction commands to get the page state after the action.
