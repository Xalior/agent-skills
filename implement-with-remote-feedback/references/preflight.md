# Preflight — detailed procedure

Twelve steps. Each outcome lands in a named slot in the tracker's `Preflight Decisions` block; the slot names below match the tracker skeleton. Don't start Sprint Refinement until every step is resolved.

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

No default. The user must pick one of four — later behaviour (end-of-sprint actions, feedback loop activation) forks on this. Present verbatim:

- **from-start** — real, non-draft PR opened after Sprint Refinement, marked `[In Flight]` until completion. Body lists refined sprints as a checklist. Feedback loop runs from the first commit after refinement. Best when others need live visibility.
- **at-end** — branch-only during work. PR offered at final completion; body consolidates all sprints' manual test plans.
- **per-sprint** — a separate PR at the end of each sprint. Each stands alone, targets `main`. Prior sprint's PR unmerged when the next sprint ends → stop and ask.
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

Surface each hit upfront as a decision to make now. Resolving at the start keeps the sprint loop autonomous — mid-sprint decision prompts are exactly what the autonomy stance is trying to prevent.

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

## 12. Elect tracker layout

Ask: **"Should the tracker be split by sprint?"** Default `single` — the skeleton in SKILL.md `## The Doc`, one file. Recommend `split-by-sprint` when the work parcel is large enough that a single-file tracker will bloat: many sprints, long-running feedback loops, substantial Decisions & Notes expected. The user knows their own appetite for bloat better than a threshold rule.

Record in `Preflight Decisions → Tracker layout`.

### Layout: `split-by-sprint`

Index and per-sprint state live in sibling files under `docs/plans/`:

- `plan_<slug>_implementation.md` — **index**, session-global only.
- `plan_<slug>_implementation_sprint_<N>.md` — per-sprint state only, one per refined sprint. Created during Sprint Refinement as each sprint is written into the index's `Sprints` overview.

**What lives where:**

| Section | File |
|---|---|
| Header (Status, Branch, Plan, PR) | index |
| Preflight Decisions | index |
| Sprints (overview + links) | index |
| Phase-level Success Criteria rollup | index |
| Last-seen Feedback State | index |
| Sprint header (Name, Covers, Success Criteria, Status) | sprint file |
| Tasks | sprint file |
| Progress Log | sprint file |
| Decisions & Notes | sprint file |
| Blockers | sprint file |
| Commits | sprint file |

`Last-seen Feedback State` stays on the index: it's a session-global cursor, and migrating it between sprint files at each boundary would break the "process new items only" diff the feedback loop depends on.
