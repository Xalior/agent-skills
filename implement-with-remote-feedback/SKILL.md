---
name: implement-with-remote-feedback
description: Execute a plan document into code through a disciplined, git-centric workflow — plan's phases refined into reviewable sprints up front, clean checkout, properly named branch, continuous WIP tracking, small meaningful commits, and a push after every commit so the remote branch is the monitoring channel. Use when a plan doc exists, the user is ready to build it, and others need live visibility, with optional in-flight PR, per-sprint PR, or PR-at-end strategies elected up front.
argument-hint: [plan-path-or-slug]
---

# Implement with Remote Feedback

Execute a plan into code, publishing every commit so the remote branch is the monitoring channel. The plan defines **phases** (design units); implementation refines them into **sprints** (review units) and ships sprint by sprint. You are an implementer, not a designer.

## Flow at a glance

1. **Preflight** — 12-step election: plan, clean tree, branch, PR strategy, baseline target, deferred choices, labels, comment-trust scope, tracker layout. Results land in `Preflight Decisions`.
2. **The tracker** — create `docs/plans/plan_<slug>_implementation.md`, commit, push. Living document.
3. **Sprint Refinement** — decompose the plan's phases into reviewable sprints in one pass. If strategy is `from-start`, open the PR now.
4. **Execute sprints** — one at a time. Per-sprint rhythm: code → commit → push → feedback → tracker. Stop only at sprint boundaries or blockers.
5. **Wrap up** — strategy-specific completion: strip `[In Flight]`, confirm merges, offer the final PR, or hand off the branch.

Each phase has a section below; procedural detail for Preflight, PR strategies, and the feedback loop lives in `references/`.

## Preflight

Twelve steps. Every outcome lands in `Preflight Decisions`. Resolve all twelve before Sprint Refinement — later behaviour forks on these elections, and getting them right up front is what keeps the sprint loop autonomous.

See `references/preflight.md` for the full procedure on each step: commands, defaults, prompts, failure handling. The checklist here is the map.

1. **Locate the plan.** Use `$ARGUMENTS` if it points to one; otherwise glob `docs/plans/plan_*.md` and ask. No plan → STOP; this skill does not invent scope.
2. **Read the plan in full.** Read tool, no limit/offset. The plan is the arbiter of scope, phases, and Success Criteria.
3. **Clean working tree.** `git status`. Dirty → STOP.
4. **Branch check.** If on `main`/`master`, warn and ask before proceeding.
5. **Pull latest.** `git pull`.
6. **Confirm and create the branch.** Default `<type>/<slug>`; types: `feature` / `bugfix` / `spike` / `refactor` / `docs` / `chore`. Create and `git push -u`.
7. **Elect PR strategy.** Four options, no default: `from-start`, `at-end`, `per-sprint`, `none`. The user must pick.
8. **Baseline verification audit.** Find the repo's test target, ask whether to run it against the base branch now.
9. **Surface plan-deferred choices.** Scan the plan for TBD markers; resolve each upfront so the sprint loop stays autonomous.
10. **Label setup** (`from-start` only). `gh label create --force in-flight` and `do-not-merge`. Failure is a blocker — labels are load-bearing for this strategy.
11. **Elect comment trust scope.** Detect repo visibility, propose a default, record the minimum. Runs for all strategies so the threshold is ready if a PR opens later.
12. **Elect tracker layout.** `single` (default — one file) or `split-by-sprint` (index + one file per sprint). Recommend `split-by-sprint` when the work parcel is large enough that a single tracker will bloat.

PR creation happens after Sprint Refinement, not here — the PR body needs the refined sprints.

## The Doc

Tracker skeleton at `docs/plans/plan_<slug>_implementation.md`:

````markdown
# Implementation: <title from plan>

**Status:** In Progress
**Branch:** `<type>/<slug>`
**Plan:** `<path to plan_<slug>.md>`
**PR:** `<URL or "none yet" or "n/a (no-PR strategy)">`

## Preflight Decisions

- **PR strategy:** `<from-start | at-end | per-sprint | none>`
- **Comment trust minimum:** `<admin | maintain | write | triage | read | none>`
- **Baseline verification:** `<command run + result, or "no baseline target found">`
- **In-flight labels created:** `<yes | no | n/a>`
- **Tracker layout:** `<single | split-by-sprint>`
- **Plan-deferred decisions:**
  - `<item>`: `<resolution>`

## Sprints

<!-- Filled in during Sprint Refinement, one entry per sprint -->

## Tasks

<!-- Sprint-keyed progress checklist. Filled in during Sprint Refinement. -->

## Progress Log

## Decisions & Notes

## Blockers

## Last-seen Feedback State

- **Last-seen comment id:** `<id or "none yet">`
- **Last-seen review id:** `<id or "none yet">`
- **Last-seen check suite:** `<id or "none yet">`
- **Ignored (below trust threshold):**
  - `<comment URL>` — `<reason>`

## Commits
````

The tracker is living — update continuously. It tells the story to anyone reading the remote branch.

Commit immediately with `chore: init implementation tracker for <slug>`, then push.

### When `Tracker layout: split-by-sprint`

The file above becomes the **index** (session-global state). Per-sprint state — `Tasks`, `Progress Log`, `Decisions & Notes`, `Blockers`, `Commits` — moves to sibling files `plan_<slug>_implementation_sprint_<N>.md`, one per refined sprint. Full mapping in `references/preflight.md → §12`.

"The tracker's <section>" below resolves to the right file by layout; the mapping is set once at §12.

## Sprint Refinement

After the tracker exists and before any code, refine the plan's phases into **sprints**. A sprint is a manageably-reviewable, shippable unit — small enough to hold in one sitting, large enough to be meaningful on its own. The review boundary drives sprint shape, not the plan's phase boundary.

### Sprints and phases are orthogonal

Refinement maps one onto the other three ways:

- One whole phase → one sprint. The simple case.
- Several small phases → one sprint. E.g. a "bootstrap" sprint covering repo-init + deps + CI + linter-config.
- One large phase → several sprints. E.g. an auth rewrite refined into read-path, write-path, migration sprints.

**Refinement decomposes and regroups; it never redefines.** A sprint's Success Criteria are drawn from — not added to — the phases it covers. If refinement reveals a missing or wrong phase criterion, that is a blocker. The plan is the arbiter; refinement only reshapes it into review units.

### Procedure

Propose the full breakdown in one pass. Don't round-robin — it loses the whole-picture view that makes good refinement possible, and slows the user down.

1. Write the proposal into the tracker's `Sprints` section (the index, under `split-by-sprint`). Per sprint:
   - **Name** — short, descriptive.
   - **Covers** — which phase(s) by name, and what portion of each (`full`, or `Changes Required items X and Y`, or `read-path only`).
   - **Success Criteria** — Automated and Manual, drawn from the covered phases. Part of a phase? Include only that part.
   - **Status** — `not started`.

   Under `split-by-sprint`, also create `docs/plans/plan_<slug>_implementation_sprint_<N>.md` per sprint — skeleton only (header + empty `Tasks`, `Progress Log`, `Decisions & Notes`, `Blockers`, `Commits`). Link each from the index's `Sprints` overview. One commit carries the shape; bodies fill in as sprints run.

2. Populate `Tasks` with one entry per sprint (`- [ ] Sprint 1: <name>`). Under `split-by-sprint`, this overview is on the index; per-sprint Tasks go in the sprint files.
3. Default to **vertical slices**: each sprint a thin end-to-end cut, unless the plan's Approach says otherwise. Vertical slices ship value and verify independently; horizontal slices (all DB, then all API, then all frontend) defer verification and hide integration risk.
4. Show the proposal to the user; ask them to accept or nudge. Log their response in `Progress Log` with a timestamp.
5. Once accepted, commit: `chore: refine <slug> into N sprints`.
6. **If strategy is `from-start`**, create the PR now — see `references/pr-strategies.md`. Other strategies create PRs later or not at all; skip.

### Sizing for manageable review

Err small. A sprint that would take a reviewer more than an hour to review carefully is probably two sprints.

Too large if:

- More than ~5 distinct logical units of change.
- Touches unrelated subsystems that don't need to land together.
- Requires the reviewer to hold two separate mental models at once.

Too small if:

- Ships nothing end-to-end (and the plan's Approach doesn't call for a non-sliced shape).
- Can't be verified independently.
- Has no meaningful Success Criteria of its own.

### Re-refining mid-flight

If executing Sprint N surfaces that a later sprint is wrong — wrong split, wrong order, missing scope, too large — that is a blocker. Stop, record, surface, re-refine only on explicit go-ahead. Silently reshaping future sprints is this workflow's most corrosive failure: it makes the refined sprint list a lie, breaks any PR body that references it, and hides scope changes from reviewers.

If re-refinement changes the sprint list on a `from-start` PR, update the PR body's `## Sprints` checklist to match after the user accepts.

If re-refinement would change phase-level Success Criteria or scope, that is a plan change, not refinement — offer `/plan`. See *Planning docs during implementation* below.

### Planning docs during implementation

The plan is set in stone for the session. Two exceptions only:

- The plan itself instructs a return to plan mode ("implement Phase 1, then return to `/plan` for Phase 2").
- A direct user instruction during implementation ("add that to the plan").

Anything else that would require a plan change — scope creep, new phases, altered Success Criteria, a better idea mid-work — is a blocker. Stop, record, surface, offer `/plan`. Never redefine in place: the plan and the implementation drift, and reviewers lose the ability to trust either.

Document status during implementation:

- **Pre-plan** — immutable once a plan exists; decision changes go through plan revisions. Updates only on explicit user instruction.
- **Plan** — set in stone, per the two exceptions above.
- **Tracker** — living document, updated continuously; the record of how the work unfolded.

An explicit, unambiguous user instruction to update any of these inline is honoured — no bounce back to `/pre-plan` or `/plan`.

Repo documentation (README, code comments, anything outside `docs/plans/`) is code — maintained only when the plan instructs, same rules as any other code change. Immutability above applies to planning docs only.

## Execute sprints

Execute refined sprints in order, one at a time. Stance is skepticism: a Success Criterion that isn't verifiable (Automated = a command to run; Manual = a specific thing to observe) isn't done. Don't mark complete on vibes.

You execute the plan; you don't extend it. Anything not in the plan is either a blocker or a return to `/plan`, never part of the work. This is the principle that shapes everything below: each rhythm step, each standing rule, each hard limit is a corollary of it — batching sprints, reassigning Success Criteria, "handled elsewhere" fixes, helpful-but-unplanned refactors are all the same failure mode wearing different costumes.

Between stop conditions, proceed autonomously — no preference polls, no reassurance check-ins. Push after every commit; that is how the remote branch becomes the monitoring channel.

### Stop conditions

Only three. Anything outside these means keep working.

1. **End of a sprint** — pause for manual verification.
2. **A true blocker.** See Blockers below.
3. **A decision the plan does not already answer.** If the plan covers it, act; if silent or ambiguous, stop and treat it as a blocker.

### Per-sprint rhythm

For each sprint, in order:

1. **Open the sprint.** Mark `in progress` in the tracker, append a `Progress Log` entry with a timestamp, commit and push. The tracker update announces the work before any code is written.

2. **Iterate: code → commit → push → feedback → tracker update.** Small commits, one reason each. Touched five files for three reasons? Three commits. Prefixes:

    - `feat:` — new functionality
    - `fix:` — bug fix
    - `wip:` — partial work, always with context: `wip: partial auth middleware`, never bare `wip`
    - `docs:` — documentation
    - `test:` — tests only
    - `refactor:` — restructure without behaviour change
    - `chore:` — tooling, deps, maintenance

    After every commit:

    ```
    git push
    ```

    A local-only commit is invisible to reviewers — it defeats the premise. After the push, run the feedback loop if a PR exists (`references/feedback-loop.md`). Update the tracker (tick tasks, note progress), push again.

3. **Audit every diff before staging** — filenames, identifiers, comments, test descriptions, string literals. Anything that makes sense only to a reader of this conversation gets rewritten to stand alone. The codebase outlives the conversation; why-it-was-done goes in the commit message, PR, or tracker — not in the file.

4. **When the code is done, run every Automated Success Criterion.** Every checkbox; no selective verification. Prefer Makefile targets (`make test`, `make lint`, `make -C <subproject> check`); the plan's SC should name them. Missing target → extend the Makefile as part of this sprint, don't skip.

5. **Sprint-completion plan audit.** Before marking complete, re-read in full (no limit/offset) the plan section(s) this sprint covers. Every bullet under Changes Required and Success Criteria is either (a) executed and verified this sprint, or (b) behind a Blocker entry with user acknowledgment. "Handled elsewhere", "covered later", "harness doesn't exist yet" are not (b) without a Blocker. Any failing bullet → sprint isn't complete; resume or open the Blocker.

6. **End-of-sprint PR action.** Depends on the elected strategy — see `references/pr-strategies.md → End-of-sprint behaviour`.

7. **Pause for Manual Success Criteria.** When the sprint has Manual criteria, STOP after Automated checks pass and tell the user exactly what to observe. Don't start the next sprint until they confirm.

8. **Phase-level cumulative gate.** When a sprint ships the last portion of a phase, verify that phase's full plan-level Success Criteria are met cumulatively across every contributing sprint. Record in the tracker — missing this gate is how phases silently slip.

### Standing rules during execution

- **Honour the plan's approach.** Vertical slices are the default — each sprint cuts end-to-end, not one layer across all features. Horizontal slicing (all DB, then all API, then all frontend) defers verification and hides integration risk. If the plan says otherwise, follow it. Tests: red/green TDD.
- **Investigate before asking.** Read the plan, read referenced files in full (skim-and-summarise is how context drifts), look at the current code. Spawn research sub-agents in parallel for broad coverage. Ask only what investigation can't answer; don't ask about things already written down.
- **Verify claims, don't adopt them.** User claims mid-implementation — about constraints, existing behaviour, plan intent — get verified before they change direction. Corrections to your own statements too. This includes your own technical knowledge: before presenting a library, tool, or version as suitable, verify against current sources. Training data is stale; an unverified recommendation is fabrication with extra steps.
- **Wait for sub-tasks before acting.** Say what you've spawned; announce when each returns; don't act on partial findings.

### Blockers

A blocker stops the sprint loop. Record in `Blockers`, commit, push, surface to the user. Never spin silently; never route around by reinterpreting the plan or silently re-refining.

**Counts as a blocker:**

- Scope or architecture questions the work or a reviewer surfaces.
- "Use library X instead of Y"–type suggestions that would change design decisions.
- Conflicts between plan instructions and reviewer instructions.
- Failing CI with unclear cause, after one honest attempt to fix.
- Any user input or opinion needed that the plan doesn't already answer.
- Anything that would change plan scope, phases, or phase-level Success Criteria — offer `/plan`; never redefine in place.
- A need to re-refine future sprints mid-flight — see Sprint Refinement → Re-refining mid-flight.
- `gh label create --force` failing during Preflight.
- A `per-sprint` PR about to be opened while the prior sprint's PR is still unmerged.

**Handle inline (not a blocker):**

- Nit fixes, renames, clear bug reports, typo callouts.
- Failing CI with obvious cause in just-written code.
- Reviewer feedback (human or agent) that is unambiguous and does not change scope.

### When all sprints are complete

All sprints done, all phase-level Success Criteria cumulatively met → Status `Complete`, final commit and push, then the strategy's completion action (`references/pr-strategies.md → Final-completion behaviour`).

## Monitoring

Others watch progress via:

- **The PR** (when one exists): comments, reviews, checks, the `## Sprints` checklist, and `in-flight` / `do-not-merge` labels under `from-start`. Primary channel for `from-start` and `per-sprint`.
- **The branch**: `git log --oneline origin/<type>/<slug>` and `git diff main..origin/<type>/<slug>`.
- **The tracker** on the remote branch: `docs/plans/plan_<slug>_implementation.md`. `Preflight Decisions` shows how the session was configured; `Sprints` shows the refined execution shape; `Last-seen Feedback State` shows what feedback has been processed.

## Hard limits

Rules a session cannot violate and still count as an instance of this workflow. Each has a *why*; the reasoning is what lets the rule generalise. Other rules in this skill are directional advice. The ones below are gates.

Several are corollaries of one principle: **you execute the plan; you don't extend it**. The plan is the source of authority for what belongs in this session; work that doesn't trace to it is fabrication dressed as diligence.

1. **Every commit is pushed, immediately.** No local-only commits, no batching pushes.
    - *Why:* the remote branch is the only place a reviewer sees progress. A local-only commit has no readers. "I'll push them all at the end" is how this workflow degrades into a solo session.

2. **History is append-only. No force push, no amending pushed commits.**
    - *Why:* the monitoring channel rests on sha stability. Rewriting published history invalidates any `Addressed in <sha>` reply, breaks teammates' local branches, and erases the trail. Bad commit landed? Make a new commit that fixes it.

3. **One sprint at a time, executed in the refined order. Re-refinement is blocker-class.**
    - *Why:* the refined sprint list is what the PR body, tracker, and user expectation all describe. Batching or silently re-shaping makes those descriptions lies; reviewers lose the ability to reason about progress.

4. **Plan-level Success Criteria stay with the phase the plan assigned them to.**
    - *Why:* phrases like "deferred to Phase N", "covered by Phase N instead", "handled in Phase N" are plan mutations in a different spelling. Phase-to-SC assignment is part of the plan; moving it quietly is how Success Criteria go unmet unnoticed. If an SC truly cannot be met in its assigned phase, open a Blocker naming the SC and the reason — don't re-attribute.

5. **Answer the question, don't hedge or pivot.**
    - *Why:* user questions ("what X?", "which Y?", "should we Z?", or ending in `?`) are requests for information. Three failure modes, all misreads of what the user actually asked for:
      - **Conclusion instead of answer.** A verdict ("yes it matters" / "no it doesn't") without the explanation the question asked for. "What does X matter here?" wants *why* X mattered in your prior reasoning, not a yes/no on whether X matters.
      - **Hedging.** Giving a conclusion AND starting to investigate in the same turn. You're asserting a stance you haven't verified while simultaneously implying you don't trust it enough to let it stand alone. Pick one posture: either you know and explain, or you don't and you research first.
      - **Autonomy-creep.** Treating your own answer as the user's instruction. After options-on-the-table, if the user's next message is a question, the next output is text only — no tool calls beyond read-only research — then stop. Wait for an explicit instruction ("do option 1", "go with your pick", "proceed") before any implementation, tracker edit, or commit. Your recommendation, even one the user implicitly invited, is not an instruction.
    - If you know the answer, give it directly. If you need to research, say so, research, then answer. Don't combine a conclusion with ongoing research in the same turn — that is hedging, not answering.

6. **Don't leak conversation context into committed code.**
    - *Why:* comments, identifiers, or string literals that reference "the plan", "the prompt", "the refactor we just did", "so the success criteria still pass" — anything that makes sense only to a reader of this conversation — rot on day one. The codebase outlives the conversation. Why-it-was-done goes in the commit message, PR, or tracker; never in the file. (Code comments warn future readers about non-obvious constraints; they don't recap history.)

7. **Don't lead the user with unsolicited alternatives.**
    - *Why:* presenting options the user didn't ask for biases the election — the first option gets disproportionate weight, the fifth feels like filler. Default specified? Propose the default. Election needed? Present options neutrally. Don't stack the deck.
