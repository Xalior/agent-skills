# Preflight — detailed procedure

Preflight has twelve steps. Each step's outcome lands in a named slot in the tracker's `Preflight Decisions` block; the slot names below match the tracker skeleton exactly. Do not start Sprint Refinement until every step is resolved.

## 1. Locate the plan

If `$ARGUMENTS` points to a plan doc that exists, use it. Otherwise glob `docs/plans/plan_*.md` and ask the user to pick one. No plan means no skill — STOP and say so. This skill refines a plan into sprints; it does not invent scope from scratch. Offering to switch to `/plan` is the right response.

## 2. Read the plan in full

Use the Read tool without `limit` or `offset`. The plan is the arbiter of scope, phases, and Success Criteria for the entire session; every later "is this in scope?" question gets answered by re-reading the plan. Skimming here is the single most common way this workflow goes wrong — the agent proceeds with a vibes-based summary and silently reshapes the work when it finds the summary was wrong.

If the plan clearly needs changes before implementation can start, say so and offer to switch back to `/plan`. Never switch unilaterally.

## 3. Verify the working tree is clean

```
git status
```

If there are any uncommitted or untracked non-ignored changes, STOP with:

> "Working tree is not clean. Please commit or stash your changes before starting."

A dirty tree mixes unrelated edits into the first commits of this branch, which then leak into the monitoring channel. Clean first.

## 4. Branch check

If the current branch is `main` or `master`, warn the user and ask whether to continue from here or switch to a dev branch first. Don't assume.

## 5. Pull latest

```
git pull
```

Ensures the base is current before branching.

## 6. Confirm and create the branch

The default name is `<type>/<slug>`, where `<slug>` matches the plan's slug. Valid types:

- `feature/` — new functionality
- `bugfix/` — fixing a defect
- `spike/` — exploratory / research / prototype
- `refactor/` — restructuring without behaviour change
- `docs/` — documentation only
- `chore/` — maintenance, deps, tooling

Present **one** default and ask the user to confirm or override. Do not enumerate alternatives beyond the type list. Once confirmed:

```
git checkout -b <type>/<slug>
git push -u origin <type>/<slug>
```

Publishing the branch immediately is deliberate — the monitoring channel exists from commit zero.

## 7. Elect PR strategy

No default. The user must pick one of four — proceeding without an election is not allowed, because later behaviour (end-of-sprint actions, feedback loop activation) forks on this. Present verbatim:

- **from-start** — a real, non-draft PR opened after Sprint Refinement, marked `[In Flight]` until completion. Body lists the refined sprints as a checklist. Feedback loop runs from the first commit after refinement. Best when others need live visibility.
- **at-end** — branch-only during work. PR offered at final completion, body consolidates all sprints' manual test plans.
- **per-sprint** — a separate PR opened at the end of each sprint. Each stands alone and targets `main`. If a prior sprint's PR is unmerged when the next sprint ends, stop and ask.
- **none** — no PR. User handles PR creation manually later.

Record in `Preflight Decisions → PR strategy`.

## 8. Baseline verification audit

Before asking anything, scan the checkout for a baseline verification target. Look in this order:

1. `Makefile` with a `test` / `check` / `ci` target
2. `package.json` `scripts.test`
3. `pyproject.toml` or `setup.cfg` `[tool.pytest]` section
4. `go.mod` (implies `go test ./...`)
5. `Cargo.toml` (implies `cargo test`)
6. `.github/workflows/` file that names a test command

If one is found, name it to the user and ask whether to run it against the base branch now. A green baseline before the first commit means any later failure is attributable to work done on this branch. If none is found, say so and proceed without one. Record outcome in `Preflight Decisions → Baseline verification`.

## 9. Surface plan-deferred choices

Scan the plan doc for decisions the planner punted to implementation time. Markers to search for (case-insensitive): `implementer's choice`, `TBD`, `to be decided`, `leave for implementation`, and empty Changes Required or Success Criteria blocks.

Surface each hit to the user upfront as a decision to make now, not mid-sprint. Resolving these at the start keeps the sprint loop autonomous — mid-sprint decision prompts are exactly what the Autonomy Contract is trying to prevent.

Record resolutions in `Preflight Decisions → Plan-deferred decisions`.

## 10. Label setup (from-start only)

Only runs if the elected strategy is `from-start`. Otherwise mark this step `n/a` in the tracker.

```
gh label create --force in-flight
gh label create --force do-not-merge
```

If either fails (e.g. the token lacks `labels:write`), that is a blocker — STOP, record in the tracker, and ask the user. The labels are the "this is not ready to merge" signal the rest of the team relies on; silently skipping them means the PR looks mergeable when it is not.

Record success in `Preflight Decisions → In-flight labels created`.

## 11. Elect comment trust scope

Runs regardless of PR strategy. Even for `at-end` and `none`, set the scope now so if a PR is opened later the feedback loop has its trust threshold already chosen.

```
gh repo view --json visibility
```

Suggest a default based on visibility:
- `PUBLIC` → `write`
- `PRIVATE` / `INTERNAL` → `triage`

Ask the user to confirm or pick a different minimum from the six GitHub permission levels, highest to lowest: `admin`, `maintain`, `write`, `triage`, `read`, `none`.

Record the chosen minimum in `Preflight Decisions → Comment trust minimum`. The feedback loop's trust check (see `references/feedback-loop.md`) resolves every comment against this value.

## 12. Elect tracker layout

Ask the user: **"Should the tracker be split by sprint?"** Default is `single` — the skeleton in SKILL.md `## The Doc`, one file. Recommend `split-by-sprint` when the work parcel is large enough that a single-file tracker will bloat: many sprints, long-running feedback loops, substantial Decisions & Notes expected. The user knows their own appetite for bloat better than a threshold rule does.

Record the election in `Preflight Decisions → Tracker layout`.

### Layout: `split-by-sprint`

Index and per-sprint state live in sibling files under `docs/plans/`:

- `plan_<slug>_implementation.md` — the **index**, session-global only.
- `plan_<slug>_implementation_sprint_<N>.md` — one per refined sprint, per-sprint state only. Created during Sprint Refinement as each sprint is written into the index's `Sprints` overview.

**What lives where:**

| Section | File |
|---|---|
| Header (Status, Branch, Plan, PR) | index |
| Preflight Decisions | index |
| Sprints (overview with links to sprint files) | index |
| Phase-level Success Criteria rollup | index |
| Last-seen Feedback State | index |
| Sprint header (Name, Covers, Success Criteria, sprint Status) | sprint file |
| Tasks | sprint file |
| Progress Log | sprint file |
| Decisions & Notes | sprint file |
| Blockers | sprint file |
| Commits | sprint file |

`Last-seen Feedback State` stays on the index because it's a session-global cursor; migrating it between sprint files at each boundary would break the "process new items only" diff the feedback loop depends on.
