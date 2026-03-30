# 09 - Scheduled Tasks

Four always-on background tasks running on a VPS, posting results to Discord. No dev at the terminal - fully autonomous. Read-only: they monitor and report, never make code changes.

---

## Prerequisites

- Claude Code installed on VPS (`npm install -g @anthropic-ai/claude-code`)
- Claude authenticated on VPS (run `claude` once to authenticate)
- Discord hooks configured (Guide 03) with all `DISCORD_WEBHOOK_*` vars in `~/.claude/settings.json` on the VPS
- `gh` CLI installed and authenticated on VPS
- Permission relay configured (Guide 04) for any prompts needing human input

---

## The Four Tasks

### 1. Morning PR Digest

**Schedule:** Daily at 7:00 AM (your timezone)
**Posts to:** `DISCORD_WEBHOOK_SCHEDULED`

Prompt:
```
Check all open PRs on the GitHub repo. For each PR, get: title, author, files changed count, age (hours since last activity), CI status, approval status. Rank by urgency: CI failures first, then oldest without review, then everything else. Format as a clean summary table. Post results to Discord. Do not make any code changes.
```

### 2. Stale PR Detection

**Schedule:** Every 4 hours
**Posts to:** `DISCORD_WEBHOOK_APPROVALS`

Prompt:
```
Check all open PRs. Flag any PR that:
- Has been open 24+ hours without activity (no new commits, comments, or review)
- Has a pending review request older than 8 hours
- Has unaddressed change requests older than 12 hours

For each flagged PR: check for existing stale labels (stale-1, stale-2, stale-3). Add the next stale label. If already at stale-3, include escalation: "This PR has been stale for 3 cycles; consider closing or prioritizing." Post flagged PRs with reasons to Discord. Do not make any code changes.
```

### 3. Weekly Security Audit

**Schedule:** Weekly on Monday at 6:00 AM
**Posts to:** `DISCORD_WEBHOOK_SCHEDULED` (critical findings also to `DISCORD_WEBHOOK_APPROVALS`)

Prompt:
```
Run a full dependency and security audit. Check for known vulnerabilities using the project's package manager audit command. Check for outdated packages. Categorize findings:
- CRITICAL: known exploits, auth-related packages, immediate action needed
- WARNING: outdated packages with available security patches
- INFO: outdated packages with no known security issues

Post results to Discord. If any CRITICAL items found, also post to the approvals channel as urgent. Do not make any code changes.
```

### 4. CI Health Check

**Schedule:** Every 30 minutes during active hours (e.g. 8 AM - 10 PM)
**Posts to:** `DISCORD_WEBHOOK_APPROVALS` (only on failures)

Prompt:
```
Check CI status for the main branch and all open PR branches. If main is failing: alert immediately with failure details. For PR branches: check if auto-fix is already handling failures (look for auto-fix-1, auto-fix-2, auto-fix-3 labels). Only alert for failures not being handled or that have exhausted auto-fix attempts. If everything is green, output nothing. Do not make any code changes.
```

---

## Scheduler Infrastructure

Create `.claude/scripts/scheduler.sh`:

```bash
#!/usr/bin/env bash
# Manages scheduled Claude Code tasks on VPS
# Usage: scheduler.sh <task-name> | scheduler.sh run <task-name> | scheduler.sh status

set -e

LOG_DIR="/var/log/claude-scheduler"
mkdir -p "$LOG_DIR"

declare -A PROMPTS
PROMPTS[morning-digest]="Check all open PRs on the GitHub repo. For each PR, get: title, author, files changed count, age, CI status, approval status. Rank by urgency: CI failures first, then oldest without review. Format as a summary table and post results. Do not make any code changes."
PROMPTS[stale-pr]="Check all open PRs. Flag any PR open 24+ hours without activity, any with pending review request older than 8 hours, any with unaddressed change requests older than 12 hours. Add appropriate stale label. Post flagged PRs with reasons. Do not make any code changes."
PROMPTS[security-audit]="Run a full dependency and security audit. Check for vulnerabilities via package manager audit. Check for outdated packages. Categorize as CRITICAL/WARNING/INFO. Post results. Flag critical items as urgent. Do not make any code changes."
PROMPTS[ci-health]="Check CI status for main and all open PR branches. Alert only for failures not being handled by auto-fix or that exhausted auto-fix attempts. If everything is green, output nothing. Do not make any code changes."

run_task() {
  local TASK_NAME="$1"
  local PROMPT="${PROMPTS[$TASK_NAME]}"
  local LOG="$LOG_DIR/${TASK_NAME}-$(date +%Y%m%d-%H%M%S).log"

  if [ -z "$PROMPT" ]; then
    echo "Unknown task: $TASK_NAME" >&2
    exit 1
  fi

  bash "$(dirname "$0")/resource-guard.sh" "$TASK_NAME" || exit 0

  echo "[$(date)] Starting $TASK_NAME" | tee -a "$LOG"
  WORKTREE="sched-$TASK_NAME-$(date +%s)"
  claude --worktree "$WORKTREE" --channels -p "$PROMPT" >> "$LOG" 2>&1
  echo "[$(date)] Completed $TASK_NAME" | tee -a "$LOG"
}

case "$1" in
  run) run_task "$2" ;;
  status)
    echo "=== Scheduled Tasks Status ==="
    for task in morning-digest stale-pr security-audit ci-health; do
      LAST=$(ls "$LOG_DIR/${task}-"*.log 2>/dev/null | tail -1)
      if [ -n "$LAST" ]; then
        echo "$task: last run $(basename "$LAST" .log | cut -d- -f3-)"
      else
        echo "$task: never run"
      fi
    done
    ;;
  *) run_task "$1" ;;
esac
```

### Resource Guard

Create `.claude/scripts/resource-guard.sh`:

```bash
#!/usr/bin/env bash
# Checks resource headroom before launching a scheduled task
# Exit 1 to skip; exit 0 to proceed

TASK_NAME="${1:-unknown}"
LOG_DIR="/var/log/claude-scheduler"

AVAIL_MB=$(free -m | awk '/^Mem:/{print $7}')
if [ "$AVAIL_MB" -lt 1024 ]; then
  MSG="Skipped $TASK_NAME: only ${AVAIL_MB}MB RAM available"
  echo "$MSG"
  curl -s -X POST "$DISCORD_WEBHOOK_SCHEDULED" \
    -H "Content-Type: application/json" \
    -d "{\"content\": \"$MSG\"}" > /dev/null
  exit 1
fi

LOAD=$(uptime | awk -F'load average:' '{print $2}' | cut -d, -f1 | tr -d ' ')
if (( $(echo "$LOAD > 1.5" | bc -l) )); then
  MSG="Skipped $TASK_NAME: CPU load ${LOAD}"
  echo "$MSG"
  curl -s -X POST "$DISCORD_WEBHOOK_SCHEDULED" \
    -H "Content-Type: application/json" \
    -d "{\"content\": \"$MSG\"}" > /dev/null
  exit 1
fi

if pgrep -f "claude.*sched-" > /dev/null; then
  echo "Skipped $TASK_NAME: another scheduled task is running"
  exit 1
fi

exit 0
```

---

## Cron Setup

SSH into VPS and run `crontab -e`:

```cron
# Claude Code scheduled tasks
0 7 * * *        /path/to/repo/.claude/scripts/scheduler.sh morning-digest
0 */4 * * *      /path/to/repo/.claude/scripts/scheduler.sh stale-pr
0 6 * * 1        /path/to/repo/.claude/scripts/scheduler.sh security-audit
*/30 8-22 * * *  /path/to/repo/.claude/scripts/scheduler.sh ci-health
```

---

## Manual Trigger

```bash
# Run from dev machine via SSH
ssh your-vps 'scheduler.sh run morning-digest'

# Check status
ssh your-vps 'scheduler.sh status'
```

---

## CLAUDE.md Section to Add

```markdown
## Scheduled Tasks

All scheduled tasks run on VPS. All are read-only - they report, never fix.

| Task | Schedule | Posts to |
|------|----------|---------|
| Morning PR digest | Daily 7 AM | DISCORD_WEBHOOK_SCHEDULED |
| Stale PR detection | Every 4h | DISCORD_WEBHOOK_APPROVALS |
| Security audit | Mon 6 AM | DISCORD_WEBHOOK_SCHEDULED + APPROVALS (critical) |
| CI health check | Every 30m, 8AM-10PM | DISCORD_WEBHOOK_APPROVALS (failures only) |

Resource guard: skips if <1GB RAM or CPU load >1.5. Max 1 scheduled session at a time.
Logs: /var/log/claude-scheduler/
Manual trigger: scheduler.sh run <task-name>
```
