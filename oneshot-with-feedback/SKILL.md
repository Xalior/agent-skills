---
name: oneshot-with-feedback
description: Execute a plan document into code as a single uninterrupted parcel, locally — no sprint loop, no per-phase manual review pauses, and no push during the loop. The user reviews the finished result at the end. Use when the work is most reviewable as a whole — refactors, codemods, migrations, bulk transforms — and per-sprint review would be busywork. LOCAL-ONLY — never pushes during execution.
argument-hint: [plan-path-or-slug]
---

# Oneshot with Feedback

Execute a plan into code in a single uninterrupted parcel, locally. The plan defines **phases** (design units); the parcel executes them in plan-defined order with no pauses for human review until the end. Manual Success Criteria are batched and reviewed once, at the end. You are an implementer, not a designer. Commits stay on disk until the user explicitly asks for a push — typically at completion, when opening a PR.

This skill is for work where reviewing each piece on its own is more burden than value — refactors that span many files, codemods, dependency migrations, bulk transforms — and where the user wants to keep commits local until they're ready to push. The human's review is the **tip of the iceberg**: one look at the finished result. Below the waterline, every commit is logical and small, the tracker still leads the work, hard limits still hold.

## Flow at a glance

1. **Preflight** — 8-step election: plan, clean tree, branch, baseline target, deferred choices.
2. **The tracker** — create `docs/plans/plan_<slug>_implementation.md`, commit. Living document.
3. **Execute** — phase by phase in the plan's order. Per-commit rhythm: code → commit → tracker. Run each phase's Automated Success Criteria when its code is done. No human review until the end.
4. **End-of-parcel review** — run all Manual Success Criteria with the user; cumulative phase-level gate.
5. **Wrap up** — Status `Complete`, final commit, tell the user the branch is ready. If they want a PR, offer to push.

Preflight procedural detail lives in `references/preflight.md`.

## Preflight

Eight steps. Every outcome lands in `Preflight Decisions`. Resolve all eight before Execute — getting them right up front is what keeps the parcel autonomous.

See `references/preflight.md` for the full procedure on each step: commands, defaults, prompts, failure handling. The checklist here is the map.

1. **Locate the plan.** Use `$ARGUMENTS` if it points to one; otherwise glob `docs/plans/plan_*.md` and ask. No plan → STOP; this skill does not invent scope.
2. **Read the plan in full.** Read tool, no limit/offset. The plan is the arbiter of scope, phases, and Success Criteria.
3. **Clean working tree.** `git status`. Dirty → STOP.
4. **Branch check.** If on `main`/`master`, warn and ask before proceeding.
5. **Pull latest.** `git pull`.
6. **Confirm and create the branch** (local only, no push). Default `<type>/<slug>`; types: `feature` / `bugfix` / `spike` / `refactor` / `docs` / `chore`.
7. **Baseline verification audit.** Find the repo's test target, ask whether to run it against the base branch now.
8. **Surface plan-deferred choices.** Scan the plan for TBD markers; resolve each upfront so the parcel stays autonomous.

## The Doc

Tracker skeleton at `docs/plans/plan_<slug>_implementation.md`:

````markdown
# Implementation: <title from plan>

**Status:** In Progress
**Branch:** `<type>/<slug>`
**Plan:** `<path to plan_<slug>.md>`

## Preflight Decisions

- **Baseline verification:** `<command run + result, or "no baseline target found">`
- **Plan-deferred decisions:**
  - `<item>`: `<resolution>`

## Phase Status

<!-- One line per phase from the plan: in progress / Automated SC pass / blocked. Updated as phases close. -->

## Progress Log

## Decisions & Notes

## Blockers

## Commits
````

The tracker is living — update continuously. It's the record of how the work unfolded.

Commit immediately with `chore: init implementation tracker for <slug>`. No push.

## Execute

Execute the plan's phases in their defined order, one phase at a time. Within a phase, work to a per-commit rhythm. The stance is skepticism: a Success Criterion that isn't verifiable (Automated = a command to run; Manual = a specific thing to observe) isn't done. Don't mark a phase or the parcel complete on vibes.

You execute the plan; you don't extend it. Anything not in the plan is either a blocker or a return to `/plan`, never part of the work. This is the principle that shapes everything below: each rhythm step, each standing rule, each hard limit is a corollary of it — re-ordering phases, reassigning Success Criteria, "handled elsewhere" fixes, helpful-but-unplanned refactors are all the same failure mode wearing different costumes.

Between stop conditions, proceed autonomously — no preference polls, no reassurance check-ins.

### Stop conditions

Only two for this skill (the per-sprint pause that exists in the sprint-loop variant is gone here by design):

1. **A true blocker.** See Blockers below.
2. **A decision the plan does not already answer.** The plan is the arbiter of autonomy: if the plan covers it, act; if the plan is silent or ambiguous, stop and treat it as a blocker.

End-of-parcel — running Manual Success Criteria with the user, performing the completion handoff — is not a stop condition; it is the planned terminus. There is no human review pause mid-parcel.

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

2. **Audit the diff before staging** — filenames, identifiers, comments, test descriptions, string literals. Anything that makes sense only to a reader of this conversation — references like "the plan", "the prompt", "the refactor we just did", "so the success criteria still pass" — gets rewritten to stand alone. The codebase outlives the conversation; why-it-was-done goes in the commit message or tracker, not in the file. Code comments warn future readers about non-obvious constraints; they don't recap history.

3. **Commit.**

    ```
    git add <specific-files>
    git commit -m "<type>: <description>"
    ```

    Commits stay local. Do not push during the loop.

4. **Update the tracker** — append the commit's purpose to `Progress Log`, note any decisions in `Decisions & Notes`. Commit the tracker update too.

### Per-phase rhythm

The plan's phases are the structural unit. Per phase:

1. **Open the phase** in the tracker — set its `Phase Status` line to `in progress`, append a `Progress Log` entry with a timestamp, commit the tracker before any code is written.

2. **Iterate the per-commit rhythm** until the phase's Changes Required are done.

3. **Run every Automated Success Criterion for the phase.** Every checkbox; no selective verification. Prefer Makefile targets (`make test`, `make lint`, `make -C <subproject> check`); the plan's SC should name them. Missing target → extend the Makefile as part of this phase, don't skip.

4. **Phase audit.** Re-read in full (no limit/offset) the plan section for this phase. Every bullet under Changes Required and Success Criteria is either (a) executed and verified this phase, or (b) behind a Blocker entry with user acknowledgment. "Handled elsewhere", "covered later", "harness doesn't exist yet" are not (b) without a Blocker. Any failing bullet → phase isn't done; resume or open the Blocker.

5. **Close the phase in the tracker** — set its `Phase Status` line to `Automated SC pass`, commit the tracker. Move to the next phase. Manual SC are deferred to the end-of-parcel review; do not pause to ask the user.

### Standing rules during execution

- **Honour the plan's approach.** If the plan says vertical slices, each phase cuts end-to-end. Horizontal slicing (all DB, then all API, then all frontend) defers verification and hides integration risk. If the plan says otherwise, follow it. Tests: red/green TDD.
- **Investigate first, via sub-agents by default.** Sub-agents are the default for research because they read in their own context window — your context stays clean for the plan, the work, and your own thinking. Use sub-agents for: exploring codebase areas, scanning long ancillary files, verifying technology claims against current sources, summarising prior tickets or docs. Direct reads (Read tool, no limit/offset) stay reserved for the plan and the files you're about to edit; skim-and-summarise on the spec is how context drifts, sub-agent on research is how context survives. Ask the user only what investigation can't answer; don't ask about things already written down.
- **Verify claims, don't adopt them.** User claims mid-execution — about constraints, existing behaviour, plan intent — get verified before they change direction. Corrections to your own statements too. This includes your own technical knowledge: before presenting a library, tool, or version as suitable, verify against current sources. Training data is stale; an unverified recommendation is fabrication with extra steps.
- **Wait for sub-tasks before acting.** Say what you've spawned; announce when each returns; don't act on partial findings.

### Blockers

A blocker stops the parcel. Record in `Blockers`, commit, surface to the user. Never spin silently; never route around by reinterpreting the plan or silently re-ordering phases.

**Counts as a blocker:**

- Scope or architecture questions that surface during the work.
- "Use library X instead of Y"–type suggestions that would change design decisions.
- Failing tests or checks with unclear cause, after one honest attempt to fix.
- Any user input or opinion needed that the plan doesn't already answer.
- Anything that would change plan scope, phases, or phase-level Success Criteria — offer `/plan`; never redefine in place.

**Handle inline (not a blocker):**

- Nit fixes, renames, clear bug reports, typo callouts.
- Failing tests with obvious cause in just-written code.
- Unambiguous in-session user feedback that doesn't change scope.

### End-of-parcel review

When every phase has its `Phase Status` line at `Automated SC pass`, the parcel is ready for human review. Two things happen:

1. **Run all Manual Success Criteria with the user.** Phase by phase, in plan order, surface each Manual SC to the user with the specific thing to observe. Don't skip; don't summarise. The user confirms each. Failures here are a blocker — record, surface, resume.

2. **Phase-level cumulative gate.** For every phase, verify that phase's full plan-level Success Criteria have been met cumulatively. Record the audit in the tracker.

Once both pass: set the tracker's Status to `Complete`, make a final commit, tell the user the branch is ready. If they want a PR, offer to push the branch and open it.

## Local review

The work is visible locally via:

- **The branch**: `git log --oneline` and `git diff main..<branch>` to inspect what's landed.
- **The tracker**: `docs/plans/plan_<slug>_implementation.md`. `Preflight Decisions` shows how the parcel was configured; `Phase Status` shows where execution stands; `Progress Log` shows what happened when.

No PR, no remote — the user reviews at the end of the parcel on their own machine.

## Hard limits

Rules a session cannot violate and still count as an instance of this workflow. Each has a *why*; the reasoning is what lets the rule generalise. Other rules in this skill are directional advice. The ones below are gates.

Several are corollaries of one principle: **you execute the plan; you don't extend it**. The plan is the source of authority for what belongs in this parcel; work that doesn't trace to it is fabrication dressed as diligence.

1. **Commits stay local during the parcel.** No pushes, not at the end of a phase, not after a "productive" stretch.
    - *Why:* the user elected this skill specifically because they want commits held locally until they say otherwise. A push is a broadcast, and the user is the only audience until they decide otherwise. "I'll push just this one so it's backed up" is how this workflow degrades into the remote variant without the election. If they want remote, they'll ask.

2. **Write the tracker before the work, not after summarising it.**
    - *Why:* the tracker on disk is what a context-reset agent (you, in 30 minutes, with no memory of this conversation) reads to recover state, and it's what the user sees when they sit down for the end-of-parcel review. Holding "what's currently happening" in conversation while the tracker stays stale means: the recovery state is missing, the next time the user opens the file it doesn't reflect the work, and the conversation transcript becomes the authoritative artifact instead of the tracker. The rhythm is: open the phase → write the tracker → commit the tracker → start the work; commit code → update the tracker → commit again. Not: start the work → make a few commits → eventually summarise into the tracker. The tracker update *leads* the work; it doesn't trail it. If you've made three commits without a corresponding tracker entry, you've already failed — recover by writing the tracker now, not after the next commit.

3. **History is append-only. No force push (even later), no amending commits.**
    - *Why:* the commit chain is the record of how the work unfolded. Rewriting it — before or after a push — erases that trail. If a bad commit landed, make a new commit that fixes it. This holds even when the user later asks for a push; the no-force-push rule survives the push election.

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
