# 11 - Session Management

With 2-4 parallel worktrees per dev, you need a way to see all sessions at a glance, switch between them fast, and recover if a terminal crashes. Two tools to evaluate: CCManager and Claude Squad.

---

## Prerequisites

- Git worktree infrastructure (Guide 01)
- Session launchers (tw, tw-auto, tw-sched) in place

---

## The Two Tools

### CCManager

- GitHub: `github.com/kbwo/ccmanager`
- Self-contained TUI, no tmux dependency
- Key feature: context transfer - new worktrees can inherit conversation history from a parent session
- Key feature: built-in Haiku auto-approval for safe prompts
- Key feature: auto-directory creation from branch names
- Works on macOS and Linux

### Claude Squad

- GitHub: search "claude squad" CLI
- tmux-based (requires tmux)
- Advantage: session recovery - kill the terminal, reconnect, sessions still running
- Disadvantage: tmux dependency (resource overhead on VPS)

---

## Evaluation Criteria

Run both tools with 3 parallel worktrees and score each:

| Criteria | CCManager (1-5) | Claude Squad (1-5) | Notes |
|----------|----------------|-------------------|-------|
| Session visibility | | | Can you see all sessions at a glance? |
| Session switching | | | How fast? How many keystrokes? |
| Session creation | | | Integrates with tw scripts? |
| Context transfer | | | Can new worktree inherit parent context? |
| Auto-approval | | | Does Haiku auto-approve safe prompts? |
| Notification | | | Notified when session needs attention? |
| VPS compatibility | | | Works headless via SSH? |
| Recovery | | | Can you reconnect after terminal crash? |
| Install complexity | | | |
| Resource footprint | | | RAM/CPU overhead of the tool itself |
| Active maintenance | | | Last commit date, open issues |
| **TOTAL** | **/55** | **/55** | |

Save completed scorecard to `.claude/docs/session-manager-comparison.md`.

---

## Haiku Auto-Approval

Configure regardless of which tool you choose. Haiku auto-approves safe prompts so you don't have to manually approve every file read or git status check.

### Safe prompts (auto-approve)
- File reads, directory listings
- `git status`, `git log`, `git diff`
- Running tests
- Running linters
- Non-destructive commands

### Dangerous prompts (require manual approval)
- File writes to critical paths
- `git push`, `git merge`
- Any command with `sudo`
- Commands that delete files
- Docker/Nginx/infrastructure config changes

**CCManager config:** enable auto-approval in CCManager settings and define safe/dangerous rules.

**Without CCManager:** configure at Claude Code level via `~/.claude/settings.json`:

```json
{
  "permissions": {
    "allow": [
      "Bash(git status:*)",
      "Bash(git log:*)",
      "Bash(git diff:*)",
      "Read(*)"
    ],
    "deny": [
      "Bash(sudo:*)",
      "Bash(*rm -rf*)"
    ]
  }
}
```

---

## Integration with Launcher Scripts

Update `tw.sh` to register sessions with the winning tool.

**If CCManager wins:**
```bash
tw() {
  git worktree add "$WORKTREE_DIR" -b "$BRANCH"
  ccmanager start "$WORKTREE_DIR"
}
```

**If Claude Squad wins:**
```bash
tw() {
  git worktree add "$WORKTREE_DIR" -b "$BRANCH"
  cs new "$BRANCH"  # verify exact command from Claude Squad docs
}
```

---

## Session Recovery

**With tmux (Claude Squad):**
```bash
# Reconnect after terminal crash
tmux attach -t claude-sessions
```

**Without tmux (CCManager):**
- Sessions may not survive terminal crash
- Mitigation: commit frequently (every logical step), so worst case is losing recent uncommitted work

---

## CLAUDE.md Section to Add

```markdown
## Session Management

Tool: [winner after evaluation]
Launch all sessions through the session manager. Don't create orphan Claude sessions outside of it.
Check session overview before starting new work to avoid duplicating effort.

Auto-approval: Haiku handles safe prompts automatically. Manual approval required for: file writes to critical paths, git push/merge, sudo, deletes, infrastructure config.
```