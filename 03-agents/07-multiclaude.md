# 07 - Multiclaude

Multiclaude is a third-party orchestrator with a "multiplayer" mode for team-based code review. This guide covers how to evaluate it alongside your existing custom review system (Guide 05) and decide whether it adds value or duplicates it.

---

## What Multiclaude Does

- **Singleplayer mode:** PRs auto-merge without review (do not use this)
- **Multiplayer mode:** Teammates review code before merge via a supervisor/subagent model
- Manages branch/PR workflow through a supervisor that dispatches subagents

GitHub: search `dlorenc/multiclaude` for the current repo.

---

## Prerequisites

- Custom review agents set up (Guide 05) - this is the baseline for comparison
- Go runtime installed (`go version`; install from golang.org if missing)
- `gh` CLI authenticated
- At least one real PR available for testing

---

## Installation

```bash
go install github.com/dlorenc/multiclaude/cmd/multiclaude@latest
# verify
multiclaude --version
```

Configure to always use multiplayer mode. Never use singleplayer (it auto-merges without review).

---

## Evaluation Checklist

Before committing to Multiclaude, test each of these:

### Review Quality
- Run your custom `/review` on a PR
- Run Multiclaude's review on the same PR
- Which found more issues?
- Which presented findings more clearly?

### Integration Compatibility

| Area | Test | Pass/Fail |
|------|------|-----------|
| Discord | Does it post to your Discord webhooks? | |
| Card format | Can it output your tier card format? | |
| Worktrees | Does it work with your worktree setup? | |
| Session manager | Does it appear in your session manager? | |
| Custom agents | Can it use your .claude/agents/ definitions? | |
| Auto-fix | Does it respect green auto-apply rules? | |
| Scheduled tasks | Can it be triggered headlessly? | |

### Subagent Overlap
- List every agent Multiclaude defines
- Cross-reference with your agent registry
- Any that duplicate your custom agents should be removed or replaced

---

## Scorecard

Create `.claude/docs/multiclaude-evaluation.md`:

| Criteria | Custom System (1-5) | Multiclaude (1-5) | Notes |
|----------|--------------------|--------------------|-------|
| Review quality | | | |
| Review presentation | | | |
| Card format / actionability | | | |
| Discord integration | | | |
| Worktree compatibility | | | |
| Session manager compat | | | |
| Custom subagent support | | | |
| Token cost | | | |
| Speed | | | |
| Multiplayer review flow | | | |
| **TOTAL** | **/50** | **/50** | |

---

## Verdict Options

Pick one and document the rationale:

- **REPLACE** - Multiclaude is better; replace the custom review system
- **COMPLEMENT** - Multiclaude adds value for specific workflows; run alongside custom system
- **REDUNDANT** - Duplicates the custom system with no added value; uninstall
- **REVISIT** - Has potential but not ready; check back at [milestone/date]

---

## If Verdict is REDUNDANT

```bash
# Uninstall
rm $(which multiclaude)
# Remove any config files it created
# Document why in .claude/docs/multiclaude-evaluation.md
```

---

## Notes on Gas Town

The Multiclaude maintainers document a comparison between Multiclaude and Gas Town (another orchestrator). Gas Town is more complex, better suited for solo devs on hobby projects. For a multi-dev team setup, Multiclaude is the better starting point for evaluation.