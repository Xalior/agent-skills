# cmux — full command reference

The exhaustive command surface. SKILL.md carries the global options, env vars, ID
system, and common patterns; this file is the complete per-command list, loaded
when you need a command that isn't covered by a common pattern.

All commands honour the global options and `CMUX_*` env-var defaults documented in
SKILL.md (most commands work without explicit `--workspace`/`--surface` when run
from inside a cmux terminal).

## Discovery & Status
| Command | Description |
|---------|-------------|
| `version` | Show cmux version |
| `ping` | Check if cmux is running |
| `capabilities` | List supported capabilities |
| `identify [--workspace] [--surface] [--no-caller]` | Identify current context |

## Windows
| Command | Description |
|---------|-------------|
| `list-windows` | List all windows |
| `current-window` | Get the current window |
| `new-window` | Create a new window |
| `focus-window --window <id>` | Focus a window |
| `close-window --window <id>` | Close a window |
| `rename-window [--workspace] <title>` | Rename a window |
| `next-window` / `previous-window` / `last-window` | Navigate windows |

## Workspaces (tabs in the UI)
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

## Panes & Surfaces
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

## Tabs (Surface-level)
| Command | Description |
|---------|-------------|
| `tab-action --action <name> [--tab] [--surface] [--workspace] [--title] [--url]` | Trigger tab action |
| `rename-tab [--workspace] [--tab] [--surface] <title>` | Rename a tab |
| `trigger-flash [--workspace] [--surface]` | Flash a surface |

## Panels
| Command | Description |
|---------|-------------|
| `list-panels [--workspace]` | List panels |
| `focus-panel --panel <id\|ref> [--workspace]` | Focus a panel |
| `send-panel --panel <id\|ref> [--workspace] <text>` | Send text to panel |
| `send-key-panel --panel <id\|ref> [--workspace] <key>` | Send keypress to panel |

## Reading & Sending to Terminals
| Command | Description |
|---------|-------------|
| `read-screen [--workspace] [--surface] [--scrollback] [--lines <n>]` | Read terminal screen content |
| `capture-pane [--workspace] [--surface] [--scrollback] [--lines <n>]` | Alias for read-screen |
| `send [--workspace] [--surface] <text>` | Send text to a terminal |
| `send-key [--workspace] [--surface] <key>` | Send a keypress to a terminal |
| `pipe-pane --command <shell-command> [--workspace] [--surface]` | Pipe pane output to a command |
| `clear-history [--workspace] [--surface]` | Clear scrollback history |
| `find-window [--content] [--select] <query>` | Find window by content or title |

## Clipboard / Buffers
| Command | Description |
|---------|-------------|
| `set-buffer [--name <name>] <text>` | Store text in a named buffer |
| `list-buffers` | List all buffers |
| `paste-buffer [--name <name>] [--workspace] [--surface]` | Paste buffer into surface |

## Notifications
| Command | Description |
|---------|-------------|
| `notify --title <text> [--subtitle <text>] [--body <text>] [--workspace] [--surface]` | Send a notification |
| `list-notifications` | List notifications |
| `clear-notifications` | Clear notifications |

## Sidebar Metadata
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

## Browser Control

Surface ref goes BEFORE the subcommand: `cmux browser <surface> <subcommand> [args]`

**Opening:**
```
cmux browser open [--workspace <id>] [--window <id>] [url]
cmux browser open-split [--workspace <id>] [url]
cmux new-pane --type browser [--url <url>] [--direction left|right|up|down]
```

**Navigation:**
```
cmux browser <surface> navigate|goto <url> [--snapshot-after]
cmux browser <surface> back|forward|reload [--snapshot-after]
cmux browser <surface> url|get-url
```

**Screenshots & snapshots:**
```
cmux browser <surface> screenshot --out /path/to/file.png
cmux browser <surface> snapshot [--interactive|-i] [--cursor] [--compact] [--max-depth <n>] [--selector <css>]
```

**Data extraction:**
```
cmux browser <surface> get url|title|text|html|value|count|box|styles [--selector <css>]
cmux browser <surface> get attr <selector> --attr <name>
cmux browser <surface> eval <javascript>
cmux browser <surface> console list|clear
cmux browser <surface> errors list|clear
```

**Interaction:**
```
cmux browser <surface> click|dblclick|hover|focus <selector> [--snapshot-after]
cmux browser <surface> type <selector> <text> [--snapshot-after]
cmux browser <surface> fill <selector> [text] [--snapshot-after]
cmux browser <surface> press|key|keydown|keyup <key> [--snapshot-after]
cmux browser <surface> check|uncheck <selector> [--snapshot-after]
cmux browser <surface> select <selector> <value> [--snapshot-after]
cmux browser <surface> scroll [--selector <css>] [--dx <px>] [--dy <px>] [--snapshot-after]
cmux browser <surface> scroll-into-view <selector> [--snapshot-after]
cmux browser <surface> focus-webview
cmux browser <surface> is-webview-focused
```

**Element queries:**
```
cmux browser <surface> is visible|enabled|checked <selector>
cmux browser <surface> find role|text|label|placeholder|alt|title|testid <query> [--exact]
cmux browser <surface> find first|last <selector>
cmux browser <surface> find nth --index <n> <selector>
cmux browser <surface> highlight <selector>
```

**Waiting:**
```
cmux browser <surface> wait [--selector <css>] [--text <text>] [--url-contains <text>] [--load-state <state>] [--function <js>] [--timeout-ms <ms>]
```

**Frames, dialogs, tabs:**
```
cmux browser <surface> frame <selector|main>
cmux browser <surface> dialog accept|dismiss [text]
cmux browser <surface> tab list|new|switch|close [index|surface_id]
```

**Page manipulation:**
```
cmux browser <surface> viewport <width> <height>
cmux browser <surface> geolocation|geo <lat> <lon>
cmux browser <surface> offline <true|false>
cmux browser <surface> addscript|addinitscript <js>
cmux browser <surface> addstyle <css>
```

**State & network:**
```
cmux browser <surface> state save|load <path>
cmux browser <surface> cookies get|set|clear [--name <n>] [--domain <d>]
cmux browser <surface> storage local|session get|set|clear <key> [value]
cmux browser <surface> download [wait] [--path <p>] [--timeout-ms <ms>]
cmux browser <surface> network route|unroute <pattern> [--abort] [--body <text>]
cmux browser <surface> network requests
cmux browser <surface> trace start|stop [path]
cmux browser <surface> screencast start|stop
```

**Identity:**
```
cmux browser [--surface <id|ref|index>] identify
```

## Hooks & Signals
| Command | Description |
|---------|-------------|
| `set-hook [--list] [--unset <event>] \| <event> <command>` | Set/list/unset hooks |
| `wait-for [-S\|--signal] <name> [--timeout <seconds>]` | Wait for or send a signal |

## Other
| Command | Description |
|---------|-------------|
| `set-app-focus <active\|inactive\|clear>` | Set app focus state |
| `simulate-app-active` | Simulate app becoming active |
| `claude-hook <session-start\|stop\|notification> [--workspace] [--surface]` | Claude session hooks |
| `display-message [-p\|--print] <text>` | Display a message |
| `bind-key` / `unbind-key` / `copy-mode` | Key binding controls |
| `popup` | Show popup |
