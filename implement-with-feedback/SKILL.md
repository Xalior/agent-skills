---
name: implement-with-feedback
description: Execute a plan document into code through a disciplined, git-centric workflow — plan's phases groomed into reviewable sprints up front, clean checkout, properly named branch, continuous WIP tracking, small meaningful commits. Use when a plan doc exists and the user is ready to build it. LOCAL-ONLY — never pushes during the implementation loop.
argument-hint: [plan-path-or-slug]
---

# Implement with Feedback

Execute a plan document into code. The plan defines **phases** (design units); implementation grooms them into **sprints** (reviewable execution units) and ships sprint by sprint. You are an implementer, not a designer. Local-only: commits stay on disk until the user explicitly asks for a push.

## Preflight

Preflight is a questionnaire. Work through the eight steps in order; every election has a named destination in the implementation tracker's `Preflight Decisions` section. The agent MUST NOT begin Sprint Grooming until all eight steps are complete.

1. **Locate the plan.** If `$ARGUMENTS` points to an existing plan doc, use it. Otherwise glob `docs/plans/plan_*.md` and ask the user to pick one. If no plan exists, STOP and tell the user this skill requires a plan doc as input — do NOT invent scope or design from scratch.

2. **Read the plan in full.** Use the Read tool WITHOUT limit/offset. The plan IS your source of truth for scope, phases, and Success Criteria. Do not redefine them. If they need to change, say so and OFFER to switch back to `/plan` — never switch unilaterally.

3. **Verify the working tree is clean.** Run `git status`. If there are ANY uncommitted or untracked non-ignored changes, STOP and tell the user:
   > "Working tree is not clean. Please commit or stash your changes before starting."

4. **Verify if we are on main/master.** If so, warn the user and ask whether to continue from the current branch or switch to a dev branch first.

5. **Pull latest.** `git pull` to ensure we're up to date.

6. **Confirm the branch name with the user.** The default is `<type>/<slug>`, where `<slug>` matches the plan's slug. Valid types:
   - `feature/` — new functionality
   - `bugfix/` — fixing a defect
   - `spike/` — exploratory / research / prototype
   - `refactor/` — restructuring without behaviour change
   - `docs/` — documentation only
   - `chore/` — maintenance, deps, tooling

   Present the single default and ask the user to confirm or override. Do not enumerate alternatives beyond the type list above. Once confirmed, create the branch locally (do NOT push):

   ```
   git checkout -b <type>/<slug>
   ```

7. **Baseline verification audit.** Before asking anything, scan the checkout for a baseline verification target. Look (in this order) for:
   - a `Makefile` with a `test` / `check` / `ci` target,
   - a `package.json` `scripts.test`,
   - a `pyproject.toml` / `setup.cfg` `[tool.pytest]` section,
   - a `go.mod` (implies `go test ./...`),
   - a `Cargo.toml` (implies `cargo test`),
   - a `.github/workflows/` file that names a test command.

   If one is found, name it to the user and ask whether to run it against the base branch now before starting work. If none is found, say so and proceed without one. Record the outcome in the tracker's `Preflight Decisions → Baseline verification`.

8. **Surface plan-deferred choices.** Scan the plan doc for text matching `implementer's choice`, `TBD`, `to be decided`, `leave for implementation`, or an empty Changes Required / Success Criteria block. Surface each match to the user as an upfront decision. Record resolutions in the tracker's `Preflight Decisions → Plan-deferred decisions`.

## The Doc

Create the implementation tracker at `docs/plans/plan_<slug>_implementation.md` with this skeleton:

````markdown
# Implementation: <title from plan>

**Status:** In Progress
**Branch:** `<type>/<slug>`
**Plan:** `<path to plan_<slug>.md>`

## Preflight Decisions

- **Baseline verification:** `<command run + result, or "no baseline target found">`
- **Plan-deferred decisions:**
  - `<item>`: `<resolution>`

## Sprints

<!-- Filled in during Sprint Grooming, one entry per sprint -->

## Tasks

<!-- Sprint-keyed progress checklist. Filled in during Sprint Grooming. -->

## Progress Log

## Decisions & Notes

## Blockers

## Commits
````

The tracker is a living document — update it continuously. The file on disk is the persistence layer.

Commit the tracker immediately with a message like `chore: init implementation tracker for <slug>`.

## Sprint Grooming

After creating the tracker and before touching any code, groom the plan's phases into sprints. **A sprint is a managably-reviewable, shippable unit of work** — small enough that a reviewer can hold the whole change in their head in one sitting, large enough to be meaningful on its own. The review boundary drives sprint shape, not the plan's phase boundary.

Sprints and phases are orthogonal. Grooming maps one onto the other:

- A sprint can cover **one whole phase** — the simple case.
- A sprint can cover **several small phases** — e.g. a "bootstrap" sprint covering repo-init + deps + CI + linter-config phases from the plan.
- A sprint can cover **part of a large phase** — a big phase groomed into multiple sprints (e.g. "auth rewrite" → read-path sprint, write-path sprint, migration sprint).

### Procedure

Propose the full sprint breakdown in ONE pass. Do NOT round-robin with the user sprint-by-sprint — that is slow and loses the whole-picture view that makes good grooming possible.

1. Write the proposal directly into the tracker's `Sprints` section. For each proposed sprint, fill in:
   - **Name** — short, descriptive.
   - **Covers** — which phase(s) by name, and what portion of each (`full`, or `Changes Required items X and Y`, or `read-path only`).
   - **Success Criteria** — split into Automated and Manual, drawn from the covered phases' criteria. Where a sprint covers only part of a phase, include only the portion of that phase's criteria the sprint is responsible for.
   - **Status** — `not started`.
2. Populate the tracker's `Tasks` checklist with one entry per proposed sprint (`- [ ] Sprint 1: <name>`).
3. Keep **vertical slices the default**. Each sprint should be a thin end-to-end cut through the layers it touches, unless the plan's Approach section says otherwise.
4. Surface the completed proposal to the user and ask them to accept or nudge. Their response is a decision — log it to the `Progress Log` with a timestamp.
5. Once accepted, commit the tracker with a message like `chore: groom <slug> into N sprints`.

### Sizing for manageable review

Err small. A sprint that would take a reviewer more than an hour to review carefully is probably two sprints. Signals a sprint is too large:

- More than ~5 distinct logical units of change.
- Touches unrelated subsystems that don't need to land together.
- Would require the reviewer to hold two separate mental models at once.

Signals a sprint is too small:

- Doesn't ship anything end-to-end (unless the plan's Approach explicitly non-slices).
- Can't be verified independently.
- Has no meaningful Success Criteria of its own.

### Re-grooming mid-flight

If executing Sprint N surfaces that a later sprint is wrong — wrong split, wrong order, missing scope, too large — that is a **blocker-class decision**. Stop, record in the tracker, surface to the user, and re-groom only with their explicit go-ahead. NEVER silently reshape future sprints.

If re-grooming would change phase-level Success Criteria or scope, that is a plan-scope change, per Plan Immutability — offer to switch back to `/plan`.

## Plan Immutability

The plan is set in stone during implementation. Two exceptions only:

- The plan itself instructs a return to plan mode (e.g. "implement Phase 1, then return to `/plan` for Phase 2").
- A direct user instruction during implementation ("add that to the plan").

Anything else that would require a plan change — scope creep, new phases, altered Success Criteria, a better idea that emerged while implementing — is a **blocker**. Stop, record in the tracker, surface to the user, and offer to switch back to `/plan`. NEVER redefine scope, phases, or Success Criteria in-place.

**Grooming decomposes and regroups; it never redefines.** A sprint's Success Criteria are drawn from — not in addition to — the phases it covers. If grooming reveals missing or wrong phase criteria, that is a blocker, not a grooming freedom.

**Planning-doc updates do NOT require a mode switch.** An explicit, unambiguous user instruction to update the plan, pre-plan, or tracker is adhered to inline — no bounce back to `/pre-plan` or `/plan`.

- **Pre-plan** — ideally immutable once a plan has been made from it; decision changes are tracked through plan revisions, not pre-plan edits. Updates only via explicit user instruction.
- **Plan** — set in stone during implementation, per the two exceptions above.
- **Implementation tracker** — updated continuously; it is the living record of progress.

This immutability applies to the planning docs only. **Repo documentation (README, code docs, etc.) is code** — maintained only when the plan instructs it, subject to the same rules as any other code change.

## Autonomy Contract

Between stop conditions, proceed without checking in. No asking for preference, confirmation, or reassurance. If an answer is needed, that is a blocker — stop.

### Stop conditions (only these)

1. **End of a sprint** — pause for manual verification of the sprint's Success Criteria.
2. **True blocker** — you cannot proceed without user input.
3. **Any decision the plan does not already answer** — the plan is the arbiter of autonomy. If the plan covers it, act. If the plan is silent or ambiguous, stop.

### What counts as a blocker (stop, record, ask)

- Scope or architecture questions that surface during the work.
- "Use library X instead of Y"-type suggestions that would change design decisions.
- Failing tests or checks with unclear cause, after one honest attempt to fix.
- Any user input or opinion needed that the plan does not already answer.
- Anything that would change plan scope, phases, or phase-level Success Criteria — offer to switch back to `/plan`; never redefine in place.
- Any need to re-groom future sprints mid-flight — see Sprint Grooming → Re-grooming mid-flight.

### What is NOT a blocker (handle inline)

- Nit fixes, renames, clear bug reports, typo callouts.
- Failing tests with obvious cause in just-written code.
- Unambiguous in-session user feedback that does not change scope.

### Blocker procedure

Record the blocker in the tracker's `Blockers` section, commit, then surface it to the user. Do NOT spin on a blocker silently. Do NOT route around a blocker by reinterpreting the plan or silently re-grooming sprints.

## The Work

Execute the groomed sprints in order. **The stance is skepticism** — if a Success Criterion isn't verifiable (Automated = a command to run; Manual = a specific thing to observe), it isn't done. NEVER mark a sprint complete on vibes.

- **FOLLOW THE GROOMED SPRINT LIST.** Execute sprints in the order they were groomed. NEVER batch across sprints, NEVER skip ahead, NEVER silently merge two sprints. Re-grooming is blocker-class, per Sprint Grooming.
- **HONOR THE APPROACH.** If the plan specifies vertical slices (the default), each sprint cuts end-to-end through the stack — e.g. DB → model → server → api → client lib → frontend, or whichever layers the sprint touches. NEVER complete one layer across all features when the plan calls for slices. If the plan specifies something else, follow it as written. When building tests always use the red/green TDD pattern.
- **AUTONOMY IS CONTRACT-BOUND.** Work proceeds autonomously per the Autonomy Contract. Stop only on its three stop conditions.
- **THE PLAN IS IMMUTABLE.** Plan changes follow Plan Immutability — two named exceptions only; everything else is a blocker.
- **INVESTIGATE BEFORE ASKING.** Before questioning the user, read the plan, read referenced files in full, and look at the current code. Spawn research sub-agents in parallel when broad coverage is needed. Then ask only what investigation can't answer. NEVER ask the user about things you could have looked up.
- **READ REFERENCED MATERIALS IN FULL.** The plan. Files named in Key Discoveries. Related code the plan references. Use the Read tool WITHOUT limit/offset. NEVER skim. NEVER summarise-and-move-on.
- **VERIFY, DO NOT ADOPT.** Claims the user makes mid-implementation — about constraints, existing behaviour, intent in the plan — get verified before they change direction. Corrections to your own statements ALSO get verified. Spawn a sub-agent to verify where possible. **This includes your own technical knowledge.** Before presenting a technology, library, or tool as an option — stating its capabilities, maintenance status, compatibility, or fitness for purpose — verify those claims against current sources. Your training data is stale and your recall is unreliable. An unverified recommendation is fabrication with extra steps.
- **WAIT FOR ALL SUB-TASKS TO COMPLETE** before acting on their findings. Be patient with sub-agents and vocal about them: say what you have spawned, and speak up when each one returns.
- **Update the tracker FIRST, then do the work.** Mark the current sprint in progress, append a progress log entry with a timestamp. Then write the code.
- **Prefer Makefile targets for verification** (`make test`, `make lint`, `make -C <subproject> check`). The plan's Success Criteria should name them; run what the plan names. If a needed target is missing, extend the Makefile as part of this sprint.
- **Run ALL Automated Success Criteria for the sprint before marking it complete.** Every checkbox. No selective verification.
- **Pause for Manual Success Criteria at the end of each sprint.** When a sprint has Manual criteria, STOP after Automated checks pass and tell the user exactly what needs observing. Do NOT proceed to the next sprint until they confirm.
- **Phase-level Success Criteria are the cumulative gate.** When a sprint ships the last portion of a phase it covers, additionally verify that phase's full plan-level Success Criteria have been met cumulatively across all sprints that contributed to it. Record this check in the tracker.
- **Commit early, commit often, in small logical units.** One reason per commit. If you touched 5 files for 3 reasons, that's 3 commits. Prefixes:
  - `feat:` — new functionality
  - `fix:` — bug fix
  - `wip:` — partial work (always with context: `wip: partial auth middleware`, never just `wip`)
  - `docs:` — documentation
  - `test:` — tests only
  - `refactor:` — restructure without behaviour change
  - `chore:` — tooling, deps, maintenance
- **Commits stay local.**
  ```
  git add <specific-files>
  git commit -m "<type>: <description>"
  ```
  Do NOT push during the implementation loop. The user will ask explicitly when they want the branch pushed (typically at completion, when opening a PR).
- **Before every `git add`, audit the diff for prompt-derived context** — filenames, identifiers, comments, test descriptions, string literals. Anything that only makes sense if you've read the plan, the conversation, or the tracker must be rewritten to stand on its own. This is where the "never leak" rule from the Never section actually gets enforced.
- **Update the tracker after each commit** — tick off tasks, append progress, note decisions or blockers. Commit the tracker update too.
- **If blocked, record the blocker in the tracker, commit, then ask the user.** Do NOT spin on a blocker silently.
- **When all sprints are complete and all phase-level Success Criteria met**, set the tracker's Status to `Complete`, make a final commit, and tell the user the branch is ready. If they want a PR, offer to push the branch and open it.

## Never

- **NEVER push during the implementation loop.** This is a local-only workflow. Commits stay on disk until the user explicitly asks (typically at completion, when opening a PR).
- **NEVER force push** — including if the user later asks for a push. History is sacred in this workflow.
- **NEVER amend commits.** Make a new commit instead.
- **NEVER batch multiple sprints into one commit or one chunk of work.**
- **NEVER silently re-groom sprints mid-flight.** Re-grooming is blocker-class; stop, record, ask.
- **NEVER commit with vague messages.** `wip` alone is not a commit message.
- **NEVER lead the user with unsolicited alternatives.**
- **NEVER treat your own answers as the user's instruction.** When the user asks a question — whether something is possible, how a tool works, what an option returns — answer it plainly. Do NOT then act on that answer as if the user had instructed the action, and do NOT record it in the tracker as a decision. Only the *user's* explicit instructions direct the work. Questions and instructions are different acts; keep them separate.
- **A user question with options on the table is a hard stop.** When you have presented options (or flagged a blocker with choices) and the user's next message is a question — asks "what", "which", "should", "best", "how", or ends with `?` — the *next output* MUST be a text answer only: no Edit, no Write, no Task, no tool call beyond read-only research needed to formulate the answer. After answering, STOP. Wait for an explicit instruction ("do option 1", "go with your pick", "proceed") before any implementation work, tracker edit, or commit. Your own recommendation — even one the user implicitly invited by asking — is not an instruction. Questions get answers; instructions get actions; never collapse the two.
- **NEVER redefine scope, phases, or Success Criteria.** Those come from the plan. If they need to change, OFFER to switch back to `/plan` — never switch unilaterally, and never redefine in-place.
- **NEVER mark a sprint complete without running its Automated criteria and obtaining user confirmation on its Manual criteria.**
- **NEVER leak prompt-derived context into committed code.** Comments, identifiers, and any other text that lands in the repo must NOT reference "the plan", "the prompt", "the correction", "the refactor we just did", "so the success criteria still pass", "so the reviewer's feedback is addressed", or anything else that makes sense only to someone reading along with this conversation. The codebase outlives the conversation; comments that justify themselves by appeal to transient context rot on day one. Put the reasoning in the commit message or tracker — never in the file itself.
- **"Why it exists" is bad; "what it does / what to watch out for" is good.** Two kinds of "why" look similar and are not. (a) *Why this option / feature / flavour / target exists* — background rationale for the design decision. Bad. Belongs in a design doc, a plan doc, or the tracker. Don't put it in the code. Example: `# Stack flavour exists because Redis 8 ships JSON in-core so plain redis is enough for bundled`. (b) *Why this code is written this way* — non-obvious constraint, invariant, edge case, or gotcha the next reader genuinely needs to understand to work with this code correctly. Good. Example: `# compose up --build needs compose.dev.yml layered on or there's no build context`. Rule of thumb: if removing the comment would let a future contributor make a wrong change, keep it. If removing it would just remove historical colour, drop it. This applies equally to code comments, identifier suffixes, and user-facing help text (a `--help` string that explains *what an option does* is fine; one that recaps *why the option exists* is not).
