---
name: cmux
description: "Control the cmux terminal multiplexer. Use this skill when the user asks to manage terminal panes, workspaces, windows, browser splits, or interact with cmux in any way. Also use when you need to read other terminal screens, send commands to other panes, create splits, or manage the terminal layout."
allowed-tools: Bash, Read
---

This skill provides complete knowledge of the `cmux` CLI so you can control the cmux terminal multiplexer without needing to rediscover its interface.

The everyday surface — global options, environment defaults, the ID/ref system, and the common patterns — is below. The **full per-command reference** lives in `references/commands.md`; reach for it when you need a command that the patterns here don't cover.

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

## Command Reference

The complete command surface — windows, workspaces, panes & surfaces, tabs, panels, terminal read/send, clipboard, notifications, sidebar metadata, the full browser-control subcommand tree, hooks & signals — is in `references/commands.md`. Read it when a task needs a command beyond the common patterns below.

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
