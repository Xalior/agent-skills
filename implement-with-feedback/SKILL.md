---
name: implement-with-feedback
description: Execute a plan document into code through a disciplined, git-centric workflow — clean checkout, properly named branch, continuous WIP tracking, small meaningful commits. Use when a plan doc exists and the user is ready to build it. LOCAL-ONLY — never pushes during the implementation loop.
argument-hint: [plan-path-or-slug]
---

# Implement with Feedback

Execute a plan document into code. The plan is your source of truth for WHAT to build and HOW to phase it. You are an implementer, not a designer. Local-only: commits stay on disk until the user explicitly asks for a push.

## Preflight

**Locate the plan.** The skill starts from a completed plan. If `$ARGUMENTS` points to an existing plan doc, use it. Otherwise glob `docs/plans/plan_*.md` and ask the user to pick one. If no plan exists, STOP and tell the user this skill requires a plan doc as input — do NOT invent scope or design from scratch.

**Read the plan in full.** Use the Read tool WITHOUT limit/offset. The plan IS your source of truth for scope, phases, and Success Criteria. Do not redefine them. If they need to change, say so and OFFER to switch back to `/plan` — never switch unilaterally.

**Verify the working tree is clean.** Run `git status`. If there are ANY uncommitted or untracked non-ignored changes, STOP and tell the user:
> "Working tree is not clean. Please commit or stash your changes before starting."

**Verify we are on main/master.** If not, warn the user and ask whether to continue from the current branch or switch to main first.

**Pull latest.** `git pull` to ensure we're up to date.

**Confirm the branch name with the user.** The default is `<type>/<slug>`, where `<slug>` matches the plan's slug. Valid types:
- `feature/` — new functionality
- `bugfix/` — fixing a defect
- `spike/` — exploratory / research / prototype
- `refactor/` — restructuring without behaviour change
- `docs/` — documentation only
- `chore/` — maintenance, deps, tooling

Present the single default and ask the user to confirm or override. Do not enumerate alternatives beyond the type list above.

## The Doc

Create the implementation tracker at `docs/plan_<slug>_implementation.md` with this skeleton:

````markdown
# Implementation: <title from plan>

**Status:** In Progress
**Branch:** `<type>/<slug>`
**Plan:** `<path to plan_<slug>.md>`

## Tasks

- [ ] Phase 1: <name>
- [ ] Phase 2: <name>

## Progress Log

## Decisions & Notes

## Blockers

## Commits
````

The tracker is a living document — update it continuously. It tells the story of the implementation to anyone reading the git log. The file on disk is the persistence layer.

Commit the tracker immediately with a message like `chore: init implementation tracker for <slug>`.

## The Work

Execute the plan's phases in order. **The stance is skepticism** — if a Success Criterion isn't verifiable (Automated = a command to run; Manual = a specific thing to observe), it isn't done. NEVER mark a phase complete on vibes.

- **FOLLOW THE PLAN.** Execute its phases in the order they're written. NEVER batch across phases, NEVER skip ahead, NEVER silently merge two phases.
- **HONOR THE APPROACH.** If the plan specifies vertical slices (the default), each phase cuts end-to-end through the stack — e.g. DB → model → server → api → client lib → frontend, or whichever layers the feature touches. NEVER complete one layer across all features when the plan calls for slices. If the plan specifies something else, follow it as written.
- **INVESTIGATE BEFORE ASKING.** Before questioning the user, read the plan, read referenced files in full, and look at the current code. Spawn research sub-agents in parallel when broad coverage is needed. Then ask only what investigation can't answer. NEVER ask the user about things you could have looked up.
- **READ REFERENCED MATERIALS IN FULL.** The plan. Files named in Key Discoveries. Related code the plan references. Use the Read tool WITHOUT limit/offset. NEVER skim. NEVER summarise-and-move-on.
- **VERIFY, DO NOT ADOPT.** Claims the user makes mid-implementation — about constraints, existing behaviour, intent in the plan — get verified before they change direction. Corrections to your own statements ALSO get verified. Spawn a sub-agent to verify where possible.
- **WAIT FOR ALL SUB-TASKS TO COMPLETE** before acting on their findings. Be patient with sub-agents and vocal about them: say what you have spawned, and speak up when each one returns.
- **Update the tracker FIRST, then do the work.** Mark the current task in progress, append a progress log entry with a timestamp. Then write the code.
- **Prefer Makefile targets for verification** (`make test`, `make lint`, `make -C <subproject> check`). The plan's Success Criteria should name them; run what the plan names. If a needed target is missing, extend the Makefile as part of this phase.
- **Run ALL Automated Success Criteria before marking a phase complete.** Every checkbox. No selective verification.
- **Pause for Manual Success Criteria.** When a phase has Manual criteria, STOP after Automated checks pass and tell the user exactly what needs observing. Do NOT proceed to the next phase until they confirm.
- **Commit early, commit often, in small logical units.** One reason per commit. If you touched 5 files for 3 reasons, that's 3 commits. Prefixes:
  - `feat:` — new functionality
  - `fix:` — bug fix
  - `wip:` — partial work (always with context: `wip: partial auth middleware`, never just `wip`)
  - `docs:` — documentation
  - `test:` — tests only
  - `refactor:` — restructure without behaviour change
  - `chore:` — tooling, deps, maintenance
- **Update the tracker after each commit** — tick off tasks, append progress, note decisions or blockers.
- **If blocked, record the blocker in the tracker, commit, then ask the user.** Do NOT spin on a blocker silently.
- **When all phases are complete and all Success Criteria met**, set the tracker's Status to `Complete`, make a final commit, and tell the user the branch is ready. If they want a PR, offer to push the branch and open it.

## Never

- **NEVER push during the implementation loop.** This is a local-only workflow. Commits stay on disk until the user explicitly asks (typically at completion, when opening a PR).
- **NEVER amend commits.** Make a new commit instead.
- **NEVER force push** — including if the user later asks for a push. History is sacred in this workflow.
- Never redefine scope, phases, or Success Criteria. Those come from the plan. If they need to change, OFFER to switch back to `/plan` — never switch unilaterally, and never redefine in-place.
- Never mark a phase complete without running its Automated criteria and obtaining user confirmation on its Manual criteria.
- Never batch multiple phases into one commit or one chunk of work.
- Never commit with vague messages. `wip` alone is not a commit message.
- Never lead the user with unsolicited alternatives.
