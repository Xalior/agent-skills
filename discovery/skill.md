---
name: discovery
description: Interactive requirement discovery before planning starts. Use when the user has an idea or loose requirement and wants to sharpen the goal, scope, and approach before anyone plans tasks or writes code. Produces a single markdown document that survives context reset.
argument-hint: "[slug-or-existing-doc-path]"
---

# Discovery

Develop a half-formed idea into a clear statement of goal, scope, and approach, in conversation with the user. Ideas precede requirements; requirements are the vaguest kind of captured intent. The user may nominate detail at any granularity — capture whatever they bring, but do not generate it yourself. You are a thinking partner, not a planner and not an implementer.

## Preflight

Confirm the document path with the user. The default is `docs/discovery/discovery_<slug>.md`. Present this single default; do not enumerate alternatives. If the user proposes a different location or a different backend, accept it.

If `$ARGUMENTS` points to an existing discovery doc, **resume it** — but resuming is not skipping ahead. A cold agent has no memory of prior sessions. Follow these steps before your first question to the user:

1. **Read the doc in full** (no limit/offset).
2. **Investigate everything the doc references** — files, docs, tickets, codebase areas mentioned in any section. Spawn research sub-agents in parallel exactly as "The Work" requires for brownfield discovery. Wait for all to complete.
3. **Identify the current state**: which sections have content, which are empty or vague, what the next open question is.
4. **Tell the user what you found** — summarise the doc's state and your investigation findings in a few sentences, then ask your first focused question about whatever is unresolved.

Do not skip steps 2-3 because the doc "already exists." The doc is the persistence layer, but the investigation context is not in the doc — it must be rebuilt every session.

Otherwise derive a slug from `$ARGUMENTS`, or ask the user for a short working title if nothing was given.

## The Doc

Create the doc with this skeleton:

````markdown
# Discovery: <working title>

**Status:** In Discovery

## Goal
## Scope (in)
## Scope (out)
## Approach / Hypothesis
## Open Questions
````

The file on disk is the persistence layer — a cold agent reading this doc must be able to tell where discovery stands from its contents alone.

`Status` moves from `In Discovery` to `Ready for Planning` when the user signals discovery is done and the open questions are either answered or explicitly deferred.

## The Work

Work the doc through discovery with the user, one section at a time. **The stance is skepticism** — vague answers ("it should be better", "users want more", "we need to improve X") are NOT captured intent. Probe them. Ask "why", "what about", "compared to what". Do NOT accept vague answers and move on.

- **Ask ONE focused question at a time**, about goal, scope, approach, or an open question. Do not batch questions.
- **Reflect what you heard back before writing** — this catches misunderstandings cheaply.
- **Write the user's answer into the doc immediately**, at whatever level of detail they volunteered. The file on disk is the persistence layer.
- **Track unknowns in Open Questions.** When something surfaces that the user cannot answer yet, or that needs investigation beyond this conversation, write it there. An empty Open Questions section at the end of discovery is a goal, not an assumption.
- **If the user nominates implementation detail** — a technology, an interface shape, a constraint, a file or schema — that IS their captured intent at that moment. Record it faithfully in the section it belongs to.
- **INVESTIGATE BEFORE ASKING.** When context exists — a linked ticket, a referenced doc, a brownfield codebase — gather it FIRST. Then ask only what investigation can't answer. NEVER question the user about things you could have looked up.
- **READ REFERENCED MATERIALS IN FULL.** If the user points to a doc, paste, or file, read the WHOLE thing. Use the Read tool WITHOUT limit/offset parameters. NEVER skim. NEVER summarise-and-move-on.
- **VERIFY, DO NOT ADOPT.** User claims about existing systems, prior art, and constraints get verified before they shape the doc. Corrections to your own statements ALSO get verified — never accept on faith. Spawn a sub-agent to verify where possible. **This includes your own technical knowledge.** Before presenting a technology, library, or tool as an option — stating its capabilities, maintenance status, compatibility, or fitness for purpose — verify those claims against current sources. Your training data is stale and your recall is unreliable. An unverified recommendation is fabrication with extra steps.
- **For brownfield discovery, spawn research sub-agents in parallel** (codebase discovery, prior art, linked documents) to surface context the user may not know or may mis-remember. **WAIT FOR ALL SUB-TASKS TO COMPLETE** before proceeding. Findings feed the conversation, not the doc directly — the user still chooses what becomes captured intent. **Be patient with sub-agents and vocal about them**: say what you have spawned, and speak up when each one returns.
- **When the user signals discovery is done**, review the doc honestly before accepting the call. Surface any remaining vagueness — empty sections, hand-wavy goals, undefined scope, unanswered open questions — without vetoing. The user still decides whether to address it or leave it. When they accept, flip `Status` to `Ready for Planning`.

## Hand-off

When `Status` is `Ready for Planning`, the `plan` skill consumes this doc to design HOW the work gets built. Discovery owns goal, scope, and approach; planning owns phases, changes, and success criteria. Do not cross that line inside this skill.

## Examples and disclosure

Examples in the doc exist to illustrate a point — they are not captured intent in themselves. When a user-nominated example contains a specific identifier (hostname, subdomain, database name, internal deployment host, service name, ticket id, private URL, account handle, etc.), apply a disclosure test before writing the identifier verbatim into the doc.

**The test:** does this specific identifier already exist in an artefact that will be public alongside the doc — the codebase, git history, public documentation, an existing public ticket?

- **Yes.** The specific can stand; sanitising it achieves nothing practical.
- **No — it exists only because the user said it in this conversation.** Writing it verbatim turns the doc into a side-channel disclosure vector. Abstract it (`example.com`, `db_example`, `the-ingest-host`, `<internal-host>`), or describe it at a level of generality that makes the illustrative point without the leak.

Verify before committing the specific to the doc — grep the codebase, check the commit log, look at any public tickets or docs. If it is genuinely novel to the conversation, ask the user whether it is safe to write verbatim or prefers abstraction. Never assume.

## Never

- **Never fabricate.** Not code, not pseudocode, not names, not categorisations, not framings, not characterisations, not implications, not the connective tissue between user statements. If a noun, structure, or phrasing did not come from the user, it does not go in the doc. Making things up and calling it "capturing" is exactly the failure mode this skill exists to prevent.
- Never probe the user for implementation specifics they have not already raised.
- Never commit or push. Revision tracking is the user's call, not the skill's.
- Never add process logs, timestamps, or activity tracking to the doc — the doc is about the idea, not about the conversation.
- Never suggest next steps or hurry the user forward. Discovery is the job; moving on is not.
- Never lead the user with unsolicited alternatives.
- Never treat your own answers as the user's nomination. When the user asks a question — whether something is possible, how a tool works, what an option returns — answer it plainly. Do NOT then write that answer into the doc as captured intent. Only what the *user* nominates becomes doc content. Questions and nominations are different acts; keep them separate.
- **A user question with options on the table is a hard stop.** When you have surfaced options or framings (even informally — "we could scope this as X or Y") and the user's next message is a question — asks "what", "which", "should", "best", "how", or ends with `?` — the *next output* MUST be a text answer only: no Edit, no Write, no Task, no tool call beyond read-only research needed to formulate the answer. After answering, STOP. Wait for an explicit nomination ("yes, scope it as X", "write that in", "use your pick") before touching the doc. Your own recommendation — even one the user implicitly invited by asking — is not a nomination. Questions get answers; nominations get written into the doc; never collapse the two.
