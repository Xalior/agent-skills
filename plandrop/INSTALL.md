# plandrop installation (agent-led)

When the user asks to set up plandrop in a project — or the skill finds no domain/host
configured anywhere and the user wants to publish — walk them through it. Everything this
flow does is built to the user's answers, not to assumptions: don't pick a server, don't
mint a host, and don't decide the git question on their behalf.

Every step below is **idempotent and safe to re-run**: detect existing state first, and
skip or offer-to-update instead of asking again from scratch. Running this flow twice in
the same project should never re-prompt for something already settled, and never undo a
choice the user already made.

## Step 0: detect existing state (read-only)

Before asking anything, check what's already true:

```sh
# Per-user config — is a domain already configured for this machine?
cat "${XDG_CONFIG_HOME:-$HOME/.config}/plandrop/config.json" 2>/dev/null

# Local host — does this project already have one?
test -f .plandrop && grep -q '"host"' .plandrop && echo "host: yes" || echo "host: no"

# Autosync hook — does .claude/settings.json already carry one?
grep -q 'Publishing to plandrop' .claude/settings.json 2>/dev/null && echo "hook: yes" || echo "hook: no"

# .plandrop and git — is it already tracked, or already gitignored?
git ls-files --error-unmatch .plandrop >/dev/null 2>&1 && echo "git: tracked"
git check-ignore -q .plandrop 2>/dev/null && echo "git: ignored"
```

Use these results to skip questions below whose answer is already evident, and to fold the
existing value in as the recommended default of whichever question still applies.

## Step 1: which server (ask first, alone — it decides whether the rest apply)

**Ask the user (AskUserQuestion):** "Which plandrop server should this project use?"

- **Use the configured server `<domain>`** — only offered, and the recommended default,
  when Step 0 found one in the per-user config.
- **Enter a domain** — the user's own running plandrop stack. Recommended default when no
  per-user config exists yet and the user is asking to set up publishing.
- **plandrop.dev templates only** — no publishing; `newdoc` works, `create`/`upload` don't.
  The right pick when the user just wants to scaffold and edit a document, not host it.

**This is the deciding question.** If the answer is "templates only," stop here — there is
no host to mint, no hook to install, and no `.plandrop` to ask about. Just note that
`newdoc` is ready to use and move on to the normal workflow in `SKILL.md`.

Otherwise, resolve `<domain>`:
- "Use the configured server" → the value from Step 0's per-user config.
- "Enter a domain" → the value the user typed. If Step 0 found **no** per-user config at
  all, also persist it for next time so future projects don't re-ask:
  ```sh
  npx plandrop init --user --yes --domain <domain>
  ```
  If a per-user config already exists with a *different* domain, leave it alone — the
  user chose a different server for this project specifically; it'll be recorded in this
  project's own `.plandrop` in Step 2, which always wins locally.

## Step 2: mint a host now?

Skip this question if Step 0 already found `host: yes` — say so, and move on.

**Ask the user (AskUserQuestion):** "Mint a plandrop host for this project now?"

- **Yes, mint a host** (recommended) — the normal case when the user is setting up to
  publish.
- **No, skip for now** — `newdoc` still works against `<domain>`'s templates; `create` (or
  the wizard) can be run later.

If yes, don't run `create` yet — combine it with Step 3's answer below into a single
invocation, since `create`'s own flags cover both in one call.

## Step 3: install the autosync hook?

Skip this question if Step 0 already found `hook: yes` — say so, and move on. Also skip
entirely if Step 1 resolved to "templates only" or Step 2 resolved to "no host."

**Ask the user (AskUserQuestion):** "Install the Claude Code hook that republishes a saved
HTML doc automatically?"

- **Yes, watch `docs/**/*.html`** (recommended — matches the CLI's own default).
- **Yes, watch a different path** — ask for the glob.
- **No** — publish manually with `upload` when ready.

If the answer is yes, ask one more (**AskUserQuestion**): "Where should saved files
publish to?"

- **Relative to the watched directory** (recommended — matches the CLI's own default for
  an interactive answer): a save at `docs/plans/x.html`, watching `docs/**/*.html`,
  publishes to `/plans/x.html`.
- **Mirroring the full project path** — that same save publishes to `/docs/plans/x.html`.
- **Flat at the host root** — that same save publishes to `/x.html` regardless of depth.

`create`'s own interactive `[Y/n]` + publish-root offer for this same pair of questions
only fires at a real TTY; an agent invoking it via a tool call never hits one, so that
offer is always a no-op here. This is the only place the hook gets asked — never ask it a
second time by also answering `create`'s prompt.

Now run the commands, combining Step 2 and Step 3's answers into one `create` call so the
host and hook are set up together:

```sh
# host + hook, default glob, default (relative-to-watched-dir) publish root:
npx plandrop create --domain <domain> --hook-path "docs/**/*.html" --local-root docs

# host + hook, custom glob, mirroring the full project path (the CLI's silent default —
# no --local-root/--hook-flat needed):
npx plandrop create --domain <domain> --hook-path "<glob>"

# host + hook, flat at the host root regardless of the saved file's depth:
npx plandrop create --domain <domain> --hook-path "<glob>" --hook-flat

# host, no hook:
npx plandrop create --domain <domain> --no-hook

# host already existed (Step 0 found one) but no hook yet, or an older hook needs
# upgrading to the current shape — create wasn't run, so use the standalone installer
# instead (any of the flags above apply the same way):
npx plandrop hooks "docs/**/*.html" --local-root docs
```

`--local-root <dir>` and `--hook-flat` are mutually exclusive. Any hook-taking flag
implies `--hook`, and every choice — glob and publish root alike — is recorded into
`.plandrop` (`hookWatch`/`hookPublish`/`localRoot`), so a bare `plandrop hooks` later
re-applies them rather than asking again; this is also how a plandrop version upgrade
refreshes an older hook's shape without needing to be told the settings a second time —
an existing hook is found by fingerprint and replaced in place, never left duplicated.

`create` and `hooks` both report whether anything actually changed ("wrote …" / "updated
…" vs. "already … nothing changed") — relay that to the user rather than assuming.

**Hooks are read at session start.** Tell the user autosync begins from their *next*
session, not this one.

## Step 4: `.plandrop` and git

Skip this question entirely if no `.plandrop` exists (Step 1 was "templates only" or Step
2 was declined). Also skip if Step 0 already found `git: tracked` or `git: ignored` — the
question is already settled; just relay which, without touching anything.

**Ask the user (AskUserQuestion):** "`.plandrop` holds this host's passphrase and grants
write access to it. How should git treat it?"

- **Gitignore it** (recommended default for anything shared or public) — the cautious
  choice; nobody who clones the repo gets write access to the host for free.
- **Commit it** — a legitimate choice in a private repo: anyone with write access to the
  repo already has write access to the host, so the file adds no new exposure.
- **Decide later** — do nothing; leave it untracked and unignored.

Present both real options neutrally — this is the user's call, not a lint the agent
enforces. If they pick gitignore:

```sh
grep -qxF '.plandrop' .gitignore 2>/dev/null || echo '.plandrop' >> .gitignore
```

If they pick commit, do nothing further — `git add .plandrop` is the user's own action to
take when ready, not something this flow performs on their behalf. If they pick "decide
later," do nothing.

**Never gitignore or remove a `.plandrop` without this explicit answer, and never remove
or untrack one that's already committed** — even a helpful-looking `git rm --cached
.plandrop` on an already-tracked file is a destructive action the user didn't ask for.

## After the flow

Point back to `SKILL.md`'s workflow: `newdoc` to scaffold, edit, `upload` (or just save, if
the hook is on) to publish, share the printed link.
