<div align="center">

<a href="https://arc-web.github.io/claude-code-playbook/">
  <img src="https://img.shields.io/badge/🎬_Interactive_Presentation-View_Live-7B2FBE?style=for-the-badge&labelColor=0F0F1A&color=7B2FBE" alt="View Interactive Presentation" />
</a>

</div>

---

# Claude Code Playbook

Generalized guides for running Claude Code as the orchestration kernel for a multi-developer team, with GitHub for version control and Discord as the communication terminal.

These guides are distilled from real production use. Each one is standalone - read them in sequence for a full setup, or jump directly to the topic you need.

---

## Reading Order

### Phase 1 - Infrastructure
| # | Guide | What it covers |
|---|-------|----------------|
| 1 | [Git Worktrees](01-infrastructure/01-git-worktrees.md) | Parallel development with isolated worktrees, launcher scripts, naming conventions |
| 2 | [Conflict Resolution](01-infrastructure/02-conflict-resolution.md) | Three-layer system: prevent, detect, and resolve merge conflicts with Claude |

### Phase 2 - Communication
| # | Guide | What it covers |
|---|-------|----------------|
| 3 | [Discord Hooks](02-communication/03-discord-hooks.md) | Webhook integration for PR events, reviews, approvals, and merges |
| 4 | [Permission Relay](02-communication/04-permission-relay.md) | Forwarding Claude approval prompts to phone and Discord via `--channels` |

### Phase 3 - Agents
| # | Guide | What it covers |
|---|-------|----------------|
| 5 | [Agent Teams](03-agents/05-agent-teams.md) | Parallel review agents (logic, tests, performance, security) with team lead synthesis |
| 6 | [Custom Subagents](03-agents/06-custom-subagents.md) | Implementation and ops specialist agents with auto-invoke by keyword |
| 7 | [Multiclaude](03-agents/07-multiclaude.md) | Evaluating Multiclaude for multiplayer code review alongside your custom system |

### Phase 4 - Automation
| # | Guide | What it covers |
|---|-------|----------------|
| 8 | [Cloud Auto-Fix](04-automation/08-cloud-autofix.md) | GitHub Actions workflow to auto-fix CI failures and green review items |
| 9 | [Scheduled Tasks](04-automation/09-scheduled-tasks.md) | VPS cron jobs for morning digest, stale PR detection, security audits, CI monitoring |

### Phase 5 - Workflow
| # | Guide | What it covers |
|---|-------|----------------|
| 10 | [Plan Mode](05-workflow/10-plan-mode.md) | When Claude plans before acting, the plan format, and enforcement by session type |

### Phase 6 - Tooling
| # | Guide | What it covers |
|---|-------|----------------|
| 11 | [Session Management](06-tooling/11-session-management.md) | CCManager vs Claude Squad evaluation, Haiku auto-approval, session recovery |
| 12 | [Plugins](06-tooling/12-plugins.md) | Plugin ecosystem, context budget rules, overlap analysis, evaluation cycle |

---

## Prerequisites

Before starting:
- Claude Code installed and authenticated (`claude --version`)
- GitHub repo with branch protection on `main` (require 1 approval before merge)
- Discord server with admin access (to create webhooks)
- VPS or server for scheduled tasks (optional but recommended)
- `gh` CLI installed and authenticated

---

## Contributing

These guides are intentionally generic. If you have a process that works well for your Claude Code + GitHub + Discord setup, open a PR. Keep it generalized - no project-specific names, domains, or stack assumptions.
