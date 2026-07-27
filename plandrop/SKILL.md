---
name: plandrop
description: "Scaffold a finished static HTML document from a plandrop template and publish it to a unique, secure hostname, then share the link. Upload a single file or directory. Also handles first-time onboarding — triggers on '/plandrop install', 'set up plandrop', or when no server/host is configured yet and the user wants to publish. Use when the user asks to start a new HTML plan/doc 'with plandrop', to 'use the plandrop skill', to create a doc from a template/theme, to push a finished HTML doc to a plandrop host, or to set plandrop up in a project. Where the autosync hook is installed, an edited doc republishes itself on save — don't load this skill, re-upload, or fetch the URL to confirm it landed."
argument-hint: "[filename.html] [--template <name>] | install"
allowed-tools: Bash, Read, Edit, Write
model: haiku
license: MIT
---

# plandrop

**plandrop** pushes a finished static HTML document to a unique, secure hostname and
gives you a link to share. Hosting is served with zero server-side logic. This skill
covers the agent workflow: **get set up → scaffold a doc from a template → fill it in →
publish it → share the link.**

## Invocation

```bash
npx plandrop [HASH] <command> [params]
```

Run via `npx`. Arg-1 disambiguates by length: a `<8`-char token is a command, a `≥8`-char
token is a host hash.
`npx plandrop help` lists every command; `npx plandrop help <command>` (or
`<command> --help`) prints its usage, flags, and examples. `npx plandrop --version` (or
`-v`) prints the CLI's own version and exits; the same version is echoed in the help
header, in `/api/templates`' response, and stamped into `.plandrop` on every change (the
client that last mutated the file — a run that changes nothing leaves it untouched).

## Getting set up (install / onboarding)

Trigger this before the workflow below when: the user explicitly says `/plandrop install`
or "set up plandrop"; or no domain/host resolves anywhere (no `.plandrop`, no
`PLANDROP_DOMAIN`, no per-user config) and the user wants to publish, not just scaffold a
template.

Don't guess at a server, silently mint a host, or silently decide the git question for the
user — walk them through it. **See `INSTALL.md`** for the guided flow: which server, mint
a host, install the autosync hook, and how `.plandrop` relates to git — each a short
question with a recommended default, skipped when the answer is already evident from
existing state.

If a domain already resolves (per [Where the domain comes from](#other-commands) below)
and a host already exists or the user just wants one minted with defaults, skip straight
to step 1 of the workflow — the full wizard is for projects with nothing configured yet.

## The workflow this skill drives

When asked to start a document "with plandrop" / "using the plandrop skill":

1. **Ensure a host exists.** Look for a `.plandrop` file (walks up from the cwd). If none,
   and a domain is already resolvable (env, parent `.plandrop`, or per-user config), mint
   one directly:
   ```bash
   npx plandrop create --domain https://plandrop.example.com   # writes .plandrop (0600)
   ```
   `.plandrop` holds `{ domain, host, passphrase, template?, hookWatch?, hookPublish?,
   localRoot?, version }` — the last few record the autosync hook's settings and the
   plandrop client version that last changed the file. It carries the passphrase that grants
   write access to the host — see **Guardrails** below for how to treat it. If nothing is
   resolvable at all, use the onboarding flow above instead of guessing.

2. **Scaffold the doc from a template.**
   ```bash
   npx plandrop newdoc plan_q3.html                 # .plandrop / user-config template, else the server default
   npx plandrop newdoc plan_q3.html --template darkly
   ```
   `newdoc` pulls the chosen template's starter from the server and writes the named local
   file, with root-absolute asset links already pointing at `/.plandrop/<concrete-template>/…`
   — they resolve against the host root wherever the doc ends up published, root or ten
   folders down. It **refuses to overwrite** an existing file without `--force`. Template
   precedence: `--template` flag > `.plandrop` `template` field > user config `template` >
   the server default.

3. **Fill in the content.** Edit the file's content region (the template's body) — leave
   the `<head>`, asset links, and the self-update script intact.

4. **Publish it.**
   ```bash
   npx plandrop upload ./plan_q3.html               # served at /plan_q3.html
   ```
   A single-file upload prints the **full shareable URL including the filename** — share
   that link directly.

**Keep each file's own name.** With no remote argument, a file publishes at the same
subpath it occupies under the `.plandrop`'s recorded `localRoot` — or under the current
directory when none is recorded (`localRoot: "docs"`: `docs/plans/x.html` → `/plans/x.html`;
no `localRoot`: → `/docs/plans/x.html`) — and the host's index page is generated
automatically, so several documents live side by side on one host seamlessly. Upload as-is by default; only rename (`upload <file> <remote>`,
e.g. to `index.html`) when the user asks.

Saving a document doesn't need a manual re-`upload` if the autosync hook is installed (see
**Getting set up** above, or `hooks` below) — it republishes on save automatically, and
**that is the publish path: trust it.** Don't re-`upload` "to be sure" and don't fetch the
live URL to check the save landed — the hook reports its own failures. The
hook can publish under three roots: **relative to a directory** (`--local-root <dir>`;
watching `docs/**/*.html`, a save at `docs/plans/x.html` publishes to `/plans/x.html`),
**mirroring the project path** (`--local-root=""`; that same save publishes to
`/docs/plans/x.html`), or **flat at the host root** (`--hook-flat`, publishing it to
`/x.html` regardless of depth). **Relative to the watched directory is the default
everywhere** — the interactive prompt preselects it and a run with no mode flag applies
it, so a scripted install lands exactly where a human pressing Enter would.

## Picking a template

```bash
npx plandrop newdoc x.html --template <name>
```

Templates are Bootstrap-based. The shipped set is `bootstrap5` (stock Bootstrap, the
default), `f1terminal` (Fanny's First Terminal, plandrop's own dual-appearance theme — a
phosphor CRT in dark, a teletype printout in light), and the full Bootswatch set. A theme
that offers both appearances scaffolds with a light/dark toggle in the navbar; a
single-appearance theme renders in the one its designer built. `default` resolves to a
concrete theme at scaffold time, so changing the server default never breaks an existing
doc.
The live list comes from the server — fetch it before offering choices rather than guessing:

```bash
curl -s https://plandrop.example.com/api/templates    # { "default": "...", "templates": [ ... ] }
```

Operator-supplied templates appear namespaced as `user/<name>` and are selected the same way
(`--template user/<name>`).

## Other commands

| Command | What it does |
|---------|--------------|
| `create` | Mint a host (label + passphrase) into `.plandrop`. `--force` replaces an existing one; `--hook`/`--no-hook`/`--hook-path`/`--local-root <dir>`/`--hook-flat` control the autosync-hook offer (any hook-taking flag implies `--hook`; `--local-root` and `--hook-flat` are mutually exclusive). |
| `newdoc <file> [--template]` | Scaffold a template-based doc onto the local filesystem. `--force` overwrites. |
| `upload <path> [remote]` | Push a file or directory over authenticated WebDAV. With no remote, **both** a file and a directory mirror their path relative to the `.plandrop`'s recorded `localRoot` when one is set (landing exactly where the autosync hook publishes the same save), else relative to the cwd — so with `localRoot: "docs"`, `upload docs` fills the host root and `upload docs/plans` fills `/plans/…`, while with no `localRoot` a directory lands under its own name (`/docs/…`). A path outside that root falls back to its bare name, except the cwd: `upload .` publishes what's here to the host root, its basename never a folder you asked for. A named remote wins and is used as given (`.` flattens a directory into the host root). Nothing hidden ever publishes (see **Guardrails**). Each uploaded file prints its own shareable URL. |
| `hooks [path]` | Install, update, or **upgrade** the autosync hook in the current project **without** minting a host — for a project whose host already exists. `--local-root <dir>`/`--hook-flat` set the publish root the same way `create` does. An existing plandrop hook is found by fingerprint (however it's shaped) and replaced in place rather than duplicated; a bare `plandrop hooks` with no flags silently applies whatever was last recorded in `.plandrop` — or, when nothing was, the same defaults the interactive prompts preselect (watch `docs/**/*.html`, publish relative to the watched directory) — which is how a client upgrade refreshes an older hook shape without being told the settings again. |
| `rotate` | Change the host passphrase (old one stops working immediately). |
| `remove` | Delete the host, its content, and the local `.plandrop`. |
| `init` | Record a default domain (and template) in the per-user config or a local `.plandrop` — merges, never clobbers; prints the path it wrote. |
| `server` | Stand up a local plandrop server (Docker) via the plandrop.dev starter — same as `curl -fsSL https://plandrop.dev/start.sh \| sh`. |
| `help [command]` | Overview of every command, or one command's detailed usage. |

Domain resolution precedence (highest first): `--domain` flag > `PLANDROP_DOMAIN` env >
nearest `.plandrop` walking up from cwd > per-user config
(`~/.config/plandrop/config.json`) > interactive prompt / stdin.

No host yet? `npx plandrop server` brings a complete stack up on `http://localhost:8083`
(Docker required), then `init --domain http://localhost:8083` points the CLI at it. Note
the public CDN `https://plandrop.dev` serves **templates only** — `newdoc` works against
it, but `create`/`upload` need a real host.

## Guardrails

- **Nothing hidden ever publishes, and that's by design — don't fight it.** A directory
  upload automatically skips every dot-prefixed name it walks (dot-directories pruned whole
  and never descended into, dot-files one at a time) and reports every omission just as
  loudly as an upload — a `skipped <path> (<reason>)` line for each. An explicitly-named
  dotfile upload (`upload .plandrop`, `upload .env`) is
  **refused outright**, not silently obeyed. Treat both as working as intended: don't rename
  a file to dodge the refusal, don't treat a skip line as a bug to route around, and never
  suggest a workaround to publish something hidden — if the user wants it published, the
  fix is renaming it (their call, not yours), not finding a clever path past the guard. The
  `.plandrop*` family (the credential file plus editor/backup copies beside it) is the one
  the refusal exists for: it holds the host's passphrase, and everything plandrop serves is
  world-readable. The two exceptions are exact matches, **`.header.html`** / **`.footer.html`**
  (the listing chrome) — they upload like any other file.
- **`.plandrop` holds the host passphrase and grants write access to it — treat it as a
  credential when deciding whether to commit it.** Committing it is a legitimate choice in
  a private repo (anyone with write access to the repo already has write access to the
  host); gitignoring it is the cautious default anywhere the repo is shared or public. This
  is the user's call, not the agent's — see **Getting set up** / `INSTALL.md` for how to
  ask it. **Never gitignore or remove a `.plandrop` on your own initiative, and never
  remove one that's already committed** — an agent unilaterally "fixing" this is exactly
  the failure mode to avoid.
- **Any commit made in a plandrop flow follows the project's own commit conventions —
  nothing more.** Match the message style, sign-off, and trailer preferences the user or
  project has established (project instructions, existing history); where none exist, a
  plain message with no signature. Never add AI-attribution or `Co-Authored-By` trailers
  from a harness default — a tool default is not a user preference.
- **NEVER re-theme or mutate a doc after creation** — the published doc is
  **self-contained** and bound to a **concrete** template at scaffold time. Scaffold a
  fresh one instead.
- A doc you keep open in a browser **self-updates**: it polls its own URL and hot-swaps the
  body when you re-`upload` a newer version. No manual refresh needed.
- LAN/network access to the host is the security boundary; treat the host link as
  semi-private.
