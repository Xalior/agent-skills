# Changelog

All notable changes to this project will be documented in this file.

The format is loosely based on [Keep a Changelog](https://keepachangelog.com/).
Patch bumps cover edits to existing skills; minor bumps cover new skills.

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
