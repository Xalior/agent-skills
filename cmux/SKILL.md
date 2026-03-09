---
name: cmux
description: "Control the cmux terminal multiplexer. Use this skill when the user asks to manage terminal panes, workspaces, windows, browser splits, or interact with cmux in any way. Also use when you need to read other terminal screens, send commands to other panes, create splits, or manage the terminal layout."
---

cmux CLI reference. Binary: `/Applications/cmux.app/Contents/Resources/bin/cmux` (usually on PATH).

## Globals & Environment

```
cmux [--socket PATH] [--window ID] [--password PWD] [--json] [--id-format refs|uuids|both] <command>
```

`--json` for programmatic output. IDs: UUIDs, short refs (`window:1`, `workspace:2`, `pane:3`, `surface:4`), or indexes.

Auto-set env vars (`CMUX_WORKSPACE_ID`, `CMUX_SURFACE_ID`) mean most commands work without explicit `--workspace`/`--surface` when run inside cmux.

| Variable | Purpose |
|----------|---------|
| `CMUX_WORKSPACE_ID` | Auto-set. Default `--workspace` |
| `CMUX_SURFACE_ID` | Auto-set. Default `--surface` |
| `CMUX_TAB_ID` | Optional default `--tab` |
| `CMUX_SOCKET_PATH` | Override socket path (default `/tmp/cmux.sock`) |
| `CMUX_SOCKET_ENABLE` | Force socket on/off (`1`/`0`) |
| `CMUX_SOCKET_MODE` | `cmuxOnly` (default), `allowAll`, `off` |
| `TERM_PROGRAM` | `ghostty` inside cmux |
| `TERM` | `xterm-ghostty` inside cmux |

**Detection:** `[ -n "${CMUX_WORKSPACE_ID:-}" ]` = inside cmux. `[ -S "${CMUX_SOCKET_PATH:-/tmp/cmux.sock}" ]` = socket available.

## Socket API

JSON-RPC over Unix socket. Send newline-terminated JSON:
```
{"id":"1","method":"workspace.list","params":{}} → {"id":"1","ok":true,"result":{...}}
```

Socket paths: `/tmp/cmux.sock` (release), `/tmp/cmux-debug.sock` (debug). Access modes: `off`, `cmuxOnly` (default), `allowAll`.

Run `cmux capabilities --json` for full method list. CLI→socket naming: `list-workspaces`→`workspace.list`, `new-split`→`surface.split`, `send`→`surface.send_text`, `read-screen`→`surface.read_text`, `notify`→`notification.create`, `browser click`→`browser.click`, etc.

```python
# Python socket client
import json, os, socket
def rpc(method, params=None):
    with socket.socket(socket.AF_UNIX, socket.SOCK_STREAM) as s:
        s.connect(os.environ.get("CMUX_SOCKET_PATH", "/tmp/cmux.sock"))
        s.sendall(json.dumps({"id":"1","method":method,"params":params or {}}).encode() + b"\n")
        return json.loads(s.recv(65536))
```

## CLI Commands

### Discovery
`version` `ping` `capabilities` `identify [--workspace] [--surface] [--no-caller]`

### Windows
`list-windows` `current-window` `new-window` `focus-window --window <id>` `close-window --window <id>` `rename-window [--workspace] <title>` `next-window` `previous-window` `last-window`

### Workspaces
`list-workspaces` `current-workspace` `new-workspace [--command <text>]` `select-workspace --workspace <id>` `close-workspace --workspace <id>` `rename-workspace [--workspace] <title>` `workspace-action --action <name> [--workspace] [--title]` `move-workspace-to-window --workspace <id> --window <id>` `reorder-workspace --workspace <id> (--index <n>|--before <id>|--after <id>) [--window]`

### Panes & Surfaces
`list-panes [--workspace]` `list-pane-surfaces [--workspace] [--pane]` `focus-pane --pane <id> [--workspace]` `new-pane [--type terminal|browser] [--direction left|right|up|down] [--workspace] [--url]` `new-split <left|right|up|down> [--workspace] [--surface] [--panel]` `new-surface [--type terminal|browser] [--pane] [--workspace] [--url]` `close-surface [--surface] [--workspace]` `move-surface --surface <id> [--pane] [--workspace] [--window] [--before] [--after] [--index] [--focus]` `reorder-surface --surface <id> (--index <n>|--before <id>|--after <id>)` `drag-surface-to-split --surface <id> <left|right|up|down>` `resize-pane --pane <id> [--workspace] (-L|-R|-U|-D) [--amount <n>]` `swap-pane --pane <id> --target-pane <id> [--workspace]` `break-pane [--workspace] [--pane] [--surface] [--no-focus]` `join-pane --target-pane <id> [--workspace] [--pane] [--surface] [--no-focus]` `last-pane [--workspace]` `refresh-surfaces` `surface-health [--workspace]` `respawn-pane [--workspace] [--surface] [--command <cmd>]`

### Tabs & Panels
`tab-action --action <name> [--tab] [--surface] [--workspace] [--title] [--url]` `rename-tab [--workspace] [--tab] [--surface] <title>` `trigger-flash [--workspace] [--surface]`
`list-panels [--workspace]` `focus-panel --panel <id> [--workspace]` `send-panel --panel <id> [--workspace] <text>` `send-key-panel --panel <id> [--workspace] <key>`

### Terminal I/O
`read-screen [--workspace] [--surface] [--scrollback] [--lines <n>]` (alias: `capture-pane`)
`send [--workspace] [--surface] <text>` — sends raw text, follow with `send-key Enter` to execute
`send-key [--workspace] [--surface] <key>` `pipe-pane --command <cmd> [--workspace] [--surface]` `clear-history [--workspace] [--surface]` `find-window [--content] [--select] <query>`

### Buffers
`set-buffer [--name <name>] <text>` `list-buffers` `paste-buffer [--name <name>] [--workspace] [--surface]`

### Notifications
`notify --title <text> [--subtitle <text>] [--body <text>] [--workspace] [--surface]` `list-notifications` `clear-notifications`

### Sidebar
`set-status <key> <value> [--icon <name>] [--color <#hex>] [--workspace]` `clear-status <key> [--workspace]` `list-status [--workspace]` `set-progress <0.0-1.0> [--label <text>] [--workspace]` `clear-progress [--workspace]` `log [--level info|progress|success|warning|error] [--source <name>] [--workspace] [--] <message>` `clear-log [--workspace]` `list-log [--limit <n>] [--workspace]` `sidebar-state [--workspace]`

### Hooks & Other
`set-hook [--list] [--unset <event>] | <event> <command>` `wait-for [-S|--signal] <name> [--timeout <s>]` `set-app-focus <active|inactive|clear>` `simulate-app-active` `claude-hook <session-start|stop|notification> [--workspace] [--surface]` `display-message [-p] <text>` `bind-key` `unbind-key` `copy-mode` `popup`

## Browser Commands

All accept surface target: `cmux browser [surface:N|--surface ID] <subcommand>`. Most mutation commands support `--snapshot-after`.

**Open:** `open [url]` `open-split [url]` — or `cmux new-pane --type browser [--url <url>] [--direction left|right|up|down]`

**Navigate:** `goto|navigate <url>` `back` `forward` `reload` `url` `focus-webview` `is-webview-focused`

**Inspect:** `snapshot [--interactive|-i] [--cursor] [--compact] [--max-depth <n>] [--selector <css>]` `screenshot --out <path>` `get <title|url|text|html|value|count|box> [selector]` `get attr <sel> --attr <name>` `get styles <sel> --property <prop>` `console <list|clear>` `errors <list|clear>`

**Interact:** `click|dblclick|hover|focus|check|uncheck|scroll-into-view <selector>` `type <selector> <text>` `fill <selector> [text]` (empty=clear) `press|keydown|keyup <key>` `select <selector> <value>` `scroll [--selector <css>] [--dx <n>] [--dy <n>]`

**Query:** `is <visible|enabled|checked> <selector>` `find <role|text|label|placeholder|alt|title|testid|first|last|nth> <value> [--name <n>]` `highlight <selector>`

**Wait:** `wait [--selector <css>] [--text <text>] [--url-contains <text>] [--load-state interactive|complete] [--function <js>] [--timeout-ms <ms>]`

**JS/Inject:** `eval <expression>` `addscript <script>` `addstyle <css>` `addinitscript <script>`

**Page:** `frame <selector|main>` `dialog <accept|dismiss> [text]` `viewport <w> <h>` `geolocation|geo <lat> <lon>` `offline <true|false>`

**State:** `state <save|load> <path>` `cookies <get|set|clear> [--name <n>] [--domain <d>] [--path <p>] [--all]` `storage <local|session> <get|set|clear> [key] [value]` `download [--path <p>] [--timeout-ms <ms>]` `network <route|unroute|requests>` `trace <start|stop> [path]` `screencast <start|stop>` `input <mouse|keyboard|touch>`

**Tabs:** `tab <list|new|switch|close> [url|index|surface-id]`

**Identity:** `identify [--surface <id>]`

## Patterns

```bash
# Layout discovery
cmux identify --json; cmux list-workspaces --json; cmux list-panes --json

# Run command in new split
cmux new-split right  # capture output for new surface ref
cmux send --surface <ref> "npm run dev" && cmux send-key --surface <ref> Enter

# Read another terminal
cmux read-screen --surface <ref> --scrollback --lines 200

# Browser: navigate and verify
cmux browser open http://localhost:3000
cmux browser wait --load-state complete && cmux browser snapshot --compact

# Browser: fill form
cmux browser fill "#email" "user@example.com"
cmux browser click "button[type=submit]" --snapshot-after
cmux browser wait --text "Welcome"

# Browser: debug
cmux browser console list && cmux browser errors list
cmux browser screenshot --out /tmp/debug.png

# Browser: session persistence
cmux browser state save /tmp/session.json  # later: state load /tmp/session.json

# Sidebar progress
cmux set-progress 0.5 --label "Building..."
cmux log --level info --source claude -- "Step 3/5 complete"
cmux set-status task "Running tests" --icon checkmark --color "#00ff00"
```

## Tips
- `--json` for programmatic parsing. Capture split/pane output to get new surface IDs.
- `send` is raw text; follow with `send-key Enter` to execute.
- `snapshot --compact` for smaller DOM output. `--snapshot-after` on interactions to verify.
- `read-screen` to check other terminals. `--scrollback --lines N` for history.
