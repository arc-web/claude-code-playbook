# 10 - Plan Mode

Claude proposes what it intends to do before executing. Required for complex tasks; skipped for simple ones. Without this, autonomous sessions and agent teams can go off-track and waste tokens on a bad decomposition.

---

## Prerequisites

- `CLAUDE.md` with critical paths defined (Guide 02)
- Worktree launchers in place (Guide 01)

---

## What Counts as Complex

Add to `CLAUDE.md`. Claude uses this to self-select whether to enter plan mode.

### Complex - Plan Mode Required
- Any task touching 3+ files
- Any task involving critical path files (define in `CLAUDE.md`)
- Any new feature implementation (not a fix to existing code)
- Any refactor or architectural change
- Any database migration or schema change
- Any agent team session (team lead must plan before spawning teammates)
- Any task started via `tw-auto` (autonomous sessions always plan first)
- Any task touching CI/CD configuration or deployment files
- Any time the dev explicitly says "plan this" or "plan mode"

### Simple - Execute Directly
- Single-file bug fix with clear cause
- Typo, rename, or formatting fix
- Updating a dependency version
- Adding or modifying a single test
- README or documentation-only changes
- Any time the dev explicitly says "just do it" or "skip plan"
- Any task where the review agent already described the exact fix in a card (the card IS the plan)

**When unsure: default to plan mode.**

---

## Plan Output Format

When Claude enters plan mode, it outputs this format:

```
┌─────────────────────────────────────────────────────┐
│ PLAN | [task summary in one line]                    │
│─────────────────────────────────────────────────────│
│ Files to modify:                                     │
│ > [file1] - [what changes]                           │
│ > [file2] - [what changes]                           │
│─────────────────────────────────────────────────────│
│ Steps:                                               │
│ 1. [first action]                                    │
│ 2. [second action]                                   │
│ 3. [third action]                                    │
│─────────────────────────────────────────────────────│
│ Risk: Low / Medium / High                            │
│ Scope: small / medium / large                        │
│ Tests affected: [list or "none"]                     │
│─────────────────────────────────────────────────────│
│ Approve? (y to proceed, n to revise, or give notes)  │
└─────────────────────────────────────────────────────┘
```

Rules for plan format:
- Max 10 steps; group related steps if more are needed
- Every file that will be modified must be listed upfront - no surprise edits
- Risk assessment: Low (isolated), Medium (multiple files, some dependencies), High (architectural / critical path)
- Any critical path file in the plan automatically escalates risk to Medium or Higher
- Dev approves, rejects, or gives revision notes before Claude proceeds

---

## Agent Team Planning

Before the team lead spawns any review subagents, it must output a team plan:
- How many agents will be spawned
- What each agent's scope is (which files/modules)
- Expected completion order
- How findings will be synthesized

The dev approves the team plan before any agents spawn.

Add this instruction to your `review-lead.md` agent:
```markdown
Before spawning any review agents, output a team plan showing: how many agents, each agent's scope, expected order, how findings will be synthesized. Wait for human approval before spawning.
```

---

## Session Enforcement

Update `tw.sh` to inject plan mode guidance based on session type:

| Session | Plan mode behavior |
|---------|-------------------|
| `tw` | Claude self-selects based on CLAUDE.md complexity rules |
| `tw-auto` | Plan mode mandatory for the first task; auto-accept takes over for execution after approval |
| `tw-sched` | Optional; the prompt itself is the plan |

For `tw-auto`, inject at session start by adding to the launcher:

```bash
tw_auto() {
  # ... worktree setup ...
  SYSTEM_PROMPT="Plan mode is mandatory for the first task in this session. Present a plan and wait for approval before executing."
  claude --channels --system "$SYSTEM_PROMPT"
}
```

---

## Override Commands

These always work, regardless of complexity classification:

| Command | Effect |
|---------|--------|
| "just do it" | Skip plan mode |
| "skip plan" | Skip plan mode |
| "plan this" | Enter plan mode |
| "plan mode" | Enter plan mode |

---

## CLAUDE.md Section to Add

```markdown
## Plan Mode

Complex tasks require a plan before execution. Simple tasks execute directly.

**Complex (plan required):** 3+ files, critical paths, new features, refactors, migrations, agent teams, tw-auto sessions, CI/CD changes, or "plan this"
**Simple (execute directly):** Single-file fix, typo/rename, dependency update, single test, docs only, or "just do it"
**Default when unsure:** Plan mode.

Every plan must list all files to be modified. No surprise edits.
Critical path files in plan = automatic Medium or High risk.
Agent team leads must present a team plan before spawning teammates.
```