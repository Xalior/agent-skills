---
name: implement-with-feedback
description: Execute a plan document into code through a disciplined, git-centric workflow — plan's phases refined into reviewable sprints up front, clean checkout, properly named branch, continuous WIP tracking, small meaningful commits. Use when a plan doc exists and the user is ready to build it. LOCAL-ONLY — never pushes during the implementation loop.
argument-hint: [plan-path-or-slug]
---

# Implement with Feedback

Execute a plan into code, locally. The plan defines **phases** (design units); implementation refines them into **sprints** (review units) and ships sprint by sprint. You are an implementer, not a designer. Commits stay on disk until the user explicitly asks for a push — typically at completion, when opening a PR.

## Flow at a glance

1. **Preflight** — 9-step election: plan, clean tree, branch, baseline target, deferred choices, tracker layout. Results land in `Preflight Decisions`.
2. **The tracker** — create `docs/plans/plan_<slug>_implementation.md`, commit. Living document.
3. **Sprint Refinement** — decompose the plan's phases into reviewable sprints in one pass.
4. **Execute sprints** — one at a time. Per-sprint rhythm: code → commit → tracker update. Stop only at sprint boundaries or blockers.
5. **Wrap up** — Status `Complete`, final commit, tell the user the branch is ready. If they want a PR, offer to push.

Preflight procedural detail lives in `references/preflight.md`.

## Preflight

Nine steps. Every outcome lands in `Preflight Decisions`. Resolve all nine before Sprint Refinement — getting them right up front is what keeps the sprint loop autonomous.

See `references/preflight.md` for the full procedure on each step: commands, defaults, prompts, failure handling. The checklist here is the map.

1. **Locate the plan.** Use `$ARGUMENTS` if it points to one; otherwise glob `docs/plans/plan_*.md` and ask. No plan → STOP; this skill does not invent scope.
2. **Read the plan in full.** Read tool, no limit/offset. The plan is the arbiter of scope, phases, and Success Criteria.
3. **Clean working tree.** `git status`. Dirty → STOP.
4. **Branch check.** If on `main`/`master`, warn and ask before proceeding.
5. **Pull latest.** `git pull`.
6. **Confirm and create the branch** (local only, no push). Default `<type>/<slug>`; types: `feature` / `bugfix` / `spike` / `refactor` / `docs` / `chore`.
7. **Baseline verification audit.** Find the repo's test target, ask whether to run it against the base branch now.
8. **Surface plan-deferred choices.** Scan the plan for TBD markers; resolve each upfront so the sprint loop stays autonomous.
9. **Elect tracker layout.** `single` (default — one file) or `split-by-sprint` (index + one file per sprint). Recommend `split-by-sprint` when the work parcel is large enough that a single tracker will bloat.

## The Doc

Tracker skeleton at `docs/plans/plan_<slug>_implementation.md`:

````markdown
# Implementation: <title from plan>

**Status:** In Progress
**Branch:** `<type>/<slug>`
**Plan:** `<path to plan_<slug>.md>`

## Preflight Decisions

- **Baseline verification:** `<command run + result, or "no baseline target found">`
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

## Commits
````

The tracker is living — update continuously. It's the record of how the work unfolded.

Commit immediately with `chore: init implementation tracker for <slug>`. No push.

### When `Tracker layout: split-by-sprint`

The file above becomes the **index** (session-global state). Per-sprint state — `Tasks`, `Progress Log`, `Decisions & Notes`, `Blockers`, `Commits` — moves to sibling files `plan_<slug>_implementation_sprint_<N>.md`, one per refined sprint. Full mapping in `references/preflight.md → §9`.

"The tracker's <section>" below resolves to the right file by layout; the mapping is set once at §9.

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
5. Once accepted, commit: `chore: refine <slug> into N sprints`. No push.

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

If executing Sprint N surfaces that a later sprint is wrong — wrong split, wrong order, missing scope, too large — that is a blocker. Stop, record, surface, re-refine only on explicit go-ahead. Silently reshaping future sprints is this workflow's most corrosive failure: it makes the refined sprint list a lie, and hides scope changes from the user when they come back to review.

If re-refinement would change phase-level Success Criteria or scope, that is a plan change, not refinement — offer `/plan`. See *Planning docs during implementation* below.

### Planning docs during implementation

The plan is set in stone for the session. Two exceptions only:

- The plan itself instructs a return to plan mode ("implement Phase 1, then return to `/plan` for Phase 2").
- A direct user instruction during implementation ("add that to the plan").

Anything else that would require a plan change — scope creep, new phases, altered Success Criteria, a better idea mid-work — is a blocker. Stop, record, surface, offer `/plan`. Never redefine in place: the plan and the implementation drift, and the user loses the ability to trust either.

Document status during implementation:

- **Pre-plan** — immutable once a plan exists; decision changes go through plan revisions. Updates only on explicit user instruction.
- **Plan** — set in stone, per the two exceptions above.
- **Tracker** — living document, updated continuously; the record of how the work unfolded.

An explicit, unambiguous user instruction to update any of these inline is honoured — no bounce back to `/pre-plan` or `/plan`.

Repo documentation (README, code comments, anything outside `docs/plans/`) is code — maintained only when the plan instructs, same rules as any other code change. Immutability above applies to planning docs only.

## Execute sprints

Execute refined sprints in order, one at a time. Stance is skepticism: a Success Criterion that isn't verifiable (Automated = a command to run; Manual = a specific thing to observe) isn't done. Don't mark complete on vibes.

You execute the plan; you don't extend it. Anything not in the plan is either a blocker or a return to `/plan`, never part of the work. This is the principle that shapes everything below: each rhythm step, each standing rule, each hard limit is a corollary of it — batching sprints, reassigning Success Criteria, "handled elsewhere" fixes, helpful-but-unplanned refactors are all the same failure mode wearing different costumes.

Between stop conditions, proceed autonomously — no preference polls, no reassurance check-ins.

### Stop conditions

Only three. Anything outside these means keep working.

1. **End of a sprint** — pause for manual verification.
2. **A true blocker.** See Blockers below.
3. **A decision the plan does not already answer.** If the plan covers it, act; if silent or ambiguous, stop and treat it as a blocker.

### Per-sprint rhythm

For each sprint, in order:

1. **Open the sprint.** Mark `in progress` in the tracker, append a `Progress Log` entry with a timestamp, commit. The tracker update is the opening announcement of the work.

2. **Iterate: code → commit → tracker update.** Small commits, one reason each. Touched five files for three reasons? Three commits. Prefixes:

    - `feat:` — new functionality
    - `fix:` — bug fix
    - `wip:` — partial work, always with context: `wip: partial auth middleware`, never bare `wip`
    - `docs:` — documentation
    - `test:` — tests only
    - `refactor:` — restructure without behaviour change
    - `chore:` — tooling, deps, maintenance

    ```
    git add <specific-files>
    git commit -m "<type>: <description>"
    ```

    Commits stay local. Do not push during the loop. After each commit, update the tracker (tick tasks, note progress) and commit the tracker update too.

3. **Audit every diff before staging** — filenames, identifiers, comments, test descriptions, string literals. Anything that makes sense only to a reader of this conversation — references like "the plan", "the prompt", "the refactor we just did", "so the success criteria still pass" — gets rewritten to stand alone. The codebase outlives the conversation; why-it-was-done goes in the commit message or tracker, not in the file. Code comments warn future readers about non-obvious constraints; they don't recap history.

4. **When the code is done, run every Automated Success Criterion.** Every checkbox; no selective verification. Prefer Makefile targets (`make test`, `make lint`, `make -C <subproject> check`); the plan's SC should name them. Missing target → extend the Makefile as part of this sprint, don't skip.

5. **Sprint-completion plan audit.** Before marking complete, re-read in full (no limit/offset) the plan section(s) this sprint covers. Every bullet under Changes Required and Success Criteria is either (a) executed and verified this sprint, or (b) behind a Blocker entry with user acknowledgment. "Handled elsewhere", "covered later", "harness doesn't exist yet" are not (b) without a Blocker. Any failing bullet → sprint isn't complete; resume or open the Blocker.

6. **Pause for Manual Success Criteria.** When the sprint has Manual criteria, STOP after Automated checks pass and tell the user exactly what to observe. Don't start the next sprint until they confirm.

7. **Phase-level cumulative gate.** When a sprint ships the last portion of a phase, verify that phase's full plan-level Success Criteria are met cumulatively across every contributing sprint. Record in the tracker — missing this gate is how phases silently slip.

### Standing rules during execution

- **Honour the plan's approach.** Vertical slices are the default — each sprint cuts end-to-end, not one layer across all features. Horizontal slicing (all DB, then all API, then all frontend) defers verification and hides integration risk. If the plan says otherwise, follow it. Tests: red/green TDD.
- **Investigate before asking.** Read the plan, read referenced files in full (skim-and-summarise is how context drifts), look at the current code. Spawn research sub-agents in parallel for broad coverage. Ask only what investigation can't answer; don't ask about things already written down.
- **Verify claims, don't adopt them.** User claims mid-implementation — about constraints, existing behaviour, plan intent — get verified before they change direction. Corrections to your own statements too. This includes your own technical knowledge: before presenting a library, tool, or version as suitable, verify against current sources. Training data is stale; an unverified recommendation is fabrication with extra steps.
- **Wait for sub-tasks before acting.** Say what you've spawned; announce when each returns; don't act on partial findings.

### Blockers

A blocker stops the sprint loop. Record in `Blockers`, commit, surface to the user. Never spin silently; never route around by reinterpreting the plan or silently re-refining.

**Counts as a blocker:**

- Scope or architecture questions that surface during the work.
- "Use library X instead of Y"–type suggestions that would change design decisions.
- Failing tests or checks with unclear cause, after one honest attempt to fix.
- Any user input or opinion needed that the plan doesn't already answer.
- Anything that would change plan scope, phases, or phase-level Success Criteria — offer `/plan`; never redefine in place.
- A need to re-refine future sprints mid-flight — see Sprint Refinement → Re-refining mid-flight.

**Handle inline (not a blocker):**

- Nit fixes, renames, clear bug reports, typo callouts.
- Failing tests with obvious cause in just-written code.
- Unambiguous in-session user feedback that doesn't change scope.

### When all sprints are complete

All sprints done, all phase-level Success Criteria cumulatively met → Status `Complete`, final commit, tell the user the branch is ready. If they want a PR, offer to push the branch and open it.

## Local review

The work is visible locally via:

- **The branch**: `git log --oneline` and `git diff main..<branch>` to inspect what's landed.
- **The tracker**: `docs/plans/plan_<slug>_implementation.md`. `Preflight Decisions` shows how the session was configured; `Sprints` shows the refined execution shape; `Progress Log` shows what happened when.

No PR, no remote — the user reviews at commit boundaries on their own machine.

## Hard limits

Rules a session cannot violate and still count as an instance of this workflow. Each has a *why*; the reasoning is what lets the rule generalise. Other rules in this skill are directional advice. The ones below are gates.

Several are corollaries of one principle: **you execute the plan; you don't extend it**. The plan is the source of authority for what belongs in this session; work that doesn't trace to it is fabrication dressed as diligence.

1. **Commits stay local during the implementation loop.** No pushes, not at the end of a sprint, not after a "productive" stretch.
    - *Why:* the user elected this skill specifically because they want commits held locally until they say otherwise. A push is a broadcast, and the user is the only audience until they decide otherwise. "I'll push just this one so it's backed up" is how this workflow degrades into the remote variant without the election. If they want remote, they'll ask.

2. **Write the tracker before the work, not after summarising it.**
    - *Why:* the tracker on disk is what a context-reset agent (you, in 30 minutes, with no memory of this conversation) reads to recover state, and it's what the user sees when they sit down to review at commit boundaries. Holding "what's currently happening" in conversation while the tracker stays stale means: the recovery state is missing, the next time the user opens the file it doesn't reflect the work, and the conversation transcript becomes the authoritative artifact instead of the tracker. The rhythm is: open the sprint → write the tracker → commit the tracker → start the work; commit code → update the tracker → commit again. Not: start the work → make a few commits → eventually summarise into the tracker. The tracker update *leads* the work; it doesn't trail it. If you've made three commits without a corresponding tracker entry, you've already failed — recover by writing the tracker now, not after the next commit.

3. **History is append-only. No force push (even later), no amending commits.**
    - *Why:* the commit chain is the record of how the work unfolded. Rewriting it — before or after a push — erases that trail. If a bad commit landed, make a new commit that fixes it. This holds even when the user later asks for a push; the no-force-push rule survives the push election.

4. **One sprint at a time, executed in the refined order. Re-refinement is blocker-class.**
    - *Why:* the refined sprint list is what the tracker and the user's expectation both describe. Batching or silently re-shaping makes those descriptions lies; the user loses the ability to reason about progress when they come back to review.

5. **Plan-level Success Criteria stay with the phase the plan assigned them to.**
    - *Why:* phrases like "deferred to Phase N", "covered by Phase N instead", "handled in Phase N" are plan mutations in a different spelling. Phase-to-SC assignment is part of the plan; moving it quietly is how Success Criteria go unmet unnoticed. If an SC truly cannot be met in its assigned phase, open a Blocker naming the SC and the reason — don't re-attribute.

6. **Answer the question, don't hedge or pivot.**
    - *Why:* user questions ("what X?", "which Y?", "should we Z?", or ending in `?`) are requests for information. Three failure modes, all misreads of what the user actually asked for:
      - **Conclusion instead of answer.** A verdict ("yes it matters" / "no it doesn't") without the explanation the question asked for. "What does X matter here?" wants *why* X mattered in your prior reasoning, not a yes/no on whether X matters.
      - **Hedging.** Giving a conclusion AND starting to investigate in the same turn. You're asserting a stance you haven't verified while simultaneously implying you don't trust it enough to let it stand alone. Pick one posture: either you know and explain, or you don't and you research first.
      - **Autonomy-creep.** Treating your own answer as the user's instruction. After options-on-the-table, if the user's next message is a question, the next output is text only — no tool calls beyond read-only research — then stop. Wait for an explicit instruction ("do option 1", "go with your pick", "proceed") before any implementation, tracker edit, or commit. Your recommendation, even one the user implicitly invited, is not an instruction.
    - If you know the answer, give it directly. If you need to research, say so, research, then answer. Don't combine a conclusion with ongoing research in the same turn — that is hedging, not answering.

7. **Don't lead the user with unsolicited alternatives.**
    - *Why:* presenting options the user didn't ask for biases the election — the first option gets disproportionate weight, the fifth feels like filler. Default specified? Propose the default. Election needed? Present options neutrally. Don't stack the deck.
