# Preflight — detailed procedure

Nine steps. Each outcome lands in a named slot in the tracker's `Preflight Decisions` block; the slot names below match the tracker skeleton. Don't start Sprint Refinement until every step is resolved.

## 1. Locate the plan

If `$ARGUMENTS` points to a plan doc that exists, use it. Otherwise glob `docs/plans/plan_*.md` and ask the user to pick. No plan → STOP: this skill refines a plan, it does not invent scope. Offer `/plan`.

## 2. Read the plan in full

Read tool, no `limit` or `offset`. The plan is the arbiter of scope, phases, and Success Criteria for the whole session; later "is this in scope?" questions get answered by re-reading the plan. Skimming here is the single most common way this workflow goes wrong — the agent proceeds from a vibes-based summary and silently reshapes the work when the summary turns out wrong.

If the plan clearly needs changes before implementation, say so and offer `/plan`. Never switch unilaterally.

## 3. Verify the working tree is clean

```
git status
```

Any uncommitted or untracked non-ignored changes → STOP with:

> "Working tree is not clean. Please commit or stash your changes before starting."

A dirty tree mixes unrelated edits into the first commits of this branch. Clean first.

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
```

**Do not push.** This is a local-only workflow; the branch stays on disk until the user explicitly asks for a push (typically at completion, when opening a PR).

## 7. Baseline verification audit

Before asking, scan the checkout for a baseline target, in order:

1. `Makefile` with a `test` / `check` / `ci` target
2. `package.json` `scripts.test`
3. `pyproject.toml` or `setup.cfg` `[tool.pytest]`
4. `go.mod` (implies `go test ./...`)
5. `Cargo.toml` (implies `cargo test`)
6. `.github/workflows/` file naming a test command

Found? Name it; ask whether to run it against the base branch now. A green baseline before the first commit means any later failure is attributable to work on this branch. Not found? Say so, proceed. Record in `Preflight Decisions → Baseline verification`.

## 8. Surface plan-deferred choices

Scan the plan for decisions the planner punted to implementation time. Markers (case-insensitive): `implementer's choice`, `TBD`, `to be decided`, `leave for implementation`, plus empty Changes Required or Success Criteria blocks.

Surface each hit upfront as a decision to make now. Resolving at the start keeps the sprint loop autonomous — mid-sprint decision prompts are exactly what the autonomy stance is trying to prevent.

Record in `Preflight Decisions → Plan-deferred decisions`.

## 9. Elect tracker layout

Ask: **"Should the tracker be split by sprint?"** Default `single` — the skeleton in SKILL.md `## The Doc`, one file. Recommend `split-by-sprint` when the work parcel is large enough that a single-file tracker will bloat: many sprints, substantial Decisions & Notes expected. The user knows their own appetite for bloat better than a threshold rule.

Record in `Preflight Decisions → Tracker layout`.

### Layout: `split-by-sprint`

Index and per-sprint state live in sibling files under `docs/plans/`:

- `plan_<slug>_implementation.md` — **index**, session-global only.
- `plan_<slug>_implementation_sprint_<N>.md` — per-sprint state only, one per refined sprint. Created during Sprint Refinement as each sprint is written into the index's `Sprints` overview.

**What lives where:**

| Section | File |
|---|---|
| Header (Status, Branch, Plan) | index |
| Preflight Decisions | index |
| Sprints (overview + links) | index |
| Phase-level Success Criteria rollup | index |
| Sprint header (Name, Covers, Success Criteria, Status) | sprint file |
| Tasks | sprint file |
| Progress Log | sprint file |
| Decisions & Notes | sprint file |
| Blockers | sprint file |
| Commits | sprint file |
