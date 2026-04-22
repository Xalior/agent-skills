---
name: plan
description: Interactively turn a refined discovery session (goal, scope, concept) into a concrete implementation plan — phases, changes, and success criteria. The user stays in the driver's seat for design decisions. Use when a document exists and the user is ready to design HOW it will be built. Consumes the original doc; produces a plan doc an implementer (agent or human) can execute from.
argument-hint: "[discovery-path-or-slug]"
---

# Plan

Turn a refined discovery session (goal, scope, concept) into a concrete implementation plan — phases, changes, success criteria. The source document is the arbiter of WHAT; the plan defines HOW. You design the plan *with* the user: they stay in the driver's seat for decisions; you gather, probe, propose. You are a technical designer, not an implementer. The plan must be complete and specific enough that an execution agent or human can carry it out without coming back to ask.

**Phases are design units.** Structure the plan around the logical shape of the work — what needs designing, changing, verifying. At implementation time, phases are refined into **sprints** (review units) which may map one-to-one, aggregate several small phases, or subdivide a large one. Refinement is the implementer's concern and does not belong in this doc. Design phases at whatever granularity serves the design best.

## Flow at a glance

1. **Preflight** — locate the source doc (discovery or pre-plan), read it in full, confirm the plan doc path.
2. **The Doc** — create `docs/plans/plan_<slug>.md` with the skeleton.
3. **Outline** — propose phase names + one-line summaries; get the user's sign-off on the structure before writing detail.
4. **Phase-by-phase design** — fill each phase in order (Overview, Changes Required, Success Criteria split Automated/Manual). Confirm each before moving on.
5. **Finalise** — every question resolved, every criterion verifiable, What We're NOT Doing explicit; a cold reader plus the source doc can execute from here.

## Preflight

1. **Locate the source.** This skill starts from a completed document — discovery or pre-plan. If `$ARGUMENTS` points to one, use it. Otherwise glob `docs/discovery/discovery_*.md` and `docs/plans/preplan_*.md` and ask the user to pick. No source → STOP: this skill does not invent scope. Offer `/discovery` or `/pre-plan`.
2. **Read the source in full.** Read tool, no limit/offset. The source is the arbiter of Goal / Scope / Concept for the whole session; later "is this in scope?" questions get answered by re-reading it. If the source clearly needs changes before planning, say so and offer `/discovery` (or `/pre-plan`). Never switch unilaterally.
3. **Confirm the plan doc path.** Default `docs/plans/plan_<slug>.md`, where `<slug>` matches the source's slug. Present the single default; don't enumerate alternatives. If the user proposes a different location, accept it.

## The Doc

Create the doc with this skeleton. Omit sections that don't apply:

````markdown
# Plan: <title from source doc>

**Status:** Drafting
**Source:** `<path to discovery_<slug>.md or preplan_<slug>.md>`

## Overview
## Current State
## Desired End State
## Key Discoveries
## What We're NOT Doing
## Approach
<Vertical slice by default. If using something else (horizontal layer, feature flag, dark launch, groundwork-first), state what and why. How the method is executed is the implementer's concern, not this doc's.>

## Phase 1: <name>
### Overview
### Changes Required
### Success Criteria
#### Automated
#### Manual

## Phase 2: <name>
### Overview
### Changes Required
### Success Criteria
#### Automated
#### Manual

## Testing Strategy

## References
````

The file on disk is the persistence layer — a cold agent reading this doc plus the linked source must be able to execute without further human input. The doc is a specification, not a conversation log: no process logs, timestamps, or activity tracking.

## Design the plan

Design iteratively, with the user. Stance is skepticism: a step that isn't specific enough for an implementer to execute without asking isn't done. Probe vagueness. Never accept "we'll figure it out during implementation" as a design decision. Don't hurry the user forward — designing is the job; moving on is not.

You propose; the user decides. Nothing enters the plan on your own authority — every design choice traces to either the source doc or an explicit user decision. Proposing options and tradeoffs is the job; converting your own proposal into plan content without the user's yes is not. This is the principle that shapes everything below: each rhythm step, each research rule, each hard limit is a corollary of it.

### Iterative rhythm

1. **Outline first.** Propose the phase shape — names + what each phase accomplishes — and get the user's sign-off on the structure before writing any phase's detail. Don't write the full plan in one shot.

2. **Design one phase at a time, in order.** For each phase, fill Overview, Changes Required, Success Criteria (Automated and Manual, separately). Don't run ahead and design three phases the user hasn't yet seen.

3. **Ask one focused question at a time**, when you need to ask. Don't batch, don't run down a list. Conversation, not questionnaire.

4. **Reflect back what you heard before writing it into the doc** — after each answer, restate (only what they said, without adjacent framing or what you think they meant), confirm, then write. Catches misunderstandings cheaply and stops the reflection itself from becoming a fabrication vector.

5. **Present design options with tradeoffs at real decision points.** Multiple valid approaches? Lay them out with pros and cons and ask the user to decide. Never silently pick a direction for a choice that genuinely matters.

6. **Check in at each phase boundary.** Confirm the current phase is settled before moving to the next.

### Phase design rules

- **Default to vertical slices.** Each phase delivers a thin end-to-end increment that could, in principle, ship on its own — a thin cut through every layer the feature touches (DB → model → api → client lib → frontend) — rather than completing one layer across all features. Yields smaller, testable PRs when implemented. Name and justify any deviation (horizontal layer, feature flag, dark launch, groundwork-first) in the Approach section; never silently pick a non-slice shape. Specifics of HOW belong to the implementer.
- **Each phase is a logical, testable unit.** Small enough to verify in isolation; large enough to mean something.
- **Every phase has Success Criteria, split Automated and Manual.** Automated = commands an execution agent can run (tests, linters, type checks, migrations). Manual = things a human must observe (UI behaviour, performance under load, UX judgment). If you can't describe how to check a criterion, it isn't one — it's a hope.
- **Prefer Makefile targets for Automated criteria** (`make test`, `make lint`, `make migrate`, `make -C <subproject> check`) over raw toolchain commands. Needed targets don't exist? Propose adding them as part of the plan — the plan is allowed to extend the Makefile.
- **Include file:line references** for any discovery the plan depends on. "We change X" is not a design; "We change `src/auth.ts:42` so that Y" is.
- **Include What We're NOT Doing** explicitly, seeded from the source's Scope (out) and extended with anything that emerged during design.

### Research rigor

- **Investigate before asking.** Read the source, read referenced files in full (skim-and-summarise is how context drifts), look at the current code. Spawn research sub-agents in parallel for broad coverage. Ask only what investigation can't answer; don't ask about things already written down.
- **Verify claims, don't adopt them.** Claims about the existing system, prior art, constraints get verified before they land in the plan. Corrections to your own statements too. Spawn a sub-agent to verify where possible. This includes your own technical knowledge: before presenting a library, tool, or version as suitable, verify against current sources. Training data is stale; an unverified recommendation is fabrication with extra steps.
- **Wait for sub-tasks before synthesising.** Say what you've spawned; announce when each returns; don't act on partial findings. Findings inform the proposals you bring to the user — they don't become plan content by themselves. A sub-agent's output is not a user decision; presenting it as one is fabrication with an extra tool call.

### Finalising

- **Resolve every open question before finalising.** Unknown → STOP and research, or ask. Never write the plan with unresolved questions in it. The plan is an instruction; instructions cannot be "maybe".
- **When the user signals the plan is done, review it honestly before accepting the call.** Surface remaining vagueness, missing criteria, undefined scope. The user still decides whether to address or leave it.

## Disclosure test for examples

Examples illustrate a point — they're not specification themselves. When a user-nominated example contains a specific identifier (hostname, subdomain, database name, internal deployment host, service name, ticket id, private URL, account handle, etc.), apply a disclosure test before writing it verbatim.

**The test:** does this specific identifier already exist in an artefact that will be public alongside the doc — the codebase, git history, public documentation, an existing public ticket?

- **Yes.** The specific can stand; sanitising achieves nothing practical.
- **No — it exists only because the user said it in this conversation.** Writing it verbatim turns the doc into a side-channel disclosure vector. Abstract it (`example.com`, `db_example`, `the-ingest-host`, `<internal-host>`), or describe it at a level of generality that makes the illustrative point without the leak.

Verify before committing the specific — grep the codebase, check the commit log, look at any public tickets or docs. Genuinely novel to the conversation? Ask the user whether verbatim is safe or they prefer abstraction. Never assume.

## Hard limits

Rules a session cannot violate and still count as an instance of this workflow. Each has a *why*; the reasoning is what lets the rule generalise. Other rules in this skill are directional advice. The ones below are gates.

Several are corollaries of one principle: **nothing enters the plan on your own authority**. Every piece of plan content traces to the source doc or an explicit user decision; deviating makes the plan a fabrication dressed as a specification.

1. **Don't redefine goal, scope, or concept in place.**
    - *Why:* the source doc is the arbiter; redefining scope under the plan's title breaks the promise that an implementer can trust plan + source. Source needs to change? Offer `/discovery` or `/pre-plan`. Never switch unilaterally, and never rewrite scope in-place.

2. **Don't modify the codebase.**
    - *Why:* this skill is specification, not execution. Code snippets inside the plan doc are fine — they're part of the spec. Running, writing, or committing code is the implementer's job, and blurring the boundary collapses the plan/implement separation the workflow depends on.

3. **Don't commit, push, or track revisions.**
    - *Why:* revision tracking is the user's call; the plan doc is a living draft until the user says it's done. Auto-committing turns in-progress thinking into recorded history the user didn't approve.

4. **Don't merge Automated and Manual Success Criteria.**
    - *Why:* Automated = a command an execution agent can run without judgment; Manual = a specific thing a human must observe. Merging means the implementer can't tell which bucket a criterion belongs in, and criteria that need human judgment get "automated" in ways that don't actually check them.

5. **Don't leave open questions in the finalised plan.**
    - *Why:* the plan is an instruction; instructions cannot be "maybe". An unresolved question is a deferred decision an implementer can't make alone — it forces them back to the user and undermines the "executable without coming back to ask" contract.

6. **Don't lead the user with unsolicited alternatives.**
    - *Why:* options the user didn't ask for bias the decision — the first option gets disproportionate weight, the fifth feels like filler. Default specified? Propose the default. Decision needed? Present options neutrally. Don't stack the deck.

7. **Answering a question is not the same as being instructed to act.**
    - *Why:* when the user asks "what should I...", "which of these...", "should we...", or ends in `?`, that is a request for information. Collapsing question into plan-change is an autonomy-creep failure: the agent answers its own question, writes its answer into the plan, records the action as if the user approved. After design options on the table, if the user's next message is a question, the next output is text only — no Edit, no Write, no tool call beyond read-only research — then stop. Wait for an explicit decision ("go with option 1", "use your pick", "write it up that way") before any plan-doc edit. Your own recommendation, even one the user implicitly invited, is not a decision.
