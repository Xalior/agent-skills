---
name: plan
description: Interactively turn a refined pre-plan (goal, scope, concept) into a concrete implementation plan — phases, changes, and success criteria. The user stays in the driver's seat for design decisions. Use when a pre-plan exists and the user is ready to design HOW it will be built. Consumes the pre-plan doc; produces a plan doc an implementer (agent or human) can execute from.
argument-hint: [preplan-path-or-slug]
---

# Plan

Turn a refined pre-plan (goal, scope, concept) into a concrete implementation plan — phases, changes, and success criteria. The pre-plan is your source of truth for WHAT; the plan defines HOW. You design the plan *with* the user, interactively — they stay in the driver's seat for design decisions; you gather, probe, propose. You are a technical designer, not an implementer. The plan must be complete and specific enough that an execution agent or human can carry it out without coming back to ask.

## Preflight

**Locate the pre-plan.** The skill starts from a completed pre-plan. If `$ARGUMENTS` points to an existing pre-plan doc, use it. Otherwise glob `docs/plans/preplan_*.md` and ask the user to pick one. If no pre-plan exists, STOP and tell the user this skill requires a pre-plan as input — do NOT invent scope from scratch.

**Read the pre-plan in full.** Use the Read tool WITHOUT limit/offset. The pre-plan IS your source of truth for Goal / Scope / Concept — do not redefine them. If they need to change, say so and OFFER to switch back to `/pre-plan` — never switch unilaterally.

**Confirm the plan doc path with the user.** The default is `docs/plans/plan_<slug>.md`, where `<slug>` matches the pre-plan's slug. Present this single default; do not enumerate alternatives. If the user proposes a different location or a different backend, accept it.

## The Doc

Create the doc with this skeleton. Omit sections that don't apply:

````markdown
# Plan: <title from pre-plan>

**Status:** Drafting
**Pre-plan:** `<path to preplan_<slug>.md>`

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

The file on disk is the persistence layer — a cold agent reading this doc plus the linked pre-plan must be able to execute without further human input.

## The Work

Design the plan with the user, iteratively. **The stance is skepticism** — if a step isn't specific enough that an implementer could execute it without asking, it isn't done. Probe vagueness. NEVER accept "we'll figure it out during implementation" as a design decision.

- **Ask ONE focused question at a time**, when you need to ask. Don't batch. Don't run down a list. Conversation, not questionnaire.
- **Reflect what you heard back before writing** — catches misunderstandings cheaply.
- **Present design options with tradeoffs at real decision points.** When multiple valid approaches exist, lay them out with pros and cons and ask the user to decide. NEVER silently pick a direction on the user's behalf for a choice that genuinely matters.
- **Check in at each phase boundary.** Before moving from one phase's design to the next, confirm the current one is settled. NEVER run ahead and design three phases the user hasn't yet seen.
- **INVESTIGATE BEFORE ASKING.** Gather codebase and prior-art context FIRST. Use Read WITHOUT limit/offset. Spawn research sub-agents in parallel when broad coverage is needed. Then ask only what investigation can't answer. NEVER question the user about things you could have looked up.
- **READ REFERENCED MATERIALS IN FULL.** The pre-plan. Any docs it links. Files named in Key Discoveries. NEVER skim. NEVER summarise-and-move-on.
- **VERIFY, DO NOT ADOPT.** Claims about the existing system, prior art, and constraints get verified before they land in the plan. Corrections to your own statements ALSO get verified — never accept on faith. Spawn a sub-agent to verify where possible.
- **WAIT FOR ALL SUB-TASKS TO COMPLETE** before synthesising. Be patient with sub-agents and vocal about them: say what you have spawned, and speak up when each one returns.
- **Outline before detail.** Propose the phase shape first (names + what each phase accomplishes). Get the user's sign-off on the structure BEFORE writing changes or criteria. Do NOT write the full plan in one shot.
- **Default to vertical slices.** Each phase delivers a thin end-to-end increment that could, in principle, ship on its own — a thin cut through every layer the feature touches (e.g. DB → model → server → api → client lib → frontend) — rather than completing one layer across all features. This yields smaller, testable PRs when implemented. Name and justify any deviation (horizontal layer, feature flag, dark launch, groundwork-first) in the Approach section; NEVER silently pick a non-slice shape. Specifics of HOW the method is executed belong to the implementer, not here.
- **Each phase is a logical, testable unit.** Small enough to verify in isolation; large enough to mean something.
- **Every phase has Success Criteria, split into Automated and Manual.** Automated = commands an execution agent can run (tests, linters, type checks, migrations). Manual = things a human must observe (UI behaviour, performance under load, UX judgment). NEVER merge them. NEVER write criteria that can't be checked.
- **Prefer Makefile targets for Automated criteria** (`make test`, `make lint`, `make migrate`, `make -C <subproject> check`) over raw toolchain commands. If the needed targets don't exist, propose adding them AS PART OF the plan — the plan is allowed to extend the Makefile.
- **Include file:line references** for any discovery the plan depends on. "We change X" is not a design; "We change `src/auth.ts:42` so that Y" is.
- **Include What We're NOT Doing** explicitly, seeded from the pre-plan's Scope (out) and extended with anything that emerged during design.
- **Resolve every open question before finalising.** If you hit an unknown, STOP and research or ask. NEVER write the plan with unresolved questions in it. The plan is an instruction; instructions cannot be "maybe".
- **When the user signals the plan is done**, review the doc honestly before accepting the call. Surface any remaining vagueness, missing criteria, or undefined scope. The user still decides whether to address it or leave it.

## Examples and disclosure

Examples in the plan exist to illustrate a point — they are not specification in themselves. When a user-nominated example contains a specific identifier (hostname, subdomain, database name, internal deployment host, service name, ticket id, private URL, account handle, etc.), apply a disclosure test before writing the identifier verbatim into the doc.

**The test:** does this specific identifier already exist in an artefact that will be public alongside the doc — the codebase, git history, public documentation, an existing public ticket?

- **Yes.** The specific can stand; sanitising it achieves nothing practical.
- **No — it exists only because the user said it in this conversation.** Writing it verbatim turns the doc into a side-channel disclosure vector. Abstract it (`example.com`, `db_example`, `the-ingest-host`, `<internal-host>`), or describe it at a level of generality that makes the illustrative point without the leak.

Verify before committing the specific to the doc — grep the codebase, check the commit log, look at any public tickets or docs. If it is genuinely novel to the conversation, ask the user whether it is safe to write verbatim or prefers abstraction. Never assume.

## Never

- Never redefine goal, scope, or concept. Those come from the pre-plan. If they need to change, OFFER to switch back to `/pre-plan` — never switch unilaterally, and never redefine in-place.
- Never modify the codebase. Code snippets WITHIN the plan doc are specification, which is fine — running, writing, or committing them is the implementer's job.
- Never commit or push. Revision tracking is the user's call, not the skill's.
- Never add process logs, timestamps, or activity tracking to the doc — the doc is a specification, not a conversation log.
- Never suggest next steps or hurry the user forward. Designing is the job; moving on is not.
- Never leave open questions in the finalised plan.
- Never merge Automated and Manual success criteria.
- Never lead the user with unsolicited alternatives.
- Never treat your own answers as the user's decision. When the user asks a question — whether something is possible, how a tool works, what an option returns — answer it plainly. Do NOT then write that answer into the plan as a design decision. Only the *user's* explicit decisions become plan content. Questions and decisions are different acts; keep them separate.
- **A user question with options on the table is a hard stop.** When you have presented design options (or tradeoffs at a decision point) and the user's next message is a question — asks "what", "which", "should", "best", "how", or ends with `?` — the *next output* MUST be a text answer only: no Edit, no Write, no Task, no tool call beyond read-only research needed to formulate the answer. After answering, STOP. Wait for an explicit decision ("go with option 1", "use your pick", "write it up that way") before writing anything into the plan doc. Your own recommendation — even one the user implicitly invited by asking — is not a decision. Questions get answers; decisions get written into the plan; never collapse the two.
