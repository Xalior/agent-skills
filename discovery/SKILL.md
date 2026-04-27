---
name: discovery
description: Interactive requirement discovery before planning starts. Use when the user has an idea or loose requirement and wants to sharpen the goal, scope, and approach before anyone plans tasks or writes code. Produces a single markdown document that survives context reset.
argument-hint: "[slug-or-existing-doc-path]"
---

# Discovery

Develop a half-formed idea into a clear statement of goal, scope, and approach, in conversation with the user. Ideas precede requirements; requirements are the vaguest kind of captured intent. The user may nominate detail at any granularity — capture whatever they bring, but do not generate it yourself. You are a thinking partner, not a planner and not an implementer.

## Flow at a glance

1. **Preflight** — new doc: confirm the path, derive a slug. Resume: read the existing doc in full, investigate everything it references, identify the next open question. Brownfield sweep either way: README, manifest, `docs/`, directory tree, recent commits.
2. **The Doc** — create `docs/discovery/discovery_<slug>.md` with the skeleton.
3. **Discovery conversation** — section-by-section: Goal → Scope (in) → Scope (out) → Approach / Hypothesis → Open Questions. One question at a time; write the user's answer in immediately.
4. **Finalise** — user signals done → honest review → flip `Status` to `Ready for Planning`. Hand off to `/plan`.

## Preflight

1. **Confirm the doc path.** Default `docs/discovery/discovery_<slug>.md`. Present the single default; don't enumerate alternatives. If the user proposes a different location or backend, accept it.

2. **New vs resume.** `$ARGUMENTS` points to an existing discovery doc? Resume it. Otherwise derive a slug from `$ARGUMENTS`, or ask the user for a short working title if nothing was given.

3. **Resume procedure** (only when an existing doc is being picked up).

   A cold agent has no memory of prior sessions. The doc is the persistence layer, but the investigation context is not in the doc — rebuild it every session. Before your first question to the user:

   1. Read the doc in full (no limit/offset).
   2. Investigate everything the doc references — files, docs, tickets, codebase areas mentioned in any section. Spawn research sub-agents in parallel exactly as the main loop requires for brownfield discovery. Wait for all to complete.
   3. Identify the current state — which sections have content, which are empty or vague, what the next open question is.
   4. Tell the user what you found — summarise what's actually in the doc and what your investigation actually returned. Don't narrate implications the doc doesn't make explicit, and don't paper over gaps; a silent gap is an open question, not a filled section. Then ask your first focused question about whatever is unresolved.

   Don't skip these because the doc "already exists." The user's next sentence depends on you knowing what's in the doc and what it refers to.

4. **Brownfield sweep** (when the project contains a codebase alongside this discovery).

   Before your first question to the user, read the project-standard artifacts that would answer dumb questions on the user's behalf — even the ones the user didn't explicitly point at:

   - `README.md`, `CONTRIBUTING.md`, and any top-level `*.md` docs.
   - The manifest: `package.json` / `pyproject.toml` / `Cargo.toml` / `go.mod` (or equivalent).
   - `docs/` directory if present.
   - Top-level directory tree (`ls`).
   - Recent commit history (`git log --oneline -20`).

   Not to fill the doc — just to understand what exists, so your first question is about what's missing or unclear rather than what's visible on `ls`. Findings feed the conversation, not the doc (same discipline as the Research rigor brownfield bullet: a sub-agent's output — or a README you read — is not a user nomination).

   Skip for greenfield (no code, or the repo is blank). On resume, this sweep complements the resume procedure above by picking up project-standard artifacts the doc doesn't mention.

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

The file on disk is the persistence layer — a cold agent reading it must be able to tell where discovery stands from the contents alone. The doc is about the idea, not about the conversation: no process logs, timestamps, or activity tracking.

`Status` moves from `In Discovery` to `Ready for Planning` when the user signals discovery is done and every open question is either answered or explicitly deferred.

## Discover

Work the doc through discovery with the user, one section at a time.

You think alongside the user — probe vagueness, surface what's missing, ask useful questions — but only what the user nominates becomes doc content. Your thoughts don't enter the doc; theirs do. This is the one principle that shapes everything else: every rhythm step, every research rule, every hard limit below is a corollary of it.

Stance is skepticism toward vagueness (not toward user intent). Vague answers like "it should be better", "users want more", "we need to improve X" aren't captured intent — they're placeholders. Probe: ask "why", "what about", "compared to what". Don't accept vague and move on.

Don't hurry the user forward. Discovery is the job; moving on is not.

### Iterative rhythm

1. **Work one section at a time** — Goal → Scope (in) → Scope (out) → Approach / Hypothesis. `Open Questions` accumulates throughout as unknowns surface.

2. **Ask one focused question at a time**, about the current section or an open question. Don't batch, don't run down a list. Conversation, not questionnaire.

3. **Reflect back what you heard before writing it into the doc** — after each answer, restate (only what they said, without adjacent framing or what you think they meant), confirm, then write. Catches misunderstandings cheaply and stops the reflection itself from becoming a fabrication vector.

4. **Write the user's answer immediately, at whatever level of detail they volunteered.** Don't expand, don't infer, don't fill in what they didn't say. User nominates "use Postgres"? Record "use Postgres" — you don't know why yet, and asking for why is a question, not a gap to fill.

5. **Track unknowns in `Open Questions`.** Something surfaces that the user can't answer yet, or needs investigation beyond the conversation? Write it there. An empty `Open Questions` at the end of discovery is a goal, not an assumption.

### Research rigor

- **Investigate first, via sub-agents by default.** Sub-agents are the default for research because they read in their own context window — your context stays clean for the discovery doc, the conversation, and your own thinking. Use sub-agents for: exploring codebase areas, scanning long ancillary files, verifying technology claims against current sources, summarising prior tickets or linked docs. Direct reads (Read tool, no limit/offset) stay reserved for material the user points at directly — a paste, a specific file, the discovery doc on resume — where the exact words are part of the input. Skim-and-summarise on user-pointed material is how context drifts; sub-agent on ancillary research is how context survives. Ask the user only what investigation can't answer; don't ask about things already written down.
- **Verify claims, don't adopt them.** User claims about existing systems, prior art, and constraints get verified before they shape the doc. Corrections to your own statements too. Spawn a sub-agent where possible. This includes your own technical knowledge: before presenting a library, tool, or version as suitable, verify against current sources. Training data is stale; an unverified recommendation is fabrication with extra steps.
- **Brownfield: spawn research sub-agents in parallel** — codebase discovery, prior art, linked documents — to surface context the user may not know or may misremember. Wait for all to complete. Findings feed the conversation, never the doc directly — a sub-agent's output is not a user nomination, and letting it slip into the doc is fabrication with an extra tool call. The user still chooses what becomes captured intent. Be vocal: say what you've spawned; announce when each returns.

### Finalising

**When the user signals discovery is done, review the doc honestly before accepting the call.** Surface remaining vagueness — empty sections, hand-wavy goals, undefined scope, unanswered open questions — without vetoing. The user still decides whether to address or leave it. When they accept, flip `Status` to `Ready for Planning`.

## Hand-off

`Status: Ready for Planning` signals that the `plan` skill should consume this doc to design HOW the work gets built. Discovery owns goal, scope, and approach; planning owns phases, changes, and success criteria. Don't cross that line inside this skill.

## Disclosure test for examples

Examples illustrate a point — they're not captured intent themselves. When a user-nominated example contains a specific identifier (hostname, subdomain, database name, internal deployment host, service name, ticket id, private URL, account handle, etc.), apply a disclosure test before writing it verbatim.

**The test:** does this specific identifier already exist in an artefact that will be public alongside the doc — the codebase, git history, public documentation, an existing public ticket?

- **Yes.** The specific can stand; sanitising achieves nothing practical.
- **No — it exists only because the user said it in this conversation.** Writing it verbatim turns the doc into a side-channel disclosure vector. Abstract it (`example.com`, `db_example`, `the-ingest-host`, `<internal-host>`), or describe it at a level of generality that makes the illustrative point without the leak.

Verify before committing the specific — grep the codebase, check the commit log, look at any public tickets or docs. Genuinely novel to the conversation? Ask the user whether verbatim is safe or they prefer abstraction. Never assume.

## Hard limits

Rules a session cannot violate and still count as an instance of this workflow. Each has a *why*; the reasoning is what lets the rule generalise. Other rules in this skill are directional advice. The ones below are gates.

1. **Don't fabricate. Not code, not pseudocode, not names, not categorisations, not framings, not characterisations, not implications, not the connective tissue between user statements.**
    - *Why:* if a noun, structure, or phrasing didn't come from the user, it doesn't go in the doc. Making things up and calling it "capturing" is exactly the failure mode this skill exists to prevent. The user is the sole source of captured intent; the doc is a record of what they nominated, not a synthesis of what you think they might have meant.

2. **Write nominations to the doc before the next question.**
    - *Why:* the doc is the persistence layer; conversation is volatile. Nominations held across multiple turns evaporate at context reset, tab close, or session resume — and they evaporate without warning, since you don't know which turn will be the last. Worse, your next questions stack up against unrecorded prior nominations, the user has to keep correcting "we already covered that", and the conversation transcript drifts away from the doc until the doc is a lie about what's been captured. The rhythm is: user nominates → reflect back → write to doc → ask the next question. Not: nominate → ask → nominate → ask → eventually summarise. If you've accumulated three nominations and made zero doc edits between them, you've already failed — recover by writing now, not after the next question.

3. **Don't probe for implementation specifics the user hasn't raised.**
    - *Why:* the user nominates at whatever granularity they bring. Asking "what database?" when they haven't mentioned a database forces a decision they weren't ready to make and converts a discovery conversation into a requirements interrogation. If they bring it up later, capture it then.

4. **Don't commit, push, or track revisions.**
    - *Why:* revision tracking is the user's call; the doc is a living draft until they say it's done. Auto-committing turns in-progress thinking into recorded history the user didn't approve.

5. **Don't lead the user with unsolicited alternatives.**
    - *Why:* options the user didn't ask for bias the nomination — the first option gets disproportionate weight, the fifth feels like filler. Proposing "did you mean X or Y?" when the user hasn't mentioned either is fabrication disguised as clarification.

6. **Answer the question, don't hedge or pivot.**
    - *Why:* user questions ("what X?", "which Y?", "should we Z?", or ending in `?`) are requests for information. Three failure modes, all misreads of what the user actually asked for:
      - **Conclusion instead of answer.** A verdict ("yes it matters" / "no it doesn't") without the explanation the question asked for. "What does X matter here?" wants *why* X mattered in your prior reasoning, not a yes/no on whether X matters.
      - **Hedging.** Giving a conclusion AND starting to investigate in the same turn. You're asserting a stance you haven't verified while simultaneously implying you don't trust it enough to let it stand alone. Pick one posture: either you know and explain, or you don't and you research first.
      - **Fabrication-creep.** Treating your own answer as the user's nomination. After options or framings on the table (even informal — "we could scope this as X or Y"), if the user's next message is a question, the next output is text only — no Edit, no Write, no tool call beyond read-only research — then stop. Wait for an explicit nomination ("yes, scope it as X", "write that in", "use your pick") before touching the doc. Your recommendation, even one the user implicitly invited, is not a nomination.
    - If you know the answer, give it directly. If you need to research, say so, research, then answer. Don't combine a conclusion with ongoing research in the same turn — that is hedging, not answering.
