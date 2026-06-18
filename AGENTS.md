# Agent Skill Design Guide

Knowledge and patterns for creating effective Claude Code skills.

## Core Principles

### 1. Single Responsibility
Each skill should have **one clear purpose**. Don't combine fundamentally different workflows into a single skill with flags.

**Good:** Separate skills for local-only vs. remote-push workflows
**Bad:** One skill with a `--remote` flag that changes core behavior

**Why:** Separate skills make intent explicit, reduce accidental misuse, and are easier to understand.

### 2. Name Consistency
The skill name in YAML frontmatter **must match** the directory name exactly.

```markdown
---
name: implement-with-feedback
description: ...
---
```
Directory: `implement-with-feedback/SKILL.md`

### 3. When to Offer
The `description` field should help Claude know **when to proactively offer** the skill.

**Good patterns:**
- "Use when starting any plan-based implementation task"
- "Use when the user asks to build web components, pages, or applications"
- "Analyze other agents' sessions and construct targeted corrective prompts"

**Include triggering scenarios** so Claude can pattern-match user requests.

### 4. Argument Hints
Use `argument-hint` to show expected parameters:

```yaml
argument-hint: <branch-type>/<description> [plan-file]
```

This helps both the user and Claude understand how to invoke the skill.

## Workflow Structure

### Phase-Based Organization
Break complex workflows into clear, numbered phases:

```markdown
### Phase 1: Pre-flight Checks
### Phase 2: Branch Creation
### Phase 3: WIP Progress File
### Phase 4: Implementation Loop
### Phase 5: Completion
```

**Benefits:**
- Easy to follow
- Clear checkpoints
- Natural place for validation steps
- Resume-friendly (can pick up at any phase)

### Pre-flight Checks
Always start with validation:
- Check working tree state
- Verify branch
- Confirm prerequisites exist
- Pull latest changes

**Pattern:**
```markdown
1. **Verify X.** Run `command`. If condition fails, **STOP** and tell the user:
   > "Error message here."
   Do NOT proceed until fixed.
```

The explicit "STOP" and "Do NOT proceed" language is critical for Claude to respect boundaries.

## Rules Section

### Stating Constraints
Use **NEVER** for hard boundaries:

```markdown
## Rules

- **NEVER force push.** History is sacred in this workflow.
- **NEVER amend pushed commits.** Make a new commit instead.
- **NEVER skip pushing.** Every commit must be pushed immediately.
```

**Why NEVER works:**
- Unambiguous
- Easy to search for
- Creates strong behavioral constraints
- No room for interpretation

### Commit Discipline
If the skill involves git, specify commit patterns:

```markdown
- `feat: add auth middleware for API routes`
- `fix: handle null response from scanner`
- `wip: partial implementation of results table`
- `docs: update scanner authoring guide`
- `test: add normalizer tests for ffuf`
- `refactor: extract fingerprint logic to shared util`
```

Provide examples for each commit type the agent might need.

## Living Documentation Pattern

Skills can create **progress tracking files** that evolve during execution:

```markdown
### Phase 3: WIP Progress File

1. **Create `docs/plans/plan_<branch-name>_implementation.md`**
2. **Initial content:** [template here]
3. **Commit the WIP file immediately**
```

**Benefits:**
- Self-documenting work
- Useful for monitoring
- Creates reviewable history
- Helps agent track progress
- Provides context for future sessions

**Key elements of WIP files:**
- Status field
- Task checklist
- Progress log with timestamps
- Decisions & notes section
- Blockers section
- Commit log

## Common Patterns

### Argument Handling
```markdown
If `$ARGUMENTS` is provided, use it as the branch name directly
(e.g. `/implement-with-feedback feature/add-auth`).
Otherwise, ask the user what kind of work this is and derive a branch name.
```

Make it work with or without arguments.

### Conditional Behavior
```markdown
**Verify we are on main/master.** If not, warn the user and ask whether
to continue from the current branch or switch to main first.
```

Don't assume - ask when there's ambiguity.

### Completion Actions
```markdown
3. **Inform the user** the branch is ready for review / PR creation.
   Offer to create the PR.
```

Always offer next logical steps at completion.

## When to Split Skills

Consider separate skills when:

1. **Core behavior differs** (local vs. remote, sync vs. async)
2. **Safety requirements differ** (destructive vs. safe operations)
3. **Target audience differs** (solo vs. team, internal vs. external)
4. **Workflow branches significantly** (not just parameter tweaks)

Consider combining when:
1. Only presentation differs
2. It's truly just an optional parameter
3. The core flow is identical
4. Safety constraints are the same

## Skill Metadata

### Required Fields
```yaml
---
name: skill-name
description: Clear description with "Use when..." triggers
---
```

- **`name`** — lowercase, hyphen-separated, ≤ 64 chars, and **must match the directory** (rule #2 above). The lone exception in this repo is `pre-plan`, a deliberate symlink to `discovery`.
- **`description`** — the *only* part of the skill loaded into context until it triggers, so it carries the entire matching burden. Keep it **≤ 1024 characters**, write it in the **third person** ("Analyze…", "Turn a discovery doc into…", "Use when…"), and pack it with concrete trigger phrases. This is the single highest-leverage field for correct invocation.

### Optional Fields
```yaml
argument-hint: <type>/<name> [optional-param]
allowed-tools: Read, Glob, Grep
```

- **`argument-hint`** — show expected parameters to the user and the model.
- **`allowed-tools`** — restrict the skill to a named set of tools. Use it to turn a prose constraint into an enforced guardrail: a read-only analysis skill should declare `Read, Glob, Grep` rather than merely *asking* the model not to write; a CLI-wrapper skill can scope to `Bash, Read`. If omitted, the skill inherits the session's full toolset.

## Progressive Disclosure

A skill is loaded in layers, and you control how much context each layer costs:

1. **Metadata** (`name` + `description`) — always in context, for every skill, every session. Pays rent constantly; keep it tight.
2. **`SKILL.md` body** — loaded only when the skill triggers. Target **under ~500 lines**. It should read as the *map* of the workflow, not the full territory.
3. **Bundled resources** (`references/`, scripts, templates) — loaded on demand, only when the body points the model at them.

**Push depth down a layer.** Exhaustive command tables, per-strategy procedure, long preflight detail — anything the model needs *sometimes* but not to understand the shape of the work — belongs in `references/`, with the body carrying a one-line pointer to it.

**Patterns in this repo:**
- `implement-with-feedback` / `*-remote-feedback` keep the body as the phase map and push step-by-step procedure into `references/preflight.md`, `pr-strategies.md`, `feedback-loop.md`.
- `cmux` keeps global options, the ID system, and common patterns in the body, and moves the full per-command reference to `references/commands.md` — so the always-on-when-triggered cost stays small even though the CLI surface is huge.

When a body starts growing past ~500 lines or repeating a reference table inline, that's the signal to split.

## Testing & Iteration

### Validation Checklist
- [ ] Name matches directory (lowercase, hyphens, ≤ 64 chars)
- [ ] Description is third-person, ≤ 1024 chars, and includes "Use when..." triggers
- [ ] `allowed-tools` declared where the skill has a constraint worth enforcing (read-only, CLI-only)
- [ ] Body is under ~500 lines; deep/occasional detail pushed to `references/`
- [ ] Pre-flight checks prevent bad states
- [ ] Rules section uses NEVER for boundaries
- [ ] Phases are clearly numbered
- [ ] Argument handling is documented
- [ ] Completion step offers next actions
- [ ] Commit patterns provided (if git-related)

### Common Pitfalls
1. **Ambiguous stopping conditions** - Always use explicit "STOP" language
2. **Missing pre-flight checks** - Validate before action
3. **Combining unrelated workflows** - Split instead
4. **Vague rule language** - Use NEVER for hard boundaries
5. **No completion guidance** - Always offer next steps

## Examples from This Repo

### agent-feedback
- **Purpose:** Analyze agent sessions and construct corrective prompts
- **Pattern:** Multi-phase analysis with markdown output
- **Key feature:** Structured feedback format

### implement-with-feedback
- **Purpose:** Local-only git workflow with WIP tracking
- **Pattern:** Pre-flight → Branch → Track → Loop → Complete
- **Key feature:** Living documentation (WIP file)

### implement-with-remote-feedback
- **Purpose:** Remote-push git workflow with monitoring
- **Pattern:** Same as above but with continuous push
- **Key feature:** Real-time monitoring via git log
- **Why separate:** Fundamentally different push behavior (NEVER vs. ALWAYS)

### frontend-design
- **Purpose:** Create distinctive, production-grade UI
- **Pattern:** Design-focused generation
- **Key feature:** Avoids generic AI aesthetics

## Naming Conventions

### Skill Names
- Use kebab-case
- Be descriptive but concise
- Include key differentiators (`with-feedback`, `remote-feedback`)

### File Paths in Instructions
When creating files, use consistent paths:
- `docs/plans/` for planning documents
- `docs/wip/` for work-in-progress tracking
- Include branch name in filename for traceability

### Branch Prefixes
Standardize on common types:
- `feature/` — new functionality
- `bugfix/` — fixing a defect
- `spike/` — exploratory / research
- `refactor/` — restructuring
- `docs/` — documentation only
- `chore/` — maintenance, deps, tooling

## Meta-Knowledge

### How Claude Uses Skills
1. Claude sees skill descriptions in system reminders
2. Pattern matches user requests to "Use when..." descriptions
3. Loads skill content as extended prompt
4. Follows phases sequentially
5. Respects NEVER boundaries
6. Offers skill proactively when triggered

### Skill Invocation
Users invoke with `/skill-name [args]` or Claude can suggest them.

### Argument Passing
`$ARGUMENTS` contains everything after the skill name:
- `/implement-with-feedback feature/auth docs/plan.md`
- `$ARGUMENTS` = "feature/auth docs/plan.md"

Parse positional parameters or use the whole string as needed.

## Evolution & Maintenance

### When to Update
- User reports unexpected behavior
- New git commands available
- Workflow improvements discovered
- Safety issues identified

### Version Control
Skills are versioned with the repo. Document significant changes in commit messages:
```
feat(skills): add remote monitoring to implement-with-feedback
fix(skills): prevent double-push in completion phase
docs(skills): clarify branch naming conventions
```

### Deprecation
If replacing a skill:
1. Create new version
2. Keep old version with deprecation notice
3. Document migration path
4. Remove after transition period

---

**Remember:** Skills are prompts that guide Claude's behavior. Write them as if you're teaching a diligent but literal-minded assistant. Be explicit, provide examples, and state boundaries clearly.
