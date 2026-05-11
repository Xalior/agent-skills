# Changelog

All notable changes to this project will be documented in this file.

The format is loosely based on [Keep a Changelog](https://keepachangelog.com/).
Patch bumps cover edits to existing skills; minor bumps cover new skills.

## [0.5.16] — 2026-04-22

### Changed
- `implement-with-remote-feedback` — terseness pass across SKILL.md and
  references. No behavioural rules dropped; rhetorical filler, hedging,
  and ornate phrasings trimmed throughout.

## [0.5.15] — 2026-04-22

### Added
- `implement-with-remote-feedback` — new Preflight step 12: elect tracker
  layout. Default `single` keeps current behaviour; `split-by-sprint`
  separates the tracker per sprint for larger work.

## [0.5.14] — 2026-04-22

### Changed
- `implement-with-remote-feedback` — Sprint 4/4. Added a 5-point Flow at
  a glance map after the intro, rewrote the Hard limits block, final
  prune pass.

## [0.5.13] — 2026-04-22

### Changed
- `implement-with-remote-feedback` — Sprint 3/4. Autonomy Contract and
  Feedback Integration Loop dissolved into a new temporally-ordered
  Execute sprints section (stop conditions, in-flight feedback rules,
  completion). Detail moved to references.

## [0.5.12] — 2026-04-22

### Changed
- `implement-with-remote-feedback` — Sprint 2/4. Sprint Grooming renamed
  to Sprint Refinement and rewritten; the Plan Immutability section was
  dissolved into directional rules; PR creation extracted to a reference.

## [0.5.11] — 2026-04-22

### Changed
- `implement-with-remote-feedback` — Sprint 1/4 of a skill rewrite. Preflight
  tightened from ~63 lines of prose to a ~20-line 11-step checklist; all
  procedural detail extracted to `references/preflight.md`.

## [0.5.10] — 2026-04-21

### Changed
- `implement-with-remote-feedback` — more guardrails against skipping
  rules already set in the plan.

## [0.5.9] — 2026-04-18

### Changed
- `implement-with-feedback`, `implement-with-remote-feedback` —
  `s/groom/refine/g` across both skills (Sprint Grooming → Sprint
  Refinement and related vocabulary).

## [0.5.8] — 2026-04-18

### Changed
- `pre-plan` renamed to `discovery`. The `pre-plan` directory is kept as
  a compatibility symlink. `plan` updated to reference the new name.

## [0.5.7] — 2026-04-17

### Changed
- `implement-with-feedback`, `implement-with-remote-feedback` — renamed
  "development phases" to "sprints" throughout, matching the new plan /
  implement vocabulary split.

## [0.5.6] — 2026-04-17

### Changed
- `plan` — nailed down the division of labour: planning produces phases,
  implementation runs in sprints.

## [0.5.5] — 2026-04-17

### Changed
- `implement-with-feedback` — backported the non-GitHub parts of the remote
  workflow (clean-tree gate, branch naming, WIP-tracker rhythm) to the
  local-only variant.

## [0.5.4] — 2026-04-17

### Changed
- `implement-with-remote-feedback`, `plan`, `pre-plan` — staging tweaks
  ahead of backporting remote-flow features to the non-git workflow.

## [0.5.3] — 2026-04-16

### Changed
- `implement-with-remote-feedback`, `plan`, `pre-plan` — additions to help
  hold context straight across long sessions.

## [0.5.2] — 2026-04-15

### Changed
- `plan`, `pre-plan` — stricter guidelines for handling questions with
  "one obvious answer" (don't fabricate a default; ask).

## [0.5.1] — 2026-04-15

### Changed
- `implement-with-remote-feedback` — Preflight rewritten as a 12-step
  questionnaire. Steps 7–12 add PR strategy election (from-start | at-end |
  per-phase | none), baseline verification audit, plan-deferred choices
  surfacing, and other up-front elections.

## [0.5.0] — 2026-04-13

### Added
- `pre-plan` — interactive requirement refinement (goal, scope, concept)
  before planning; filesystem is persistence, no git churn, no process logs.
- `plan` — interactive design of phases, changes, and success criteria;
  default vertical slices, Automated vs Manual SC split, prefer Makefile
  targets, no open questions at finalisation.

### Changed
- `implement-with-feedback` / `implement-with-remote-feedback` — restructured
  into the Preflight / The Doc / The Work / Never pattern. Now consume a
  plan doc as input; honour the plan's Approach; pause at Manual criteria;
  offer revert to `/plan` on scope drift.
- Typo `implimentation` → `implementation` corrected throughout.

## [0.4.0] — 2026-04-02

### Added
- `nicelicense` — manage open-source LICENSE files via the nicelicense CLI:
  add, change, or identify a project's license.

## [0.3.2] — 2026-03-30

### Changed
- `cmux` — updated browser-pane documentation; reinstated detail around
  managing browser splits and reading other screens.

## [0.3.1] — 2026-03-09

### Changed
- `cmux` — terseness pass: dropped from ~355 lines to ~104 to preserve
  context bloat without losing the workflow.

## [0.3.0] — 2026-03-08

### Added
- `cmux` — control the cmux terminal multiplexer: manage panes, workspaces,
  windows, browser splits, read other terminal screens, send commands to
  other panes.

## [0.2.0] — 2026-02-09

### Added
- `implement-with-feedback` — local-only, git-centric execution loop for
  long-running implementation work, with clean checkout, branch naming,
  WIP tracking, and small meaningful commits.
- `implement-with-remote-feedback` — variant that pushes after every commit
  so the remote branch becomes the live monitoring channel.

## [0.1.0] — 2026-02-08

### Added
- Initial release of the Agent Skills Collection.
- `agent-feedback` — analyze other agents' sessions and construct targeted
  corrective prompts to fix mistakes, correct context drift, or drive home
  task requirements.
