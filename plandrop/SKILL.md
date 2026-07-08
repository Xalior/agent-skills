---
name: plandrop
description: "Scaffold a finished static HTML document from a plandrop template and publish it to a unique, secure hostname on the local network, then share the link. Use when the user asks to start a new HTML plan/doc 'with plandrop', to 'use the plandrop skill', to create a doc from a template/theme, or to push a finished HTML doc to the plandrop host."
argument-hint: "[filename.html] [--template <name>]"
allowed-tools: Bash, Read, Edit, Write
license: MIT
---

# plandrop

**plandrop** pushes a finished static HTML document to a unique, secure hostname on the
local network and gives you a link to share. Hosting is served with zero server-side
logic. This skill covers the agent workflow: **scaffold a doc from a template → fill it
in → publish it → share the link.**

## Invocation

```bash
npx plandrop [HASH] <command> [params]
```

Run via `npx`. Arg-1 disambiguates by length: a `<8`-char token is a command, a `≥8`-char
token is a host hash.
`npx plandrop help` lists every command; `npx plandrop help <command>` (or
`<command> --help`) prints its usage, flags, and examples.

## The workflow this skill drives

When asked to start a document "with plandrop" / "using the plandrop skill":

1. **Ensure a host exists.** Look for a `.plandrop` file (walks up from the cwd). If none,
   mint one:
   ```bash
   npx plandrop create --domain https://plandrop.example.com   # writes .plandrop (0600)
   ```
   `.plandrop` holds `{ domain, host, passphrase, template? }`. **Never commit it** — it
   carries the passphrase. The domain can also come from `PLANDROP_DOMAIN`, a parent
   `.plandrop`, `~/.config/plandrop/config.json`, or a system config
   (`$XDG_CONFIG_DIRS`/`/etc/plandrop/`). An interactive `create` also offers to scaffold
   a Claude Code hook that republishes watched HTML docs on save (`--hook`/`--no-hook`/
   `--hook-path <glob>` control it non-interactively).

2. **Scaffold the doc from a template.**
   ```bash
   npx plandrop newdoc plan_q3.html                 # uses the .plandrop template, else `default`
   npx plandrop newdoc plan_q3.html --template darkly
   ```
   `newdoc` pulls the chosen template's starter from the server and writes the named local
   file, with asset links already pointing at `.plandrop/<concrete-template>/…` (served
   same-origin once published). It **refuses to overwrite** an existing file without
   `--force`. Template precedence: `--template` flag > `.plandrop` `template` field >
   user config `template` > the server default.

3. **Fill in the content.** Edit the file's content region (the template's body) — leave
   the `<head>`, asset links, and the self-update script intact.

4. **Publish it.**
   ```bash
   npx plandrop upload ./plan_q3.html               # served at /plan_q3.html
   ```
   A single-file upload prints the **full shareable URL including the filename** — share
   that link directly.

**Keep each file's own name.** A document publishes under its filename and the host's
index page is generated automatically, so several documents live side by side on one host
seamlessly. Upload as-is by default; only rename (`upload <file> <remote>`, e.g. to
`index.html`) when the user asks.

## Picking a template

```bash
npx plandrop newdoc x.html --template <name>
```

Templates are Bootstrap-based (the shipped set derives from Bootswatch); `default` resolves
to a concrete theme at scaffold time, so changing the server default never breaks an
existing doc.
The live list comes from the server — fetch it before offering choices rather than guessing:

```bash
curl -s https://plandrop.example.com/api/templates    # { "default": "...", "templates": [ ... ] }
```

Operator-supplied templates appear namespaced as `user/<name>` and are selected the same way
(`--template user/<name>`).

## Other commands

| Command | What it does |
|---------|--------------|
| `create` | Mint a host (label + passphrase) into `.plandrop`. `--force` replaces an existing one; `--hook`/`--no-hook`/`--hook-path` control the autosync-hook offer. |
| `newdoc <file> [--template]` | Scaffold a template-based doc onto the local filesystem. `--force` overwrites. |
| `upload <path> [remote]` | Push a file or directory (recursively) over authenticated WebDAV. Single files print their full URL; directories print the host root. |
| `rotate` | Change the host passphrase (old one stops working immediately). |
| `remove` | Delete the host, its content, and the local `.plandrop`. |
| `init` | Record a default domain (and template) in the per-user config or a local `.plandrop` — merges, never clobbers; prints the path it wrote. |
| `server` | Stand up a local plandrop server (Docker) via the plandrop.dev starter — same as `curl -fsSL https://plandrop.dev/start.sh \| sh`. |
| `help [command]` | Overview of every command, or one command's detailed usage. |

No host yet? `npx plandrop server` brings a complete stack up on `http://localhost:8083`
(Docker required), then `init --domain http://localhost:8083` points the CLI at it. Note
the public CDN `https://plandrop.dev` serves **templates only** — `newdoc` works against
it, but `create`/`upload` need a real host.

## Guardrails

- **NEVER commit `.plandrop`.** It carries the host passphrase.
- **NEVER re-theme or mutate a doc after creation** — the published doc is
  **self-contained** and bound to a **concrete** template at scaffold time. Scaffold a
  fresh one instead.
- A doc you keep open in a browser **self-updates**: it polls its own URL and hot-swaps the
  body when you re-`upload` a newer version. No manual refresh needed.
- LAN access is the security boundary; treat the host link as semi-private.
