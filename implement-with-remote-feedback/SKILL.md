---
name: implement-with-remote-feedback
description: Execute a plan document into code through a disciplined, git-centric workflow — plan's phases refined into reviewable sprints up front, clean checkout, properly named branch, continuous WIP tracking, small meaningful commits, and a push after every commit so the remote branch is the monitoring channel. Use when a plan doc exists, the user is ready to build it, and others need live visibility, with optional in-flight PR, per-sprint PR, or PR-at-end strategies elected up front.
argument-hint: [plan-path-or-slug]
---

# Implement with Remote Feedback

Execute a plan document into code, publishing every commit so the remote branch serves as the primary monitoring channel. The plan defines **phases** (design units); implementation refines them into **sprints** (reviewable execution units) and ships sprint by sprint. You are an implementer, not a designer.

## Flow at a glance

1. **Preflight** — 12-step election: locate plan, clean tree, branch name, PR strategy, baseline test target, deferred choices, labels, comment-trust scope, tracker layout. Results recorded in the tracker's `Preflight Decisions` block.
2. **The tracker** — create `docs/plans/plan_<slug>_implementation.md`, commit, push. Living document for the rest of the session.
3. **Sprint Refinement** — decompose the plan's phases into reviewable sprints in one pass; if strategy is `from-start`, open the PR now.
4. **Execute sprints** — per-sprint rhythm of code → commit → push → feedback → tracker update, one sprint at a time. Stop only at sprint boundaries or blockers.
5. **Wrap up** — strategy-specific completion action: strip `[In Flight]`, confirm merges, offer final PR, or hand off the branch.

Each phase has a section below; procedural detail for Preflight, PR strategies, and the feedback loop is in `references/`.

## Preflight

Preflight is a twelve-step questionnaire. Every step's outcome lands in the tracker's `Preflight Decisions` block. Do not begin Sprint Refinement until every step is resolved — later behaviour forks on these elections, so getting them right up front is what keeps the sprint loop autonomous.

See `references/preflight.md` for the full procedure on each step (exact commands, defaults, prompts to present, failure handling). The checklist here is the map; the reference is the detail.

1. **Locate the plan.** Use `$ARGUMENTS` if it points to one; otherwise glob `docs/plans/plan_*.md` and ask. No plan → STOP; this skill does not invent scope.
2. **Read the plan in full.** Read tool, no limit/offset. The plan is the arbiter of scope, phases, and Success Criteria for the rest of the session.
3. **Clean working tree.** `git status`. Dirty → STOP.
4. **Branch check.** If on `main`/`master`, warn and ask before proceeding.
5. **Pull latest.** `git pull`.
6. **Confirm and create the branch.** Default `<type>/<slug>`; types: `feature` / `bugfix` / `spike` / `refactor` / `docs` / `chore`. Create and `git push -u`.
7. **Elect PR strategy.** Four options, no default: `from-start`, `at-end`, `per-sprint`, `none`. The user must pick.
8. **Baseline verification audit.** Find the repo's test target, ask whether to run it against the base branch now.
9. **Surface plan-deferred choices.** Scan the plan for TBD markers; resolve each with the user upfront so the sprint loop stays autonomous.
10. **Label setup** (from-start only). `gh label create --force in-flight` and `do-not-merge`. Failure is a blocker — labels are load-bearing for this strategy.
11. **Elect comment trust scope.** Detect repo visibility, propose a default, record the minimum. Runs even for no-PR strategies so the threshold is ready if a PR is opened later.
12. **Elect tracker layout.** `single` (default — one file) or `split-by-sprint` (index file + one file per sprint). Recommend `split-by-sprint` when the work parcel is large enough that a single tracker will bloat.

PR creation happens after Sprint Refinement, not here — the PR body needs the refined sprints, not the plan's raw phases.

## The Doc

Create the implementation tracker at `docs/plans/plan_<slug>_implementation.md` with this skeleton:

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

The tracker is a living document — update it continuously. It tells the story of the implementation to anyone reading the git log. The file on disk plus the remote branch are the persistence layer.

Commit the tracker immediately with a message like `chore: init implementation tracker for <slug>` — then push.

### When `Tracker layout: split-by-sprint`

The file above becomes the **index** (session-global state only). Per-sprint state (`Tasks`, `Progress Log`, `Decisions & Notes`, `Blockers`, `Commits`) moves to sibling files named `plan_<slug>_implementation_sprint_<N>.md`, one per refined sprint. See `references/preflight.md → §12` for the full section-to-file mapping.

Throughout the rest of this skill, "the tracker's <section>" resolves to the right file by layout — the mapping is set once at Preflight step 12 and need not be restated at every reference.

## Sprint Refinement

After the tracker exists and before any code is touched, refine the plan's phases into **sprints**. A sprint is a manageably-reviewable, shippable unit of work — small enough that a reviewer can hold the whole change in their head in one sitting, large enough to be meaningful on its own. The review boundary drives sprint shape, not the plan's phase boundary.

### Sprints and phases are orthogonal

Refinement maps one onto the other three ways:

- One whole phase → one sprint. The simple case.
- Several small phases → one sprint. E.g. a "bootstrap" sprint covering repo-init + deps + CI + linter-config phases together.
- One large phase → several sprints. E.g. an auth rewrite refined into read-path sprint, write-path sprint, migration sprint.

**Refinement decomposes and regroups; it never redefines.** A sprint's Success Criteria are drawn from — not added to — the phases it covers. If refinement reveals a missing or wrong phase criterion, that is a blocker, not a refinement freedom. The plan is the arbiter; refinement is a structural projection of it onto review units.

### Procedure

Propose the full sprint breakdown in one pass. Don't round-robin sprint-by-sprint — that loses the whole-picture view that makes good refinement possible, and it slows the user down.

1. Write the proposal into the tracker's `Sprints` section (the index, when layout is `split-by-sprint`). Per sprint:
   - **Name** — short, descriptive.
   - **Covers** — which phase(s) by name, and what portion of each (`full`, or `Changes Required items X and Y`, or `read-path only`).
   - **Success Criteria** — Automated and Manual, drawn from the covered phases. When a sprint covers only part of a phase, include only the portion it owns.
   - **Status** — `not started`.

   When layout is `split-by-sprint`, also create `docs/plans/plan_<slug>_implementation_sprint_<N>.md` for each proposed sprint — skeleton only (sprint header + empty `Tasks`, `Progress Log`, `Decisions & Notes`, `Blockers`, `Commits`). Link each from the index's `Sprints` overview. This gets the shape on the remote branch in one commit; bodies fill in as sprints run.

2. Populate the tracker's `Tasks` checklist with one entry per proposed sprint (`- [ ] Sprint 1: <name>`). Under `split-by-sprint`, this overview lives on the index; the per-sprint Tasks checklist goes into each sprint file.
3. Default to **vertical slices**: each sprint is a thin end-to-end cut through the layers it touches, unless the plan's Approach section says otherwise. Vertical slices ship value and can be verified independently; horizontal slices (all DB, then all API, then all frontend) defer verification and hide integration risk until late.
4. Show the proposal to the user and ask them to accept or nudge. Log their response in `Progress Log` with a timestamp.
5. Once accepted, commit the tracker: `chore: refine <slug> into N sprints`.
6. **If PR strategy is `from-start`**, create the PR now — procedure in `references/pr-strategies.md`. Skip for any other strategy; those strategies create PRs later or not at all.

### Sizing for manageable review

Err small. A sprint that would take a reviewer more than an hour to review carefully is probably two sprints.

Signals a sprint is too large:

- More than ~5 distinct logical units of change.
- Touches unrelated subsystems that don't need to land together.
- Requires the reviewer to hold two separate mental models at once.

Signals a sprint is too small:

- Ships nothing end-to-end (and the plan's Approach doesn't call for a non-sliced shape).
- Can't be verified independently.
- Has no meaningful Success Criteria of its own.

### Re-refining mid-flight

If executing Sprint N surfaces that a later sprint is wrong — wrong split, wrong order, missing scope, too large — that is a blocker. Stop, record in the tracker, surface to the user, re-refine only with their explicit go-ahead. Silently reshaping future sprints is the single most corrosive failure mode for this workflow: it makes the refined sprint list a lie, breaks any PR body that references it, and hides scope changes from reviewers.

If re-refinement changes the sprint list on a `from-start` PR, update the PR body's `## Sprints` checklist to match after the user accepts.

If re-refinement would change phase-level Success Criteria or scope, that is a plan change, not a refinement — offer to switch back to `/plan`. See *Planning docs during implementation* below.

### Planning docs during implementation

The plan is set in stone for the duration of this session. Two exceptions only:

- The plan itself instructs a return to plan mode ("implement Phase 1, then return to `/plan` for Phase 2").
- A direct user instruction during implementation ("add that to the plan").

Anything else that would require a plan change — scope creep, new phases, altered Success Criteria, a better idea that emerged mid-work — is a blocker. Stop, record, surface to the user, offer `/plan`. Never redefine scope, phases, or Success Criteria in place; doing so makes the plan and the implementation drift, and reviewers lose the ability to trust either as a record of intent.

Document status during implementation:

- **Pre-plan** — immutable once a plan has been made from it; decision changes are tracked through plan revisions, not pre-plan edits. Updates only via explicit user instruction.
- **Plan** — set in stone, per the two exceptions above.
- **Tracker** — living document, updated continuously; the record of how the work unfolded.

An explicit, unambiguous user instruction to update any of these inline is honoured — no bounce back to `/pre-plan` or `/plan`.

Repo documentation (README, code comments, docs/ files not in `docs/plans/`) is code — maintained only when the plan instructs it, subject to the same rules as any other code change. The immutability above applies to the planning docs only.

## Execute sprints

Execute the refined sprints in order, one at a time. The stance is skepticism — if a Success Criterion isn't verifiable (Automated = a command to run; Manual = a specific thing to observe), it isn't done. Don't mark a sprint complete on vibes.

Between the stop conditions below, proceed autonomously — no asking for preference or reassurance, no check-ins for comfort. Push after every commit; that is how the remote branch becomes the monitoring channel for anyone watching.

### Stop conditions

Only three. Anything outside these means keep working.

1. **End of a sprint** — pause for manual verification of that sprint's Success Criteria.
2. **A true blocker.** See Blockers below.
3. **A decision the plan does not already answer.** The plan is the arbiter of autonomy: if the plan covers it, act; if the plan is silent or ambiguous, stop and treat it as a blocker.

### Per-sprint rhythm

For each sprint, in order:

1. **Open the sprint.** Mark it `in progress` in the tracker, append a `Progress Log` entry with a timestamp, commit and push. The tracker update announces the work before any code is written — reviewers seeing the remote branch know what's starting.

2. **Iterate: code → commit → push → feedback → tracker update.** Small commits, one reason each. If you touched five files for three reasons, that's three commits. Prefixes:

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

    A local-only commit is invisible to reviewers — it defeats the skill's premise. After the push, run the feedback loop if a PR exists — see `references/feedback-loop.md`. Then update the tracker (tick tasks, note progress) and push again.

3. **Audit every diff before staging** — filenames, identifiers, comments, test descriptions, string literals. Anything that only makes sense to someone who has been reading along with this conversation gets rewritten to stand on its own. The codebase outlives the conversation; why-it-was-done belongs in the commit message, PR description, or tracker, not in the file.

4. **When the sprint's code is done, run every Automated Success Criterion.** Every checkbox — no selective verification. Prefer Makefile targets (`make test`, `make lint`, `make -C <subproject> check`); the plan's Success Criteria should name them. If a needed target is missing, extend the Makefile as part of this sprint rather than skipping.

5. **Sprint-completion plan audit.** Before marking the sprint complete, re-read in full (Read tool, no limit/offset) the plan section(s) this sprint covers. For every bullet under Changes Required and Success Criteria, confirm either (a) it was executed and verified this sprint, or (b) a Blocker entry exists in the tracker with user acknowledgment explaining why it is unmet. "Handled elsewhere", "naturally covered by later work", "the harness doesn't exist yet" are not (b) unless a Blocker was opened. Any failing bullet means the sprint isn't complete — resume work or open the Blocker.

6. **End-of-sprint PR action.** Depends on the elected strategy — see `references/pr-strategies.md → End-of-sprint behaviour`.

7. **Pause for Manual Success Criteria.** When the sprint has Manual criteria, STOP after Automated checks pass and tell the user exactly what to observe. Don't start the next sprint until they confirm.

8. **Phase-level cumulative gate.** When a sprint ships the last portion of a phase it covers, verify that phase's full plan-level Success Criteria have been met cumulatively across every sprint that contributed. Record the check in the tracker — this is the moment a phase closes, and missing it means a phase silently slips.

### Standing rules during execution

- **Honour the plan's approach.** Vertical slices are the default — each sprint cuts end-to-end, not one layer across all features. Horizontal slicing (all DB, then all API, then all frontend) defers verification and hides integration risk until late. If the plan says otherwise, follow what it says. When writing tests, use red/green TDD.
- **Investigate before asking.** Read the plan, read referenced files in full (no limit/offset — skim-and-summarise is how context drifts), look at the current code. Spawn research sub-agents in parallel when coverage is broad. Only ask what investigation can't answer; don't ask the user about things that are already written down.
- **Verify claims, don't adopt them.** Claims the user makes mid-implementation — about constraints, existing behaviour, the plan's intent — get verified before they change direction. Corrections to your own earlier statements also get verified. This includes your own technical knowledge: before presenting a library, tool, or version as suitable, verify against current sources. Training data is stale; an unverified recommendation is fabrication with extra steps.
- **Wait for sub-tasks before acting.** Say what you've spawned; announce when each returns; don't act on partial findings.

### Blockers

A blocker stops the sprint loop. Procedure: record in the tracker's `Blockers` section, commit, push, then surface to the user. Do not spin on a blocker silently; do not route around a blocker by reinterpreting the plan or silently re-refining sprints.

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

All sprints done and all phase-level Success Criteria cumulatively met → set the tracker's Status to `Complete`, make a final commit and push, then perform the strategy's completion action — see `references/pr-strategies.md → Final-completion behaviour`.

## Monitoring

Others can watch progress via:

- **The PR** (when one exists): comments, reviews, checks, the `## Sprints` checklist in the body, and the `in-flight` / `do-not-merge` labels when the strategy is `from-start`. For `from-start` and `per-sprint`, the PR is the primary channel.
- **The branch**: `git log --oneline origin/<type>/<slug>` and `git diff main..origin/<type>/<slug>`.
- **The tracker** on the remote branch: `docs/plans/plan_<slug>_implementation.md`. `Preflight Decisions` tells a cold reader how the session was configured; `Sprints` shows the refined execution shape; `Last-seen Feedback State` tells them what feedback has been processed.

## Hard limits

These are the rules that a model cannot violate and still produce a session consistent with this workflow. Each has a *why* because the reasoning is what lets the rule generalise to cases not explicitly spelled out.

Other rules in this skill are directional — advice about how to work well. The ones below are gates: cross them and the work is no longer an instance of this skill.

1. **Every commit is pushed, immediately.** No local-only commits, no batching pushes.
    - *Why:* the remote branch is the only place a reviewer can see progress. A local-only commit has no readers; it defeats the skill's premise. "I'll push them all at the end" is the single most common way this workflow degrades into a solo session.

2. **History is append-only. No force push, no amending pushed commits.**
    - *Why:* the monitoring channel is built on sha stability. Rewriting published history invalidates any `Addressed in <sha>` reply on the PR, breaks teammates' local branches, and erases the trail of how the work unfolded. If a bad commit landed, make a new commit that fixes it.

3. **One sprint at a time, executed in the refined order. Re-refinement is blocker-class.**
    - *Why:* the refined sprint list is what the PR body, the tracker, and the user's expectation all describe. Batching sprints or silently re-shaping the order makes those descriptions lies, and reviewers lose the ability to reason about progress.

4. **Plan-level Success Criteria stay with the phase the plan assigned them to.**
    - *Why:* phrases like "deferred to Phase N", "covered by Phase N instead", "handled in Phase N" are plan mutations in a different spelling. The phase-to-SC assignment is part of the plan; moving it quietly is the most common way Success Criteria go unmet without anyone noticing. If an SC truly cannot be met in its assigned phase, open a Blocker that names the SC and the reason — don't re-attribute it.

5. **Answering a question is not the same as being instructed to act.**
    - *Why:* when the user asks "what should I...", "which of these...", "should we...", or ends a message with `?`, that is a request for information. Collapsing question into instruction is a common autonomy-creep failure: the agent answers its own question, acts on its own answer, and records the action as if the user approved. If the user's next message after options-on-the-table is a question, the next output must be a text answer only — no tool calls beyond read-only research — and then stop. Wait for an explicit instruction ("do option 1", "go with your pick", "proceed") before any implementation work, tracker edit, or commit. Your own recommendation, even one the user implicitly invited, is not an instruction.

6. **Don't leak conversation context into committed code.**
    - *Why:* comments, identifiers, or string literals that reference "the plan", "the prompt", "the refactor we just did", "so the success criteria still pass", or anything else that makes sense only to someone reading along with this conversation rot on day one. The codebase outlives the conversation. Reasoning about *why the change was made* belongs in the commit message, PR description, or tracker — never in the file. (Code comments exist to warn a future reader about a non-obvious constraint; they do not exist to recap history.)

7. **Don't lead the user with unsolicited alternatives.**
    - *Why:* presenting options the user didn't ask for biases the election — the first option you mention gets disproportionate weight, the fifth one feels like filler. When the plan or this skill specifies a default, propose the default. When an election is needed, present the options neutrally. Don't stack the deck with your own preference.
