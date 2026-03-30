# 04 - Permission Relay

When Claude needs human approval during a long-running or autonomous session and you're not watching the terminal, `--channels` forwards the approval prompt to your phone and Discord so you can respond asynchronously.

---

## Prerequisites

- Claude Code v2.1.71 or later (`claude --version` to check; `npm update -g @anthropic-ai/claude-code` to update)
- Discord hooks configured (Guide 03) with `DISCORD_WEBHOOK_APPROVALS` set
- Worktree launcher scripts from Guide 01

---

## When to Use --channels

Not every session needs `--channels`. Use it only when:
- The session is long-running or autonomous (you won't be watching the terminal)
- The session runs on a VPS or headless server
- The task involves multiple sequential decisions where you may step away

**Don't use it for:** standard interactive sessions where you're at the terminal.

---

## Session Types

The `tw.sh` launcher (Guide 01) defines three session types:

| Launcher | `--channels` | Use case |
|----------|-------------|----------|
| `tw <task>` | No | Standard interactive - you're at the terminal |
| `tw-auto <task>` | Yes | Long-running autonomous - you may step away |
| `tw-sched <task> "<prompt>"` | Yes | Headless scheduled - no terminal at all |

---

## How --channels Works

`--channels` integrates with the Claude mobile app or notification system to forward approval prompts. When Claude needs human input:
1. The session pauses
2. The prompt is forwarded to your phone via the Claude app
3. Your `Notification` hook (Guide 03) simultaneously posts to Discord
4. You approve or reject from either interface
5. The session resumes

**Research note:** `--channels` was added in March 2026. If `claude --help` doesn't list it, update Claude Code. If mobile app pairing is required, follow the in-app setup flow. Document your specific setup steps here once confirmed.

---

## Dual-Destination Routing

Approvals land on both phone (via `--channels`) and Discord (via the Notification hook). Two scenarios:

**Path A (preferred):** `--channels` handles phone; Notification hook handles Discord independently. Both fire on the same event. No extra config needed.

**Path B (fallback):** If `--channels` intercepts the Notification event and prevents hooks from firing, add an HTTP hook that posts to Discord independently of the `--channels` flow. See Guide 03 for HTTP hook format.

Test which path applies for your setup and document it here.

---

## VPS / Headless Limitations

`--channels` may require a mobile app pairing that doesn't work on headless servers. If so:

1. Haiku auto-approval handles safe prompts automatically (see Guide 11)
2. For prompts Haiku escalates to human: they post to Discord via HTTP hook
3. A human responds from Discord or SSHes into the VPS to approve manually

Document the limitation explicitly in your `CLAUDE.md` so all devs know the fallback path.

---

## CLAUDE.md Section to Add

```markdown
## Session Types & Permission Relay

| Launcher | --channels | Use case |
|----------|-----------|----------|
| tw | No | Standard interactive |
| tw-auto | Yes | Long-running autonomous |
| tw-sched | Yes | Headless scheduled |

Approvals go to both phone (--channels) and Discord (Notification hook).
Haiku auto-approval handles safe prompts; --channels handles what Haiku escalates.

**VPS sessions:** [document your specific setup here after testing]
```

---

## Verification

```bash
# Start an auto session
tw-auto test-channels

# Trigger a permission prompt (e.g. ask Claude to run a command requiring approval)
# Verify it arrives on phone
# Verify it posts to DISCORD_WEBHOOK_APPROVALS
# Approve from phone; verify session resumes

# Start a standard session - verify --channels is NOT used
tw test-no-channels
```
