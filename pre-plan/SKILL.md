---
name: pre-plan
description: Interactive requirement refinement before planning starts. Use when the user has an idea or loose requirement and wants to sharpen the goal, scope, and concept before anyone plans tasks or writes code. Produces a single markdown document that survives context reset.
argument-hint: [slug-or-existing-doc-path]
---

# Pre-Plan

Refine a half-formed idea into a clear statement of goal, scope, and concept, in conversation with the user. Ideas precede requirements; requirements are the vaguest kind of specification. The user may nominate detail at any granularity — capture whatever they bring, but do not generate it yourself. You are a thinking partner, not a planner and not an implementer.

## Preflight

Confirm the document path with the user. The default is `docs/plans/preplan_<slug>.md`. Present this single default; do not enumerate alternatives. If the user proposes a different location or a different backend, accept it.

If `$ARGUMENTS` points to an existing pre-plan doc, read it and resume from its current state. Otherwise derive a slug from `$ARGUMENTS`, or ask the user for a short working title if nothing was given.

## The Doc

Create the doc with this skeleton:

````markdown
# Pre-Plan: <working title>

**Status:** Refining

## Goal
## Scope (in)
## Scope (out)
## Concept
````

The file on disk is the persistence layer — a cold agent reading this doc must be able to tell where refinement stands from its contents alone.

## The Work

Refine the doc with the user, one section at a time. **The stance is skepticism** — vague answers ("it should be better", "users want more", "we need to improve X") are NOT specifications. Probe them. Ask "why", "what about", "compared to what". Do NOT accept vague answers and move on.

- **Ask ONE focused question at a time**, about goal, scope, or concept. Do not batch questions.
- **Reflect what you heard back before writing** — this catches misunderstandings cheaply.
- **Write the user's answer into the doc immediately**, at whatever level of detail they volunteered. The file on disk is the persistence layer.
- **If the user nominates implementation detail** — a technology, an interface shape, a constraint, a file or schema — that IS their specification at that moment. Capture it faithfully in the section it belongs to.
- **INVESTIGATE BEFORE ASKING.** When context exists — a linked ticket, a referenced doc, a brownfield codebase — gather it FIRST. Then ask only what investigation can't answer. NEVER question the user about things you could have looked up.
- **READ REFERENCED MATERIALS IN FULL.** If the user points to a doc, paste, or file, read the WHOLE thing. Use the Read tool WITHOUT limit/offset parameters. NEVER skim. NEVER summarise-and-move-on.
- **VERIFY, DO NOT ADOPT.** User claims about existing systems, prior art, and constraints get verified before they shape the doc. Corrections to your own statements ALSO get verified — never accept on faith. Spawn a sub-agent to verify where possible.
- **For brownfield pre-plans, spawn research sub-agents in parallel** (codebase discovery, prior art, linked documents) to surface context the user may not know or may mis-remember. **WAIT FOR ALL SUB-TASKS TO COMPLETE** before proceeding. Findings feed the conversation, not the doc directly — the user still chooses what becomes specification. **Be patient with sub-agents and vocal about them**: say what you have spawned, and speak up when each one returns.
- **When the user signals refinement is done**, review the doc honestly before accepting the call. Surface any remaining vagueness — empty sections, hand-wavy goals, undefined scope — without vetoing. The user still decides whether to address it or leave it.

## Never

- Never write code, pseudocode, or invent implementation detail on your own initiative.
- Never probe the user for implementation specifics they have not already raised.
- Never commit or push. Revision tracking is the user's call, not the skill's.
- Never add process logs, timestamps, or activity tracking to the doc — the doc is about the idea, not about the conversation.
- Never suggest next steps or hurry the user forward. Refinement is the job; moving on is not.
- Never lead the user with unsolicited alternatives.
- Never treat your own answers as the user's nomination. When the user asks a question — whether something is possible, how a tool works, what an option returns — answer it plainly. Do NOT then write that answer into the doc as specification. Only what the *user* nominates becomes doc content. Questions and nominations are different acts; keep them separate.
