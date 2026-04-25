# Preflight — detailed procedure

Eleven steps. Each outcome lands in a named slot in the tracker's `Preflight Decisions` block; the slot names below match the tracker skeleton. Don't start Execute until every step is resolved.

## 1. Locate the plan

If `$ARGUMENTS` points to a plan doc that exists, use it. Otherwise glob `docs/plans/plan_*.md` and ask the user to pick. No plan → STOP: this skill refines a plan, it does not invent scope. Offer `/plan`.

## 2. Read the plan in full

Read tool, no `limit` or `offset`. The plan is the arbiter of scope, phases, and Success Criteria for the whole parcel; later "is this in scope?" questions get answered by re-reading the plan. Skimming here is the single most common way this workflow goes wrong — the agent proceeds from a vibes-based summary and silently reshapes the work when the summary turns out wrong.

If the plan clearly needs changes before implementation, say so and offer `/plan`. Never switch unilaterally.

## 3. Verify the working tree is clean

```
git status
```

Any uncommitted or untracked non-ignored changes → STOP with:

> "Working tree is not clean. Please commit or stash your changes before starting."

A dirty tree mixes unrelated edits into the first commits of this branch, leaking into the monitoring channel. Clean first.

## 4. Branch check

Current branch `main` or `master`? Warn and ask whether to continue from here or switch to a dev branch first. Don't assume.

## 5. Pull latest

```
git pull
```

Base current before branching.

## 6. Confirm and create the branch

Default name `<type>/<slug>`, where `<slug>` matches the plan's slug. Types:

- `feature/` — new functionality
- `bugfix/` — fixing a defect
- `spike/` — exploratory / research / prototype
- `refactor/` — restructuring without behaviour change
- `docs/` — documentation only
- `chore/` — maintenance, deps, tooling

Present **one** default; ask the user to confirm or override. Don't enumerate alternatives beyond the type list. Once confirmed:

```
git checkout -b <type>/<slug>
git push -u origin <type>/<slug>
```

Publishing immediately is deliberate — the monitoring channel exists from commit zero.

## 7. Elect PR strategy

Three options. Default `at-end` (the natural fit for this skill — "tip of the iceberg" review at the end). Present verbatim:

- **at-end** *(default)* — branch-only during the parcel; PR opened at completion, body consolidates each phase's manual test plan into one end-to-end plan. Best when the work is most reviewable as a whole.
- **from-start** — real, non-draft PR opened immediately after Preflight, marked `[In Flight]` until completion. Body lists the plan's phases as a checklist. Feedback loop runs from the first commit. Best when others need live visibility during execution.
- **none** — no PR. User handles PR creation manually later.

Record in `Preflight Decisions → PR strategy`.

## 8. Baseline verification audit

Before asking, scan the checkout for a baseline target, in order:

1. `Makefile` with a `test` / `check` / `ci` target
2. `package.json` `scripts.test`
3. `pyproject.toml` or `setup.cfg` `[tool.pytest]`
4. `go.mod` (implies `go test ./...`)
5. `Cargo.toml` (implies `cargo test`)
6. `.github/workflows/` file naming a test command

Found? Name it; ask whether to run it against the base branch now. A green baseline before the first commit means any later failure is attributable to work on this branch. Not found? Say so, proceed. Record in `Preflight Decisions → Baseline verification`.

## 9. Surface plan-deferred choices

Scan the plan for decisions the planner punted to implementation time. Markers (case-insensitive): `implementer's choice`, `TBD`, `to be decided`, `leave for implementation`, plus empty Changes Required or Success Criteria blocks.

Surface each hit upfront as a decision to make now. Resolving at the start keeps the parcel autonomous — mid-parcel decision prompts are exactly what the autonomy stance is trying to prevent. The whole point of this skill is no human pauses until the end.

Record in `Preflight Decisions → Plan-deferred decisions`.

## 10. Label setup (`from-start` only)

Only runs under `from-start`. Otherwise mark `n/a` in the tracker.

```
gh label create --force in-flight
gh label create --force do-not-merge
```

Failure (e.g. token lacks `labels:write`) is a blocker — STOP, record, ask the user. These labels are the "not ready to merge" signal the team relies on; skipping them means the PR looks mergeable when it isn't.

Record in `Preflight Decisions → In-flight labels created`.

## 11. Elect comment trust scope

Runs regardless of PR strategy. Even for `at-end` and `none`, set the scope now so a later PR has its threshold ready.

```
gh repo view --json visibility
```

Default by visibility:
- `PUBLIC` → `write`
- `PRIVATE` / `INTERNAL` → `triage`

Ask the user to confirm or pick a different minimum from the six GitHub levels, highest to lowest: `admin`, `maintain`, `write`, `triage`, `read`, `none`.

Record in `Preflight Decisions → Comment trust minimum`. The feedback loop (`references/feedback-loop.md`) resolves every comment against this value.
