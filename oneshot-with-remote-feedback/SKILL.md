---
name: oneshot-with-remote-feedback
description: Execute a plan document into code as a single uninterrupted parcel, publishing every commit so the remote branch is the live monitoring channel during execution; the human reviews the finished result at the end. Use when the work is most reviewable as a whole — refactors, codemods, migrations, bulk transforms — and per-sprint review would be busywork rather than value. Same hard limits as the sprint-loop variant; just no sprint loop.
argument-hint: [plan-path-or-slug]
---

# Oneshot with Remote Feedback

Execute a plan into code in a single uninterrupted parcel, publishing every commit so the remote branch is the live monitoring channel. The plan defines **phases** (design units); the parcel executes them in plan-defined order with no pauses for human review until the end. Manual Success Criteria are batched and reviewed once, at the end. You are an implementer, not a designer.

This skill is for work where reviewing each piece on its own is more burden than value — refactors that span many files, codemods, dependency migrations, bulk transforms. The human's review is the **tip of the iceberg**: one look at the finished result. Below the waterline, every commit is still pushed, the tracker still leads the work, hard limits still hold.

## Flow at a glance

1. **Preflight** — 11-step election: plan, clean tree, branch, PR strategy, baseline target, deferred choices, labels, comment-trust scope. Results land in `Preflight Decisions`.
2. **The tracker** — create `docs/plans/plan_<slug>_implementation.md`, commit, push. Living document.
3. **Execute** — phase by phase in the plan's order. Per-commit rhythm: code → commit → push → feedback → tracker. Run each phase's Automated Success Criteria when its code is done. No human review until the end.
4. **End-of-parcel review** — run all Manual Success Criteria with the user; cumulative phase-level gate.
5. **Wrap up** — strategy-specific completion: strip `[In Flight]`, offer the final PR, or hand off the branch.

Procedural detail for Preflight, PR strategies, and the feedback loop lives in `references/`.

## Preflight

Eleven steps. Every outcome lands in `Preflight Decisions`. Resolve all eleven before Execute — later behaviour forks on these elections, and getting them right up front is what keeps the parcel autonomous.

See `references/preflight.md` for the full procedure on each step: commands, defaults, prompts, failure handling. The checklist here is the map.

1. **Locate the plan.** Use `$ARGUMENTS` if it points to one; otherwise glob `docs/plans/plan_*.md` and ask. No plan → STOP; this skill does not invent scope.
2. **Read the plan in full.** Read tool, no limit/offset. The plan is the arbiter of scope, phases, and Success Criteria.
3. **Clean working tree.** `git status`. Dirty → STOP.
4. **Branch check.** If on `main`/`master`, warn and ask before proceeding.
5. **Pull latest.** `git pull`.
6. **Confirm and create the branch.** Default `<type>/<slug>`; types: `feature` / `bugfix` / `spike` / `refactor` / `docs` / `chore`. Create and `git push -u`.
7. **Elect PR strategy.** Three options, default `at-end`: `from-start` (live monitoring), `at-end` (one-shot review at the end — the natural fit for this skill), `none` (user handles PR). No `per-sprint` — there are no sprints.
8. **Baseline verification audit.** Find the repo's test target, ask whether to run it against the base branch now.
9. **Surface plan-deferred choices.** Scan the plan for TBD markers; resolve each upfront so the parcel stays autonomous.
10. **Label setup** (`from-start` only). `gh label create --force in-flight` and `do-not-merge`. Failure is a blocker — labels are load-bearing for that strategy.
11. **Elect comment trust scope.** Detect repo visibility, propose a default, record the minimum. Runs for all strategies so the threshold is ready if a PR opens later.

PR creation (under `from-start`) happens immediately after this section, before the first code commit. There is no Sprint Refinement to delay it.

## The Doc

Tracker skeleton at `docs/plans/plan_<slug>_implementation.md`:

````markdown
# Implementation: <title from plan>

**Status:** In Progress
**Branch:** `<type>/<slug>`
**Plan:** `<path to plan_<slug>.md>`
**PR:** `<URL or "none yet" or "n/a (no-PR strategy)">`

## Preflight Decisions

- **PR strategy:** `<from-start | at-end | none>`
- **Comment trust minimum:** `<admin | maintain | write | triage | read | none>`
- **Baseline verification:** `<command run + result, or "no baseline target found">`
- **In-flight labels created:** `<yes | no | n/a>`
- **Plan-deferred decisions:**
  - `<item>`: `<resolution>`

## Phase Status

<!-- One line per phase from the plan: in progress / Automated SC pass / blocked. Updated as phases close. -->

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

## Execute

Execute the plan's phases in their defined order, one phase at a time. Within a phase, work to a per-commit rhythm. The stance is skepticism: a Success Criterion that isn't verifiable (Automated = a command to run; Manual = a specific thing to observe) isn't done. Don't mark a phase or the parcel complete on vibes.

You execute the plan; you don't extend it. Anything not in the plan is either a blocker or a return to `/plan`, never part of the work. This is the principle that shapes everything below: each rhythm step, each standing rule, each hard limit is a corollary of it — re-ordering phases, reassigning Success Criteria, "handled elsewhere" fixes, helpful-but-unplanned refactors are all the same failure mode wearing different costumes.

Between stop conditions, proceed autonomously — no preference polls, no reassurance check-ins. Push after every commit; that is how the remote branch becomes the live monitoring channel.

### Stop conditions

Only two for this skill (the per-sprint pause that exists in the sprint-loop variant is gone here by design):

1. **A true blocker.** See Blockers below.
2. **A decision the plan does not already answer.** The plan is the arbiter of autonomy: if the plan covers it, act; if the plan is silent or ambiguous, stop and treat it as a blocker.

End-of-parcel — running Manual Success Criteria with the user, performing the strategy's completion action — is not a stop condition; it is the planned terminus. There is no human review pause mid-parcel.

### Per-commit rhythm

For each commit, in this order:

1. **Pick a logical change.** One reason per commit. If you touched five files for three reasons, that's three commits. Prefixes:

    - `feat:` — new functionality
    - `fix:` — bug fix
    - `wip:` — partial work, always with context: `wip: partial auth middleware`, never bare `wip`
    - `docs:` — documentation
    - `test:` — tests only
    - `refactor:` — restructure without behaviour change
    - `chore:` — tooling, deps, maintenance

2. **Audit the diff before staging** — filenames, identifiers, comments, test descriptions, string literals. Anything that makes sense only to a reader of this conversation — references like "the plan", "the prompt", "the refactor we just did", "so the success criteria still pass" — gets rewritten to stand alone. The codebase outlives the conversation; why-it-was-done goes in the commit message, PR, or tracker, not in the file. Code comments warn future readers about non-obvious constraints; they don't recap history.

3. **Commit and push.**

    ```
    git add <specific-files>
    git commit -m "<type>: <description>"
    git push
    ```

    A local-only commit is invisible to reviewers — it defeats the premise. After the push, run the feedback loop if a PR exists (`references/feedback-loop.md`).

4. **Update the tracker** — append the commit's purpose to `Progress Log`, note any decisions in `Decisions & Notes`, and push the tracker update too.

### Per-phase rhythm

The plan's phases are the structural unit. Per phase:

1. **Open the phase** in the tracker — set its `Phase Status` line to `in progress`, append a `Progress Log` entry with a timestamp, commit and push the tracker before any code is written.

2. **Iterate the per-commit rhythm** until the phase's Changes Required are done.

3. **Run every Automated Success Criterion for the phase.** Every checkbox; no selective verification. Prefer Makefile targets (`make test`, `make lint`, `make -C <subproject> check`); the plan's SC should name them. Missing target → extend the Makefile as part of this phase, don't skip.

4. **Phase audit.** Re-read in full (no limit/offset) the plan section for this phase. Every bullet under Changes Required and Success Criteria is either (a) executed and verified this phase, or (b) behind a Blocker entry with user acknowledgment. "Handled elsewhere", "covered later", "harness doesn't exist yet" are not (b) without a Blocker. Any failing bullet → phase isn't done; resume or open the Blocker.

5. **Close the phase in the tracker** — set its `Phase Status` line to `Automated SC pass`, push the tracker. Move to the next phase. Manual SC are deferred to the end-of-parcel review; do not pause to ask the user.

### Standing rules during execution

- **Honour the plan's approach.** If the plan says vertical slices, each phase cuts end-to-end. Horizontal slicing (all DB, then all API, then all frontend) defers verification and hides integration risk. If the plan says otherwise, follow it. Tests: red/green TDD.
- **Investigate before asking.** Read the plan, read referenced files in full (skim-and-summarise is how context drifts), look at the current code. Spawn research sub-agents in parallel for broad coverage. Ask only what investigation can't answer; don't ask about things already written down.
- **Verify claims, don't adopt them.** User claims mid-execution — about constraints, existing behaviour, plan intent — get verified before they change direction. Corrections to your own statements too. This includes your own technical knowledge: before presenting a library, tool, or version as suitable, verify against current sources. Training data is stale; an unverified recommendation is fabrication with extra steps.
- **Wait for sub-tasks before acting.** Say what you've spawned; announce when each returns; don't act on partial findings.

### Blockers

A blocker stops the parcel. Record in `Blockers`, commit, push, surface to the user. Never spin silently; never route around by reinterpreting the plan or silently re-ordering phases.

**Counts as a blocker:**

- Scope or architecture questions the work or a reviewer surfaces.
- "Use library X instead of Y"–type suggestions that would change design decisions.
- Conflicts between plan instructions and reviewer instructions.
- Failing CI with unclear cause, after one honest attempt to fix.
- Any user input or opinion needed that the plan doesn't already answer.
- Anything that would change plan scope, phases, or phase-level Success Criteria — offer `/plan`; never redefine in place.

**Handle inline (not a blocker):**

- Nit fixes, renames, clear bug reports, typo callouts.
- Failing CI with obvious cause in just-written code.
- Reviewer feedback (human or agent) that is unambiguous and does not change scope.

### End-of-parcel review

When every phase has its `Phase Status` line at `Automated SC pass`, the parcel is ready for human review. Two things happen:

1. **Run all Manual Success Criteria with the user.** Phase by phase, in plan order, surface each Manual SC to the user with the specific thing to observe. Don't skip; don't summarise. The user confirms each. Failures here are a blocker — record, surface, resume.

2. **Phase-level cumulative gate.** For every phase, verify that phase's full plan-level Success Criteria have been met cumulatively. Record the audit in the tracker.

Once both pass: set the tracker's Status to `Complete`, make a final commit and push, then perform the strategy's completion action — see `references/pr-strategies.md → Final-completion behaviour`.

## Monitoring

Others watch progress via:

- **The PR** (when one exists): comments, reviews, checks, the `## Phase Status` block in the body, and `in-flight` / `do-not-merge` labels under `from-start`. Primary channel for `from-start`.
- **The branch**: `git log --oneline origin/<type>/<slug>` and `git diff main..origin/<type>/<slug>`.
- **The tracker** on the remote branch: `docs/plans/plan_<slug>_implementation.md`. `Preflight Decisions` shows how the parcel was configured; `Phase Status` shows where execution stands; `Last-seen Feedback State` shows what feedback has been processed.

## Hard limits

Rules a session cannot violate and still count as an instance of this workflow. Each has a *why*; the reasoning is what lets the rule generalise. Other rules in this skill are directional advice. The ones below are gates.

Several are corollaries of one principle: **you execute the plan; you don't extend it**. The plan is the source of authority for what belongs in this parcel; work that doesn't trace to it is fabrication dressed as diligence.

1. **Every commit is pushed, immediately.** No local-only commits, no batching pushes.
    - *Why:* the remote branch is the only place a reviewer sees progress. A local-only commit has no readers. "I'll push them all at the end" is how this workflow degrades into a solo session.

2. **Write the tracker before the work, not after summarising it.**
    - *Why:* the tracker on the remote branch is what reviewers see and what a context-reset agent (you, in 30 minutes, with no memory of this conversation) reads to recover state. Holding "what's currently happening" in conversation while the tracker stays stale means three things: the remote view is wrong, the recovery state is missing, and the next push surprises reviewers with code that has no announcement. The rhythm is: open the phase → write the tracker → commit and push the tracker → start the work; commit code → push → update the tracker → push again. Not: start the work → make a few commits → eventually summarise into the tracker. The tracker update *leads* the work; it doesn't trail it. If you've made three commits without a corresponding tracker entry, you've already failed — recover by writing the tracker now, not after the next commit.

3. **History is append-only. No force push, no amending pushed commits.**
    - *Why:* the monitoring channel rests on sha stability. Rewriting published history invalidates any `Addressed in <sha>` reply, breaks teammates' local branches, and erases the trail. Bad commit landed? Make a new commit that fixes it.

4. **Execute phases in the plan's defined order. Don't reorder, skip ahead, or merge phases.**
    - *Why:* the plan's phase order is part of the design; phases are sequenced because the dependencies between them flow that way. Re-ordering, skipping, or silently merging makes the plan a lie about what's been done. The autonomous parcel only works if the plan is the source of truth — when execution drifts from the plan's order, the user can't trust the plan as a record of what happened.

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
