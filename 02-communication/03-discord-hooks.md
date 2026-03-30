# 03 - Discord Hooks

Connect Claude Code events to Discord so the whole team sees what's happening without watching terminals. Four event types post automatically: PR activity, review results, approval requests, and merge completions.

---

## Prerequisites

- Discord server with admin access (Server Settings → Integrations → Webhooks)
- Claude Code settings file at `.claude/settings.json` in the repo (shared) and `~/.claude/settings.json` (personal)
- `jq` and `curl` installed on dev machines and VPS

---

## Architecture

**Webhook URLs are stored as env vars.** This means you can remap any event to a different Discord channel just by swapping the URL - no code changes.

**Team hooks live in the repo** (`.claude/settings.json`). Every dev gets them on pull.

**Personal hooks live in `~/.claude/settings.json`**. Each dev customizes their own desktop notifications independently.

---

## Step 1: Create Discord Webhooks

In your Discord server:
1. Server Settings → Integrations → Webhooks → New Webhook
2. Name it after the event type (e.g. "PR Feed", "Reviews", "Approvals", "Merges", "Scheduled")
3. Choose the target channel
4. Copy the webhook URL

Repeat for each of the 4-5 event types you want.

---

## Step 2: Set Webhook Env Vars

Add to `~/.claude/settings.json` under the `"env"` key (do this on each dev machine and the VPS):

```json
{
  "env": {
    "DISCORD_WEBHOOK_PR": "https://discord.com/api/webhooks/YOUR_PR_WEBHOOK",
    "DISCORD_WEBHOOK_REVIEWS": "https://discord.com/api/webhooks/YOUR_REVIEWS_WEBHOOK",
    "DISCORD_WEBHOOK_APPROVALS": "https://discord.com/api/webhooks/YOUR_APPROVALS_WEBHOOK",
    "DISCORD_WEBHOOK_MERGES": "https://discord.com/api/webhooks/YOUR_MERGES_WEBHOOK",
    "DISCORD_WEBHOOK_SCHEDULED": "https://discord.com/api/webhooks/YOUR_SCHEDULED_WEBHOOK"
  }
}
```

Set values to `"PLACEHOLDER_URL"` until you create the real webhooks. The formatter script handles this gracefully.

---

## Step 3: Create the Formatter Script

Create `.claude/hooks/discord-format.sh`:

```bash
#!/usr/bin/env bash
# Reads Claude Code hook JSON from stdin, formats and posts to Discord
# Usage: echo "$HOOK_DATA" | discord-format.sh <WEBHOOK_URL>

WEBHOOK_URL="${1}"

if [ -z "$WEBHOOK_URL" ] || [ "$WEBHOOK_URL" = "PLACEHOLDER_URL" ]; then
  echo "[discord-format] No webhook URL set; skipping Discord notification" >&2
  exit 0
fi

INPUT=$(cat)
EVENT_TYPE=$(echo "$INPUT" | jq -r '.event_type // "unknown"')
SESSION_ID=$(echo "$INPUT" | jq -r '.session_id // "unknown"')

case "$EVENT_TYPE" in
  "stop")
    COLOR=3066993  # green
    TITLE="✅ Session Complete"
    DESCRIPTION=$(echo "$INPUT" | jq -r '.result // "Task completed"')
    ;;
  "notification")
    COLOR=15158332  # red
    TITLE="🔔 Needs Approval"
    DESCRIPTION=$(echo "$INPUT" | jq -r '.message // "Approval required"')
    ;;
  *)
    COLOR=3447003  # blue
    TITLE="📋 Claude Code Event"
    DESCRIPTION=$(echo "$INPUT" | jq -r '. | tostring')
    ;;
esac

PAYLOAD=$(jq -n \
  --arg title "$TITLE" \
  --arg desc "$DESCRIPTION" \
  --argjson color "$COLOR" \
  --arg session "$SESSION_ID" \
  '{
    embeds: [{
      title: $title,
      description: $desc,
      color: $color,
      footer: { text: ("Session: " + $session) }
    }]
  }')

curl -s -X POST "$WEBHOOK_URL" \
  -H "Content-Type: application/json" \
  -d "$PAYLOAD" > /dev/null
```

```bash
chmod +x .claude/hooks/discord-format.sh
```

---

## Step 4: Configure Team Hooks

Add to `.claude/settings.json` in the repo root:

```json
{
  "hooks": {
    "Stop": [
      {
        "command": "echo $CLAUDE_HOOK_DATA | .claude/hooks/discord-format.sh $DISCORD_WEBHOOK_PR"
      }
    ],
    "Notification": [
      {
        "command": "echo $CLAUDE_HOOK_DATA | .claude/hooks/discord-format.sh $DISCORD_WEBHOOK_APPROVALS"
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Bash",
        "command": "echo $CLAUDE_HOOK_DATA | grep -q 'git push' && echo $CLAUDE_HOOK_DATA | .claude/hooks/discord-format.sh $DISCORD_WEBHOOK_PR || true"
      },
      {
        "matcher": "Bash",
        "command": "echo $CLAUDE_HOOK_DATA | grep -qE 'git merge|gh pr merge' && echo $CLAUDE_HOOK_DATA | .claude/hooks/discord-format.sh $DISCORD_WEBHOOK_MERGES || true"
      }
    ]
  }
}
```

---

## Step 5: HTTP Hooks for VPS

For headless VPS sessions, use HTTP hooks instead. Create `.claude/hooks/discord-http-hook.json` as a reference template:

```json
{
  "hooks": {
    "Stop": [
      {
        "type": "http",
        "url": "$DISCORD_WEBHOOK_SCHEDULED",
        "method": "POST",
        "headers": { "Content-Type": "application/json" },
        "body": { "content": "Scheduled task complete: {{session_id}}" }
      }
    ]
  }
}
```

**Use shell hooks** for local dev machines where the script can run.
**Use HTTP hooks** for headless VPS sessions where shell execution is unreliable.

---

## Step 6: Personal Hooks Template

Create `.claude/hooks/personal-hooks-template.json` for each dev to copy to `~/.claude/settings.json`:

```json
{
  "hooks": {
    "Stop": [
      {
        "command": "osascript -e 'display notification \"Claude task complete\" with title \"Claude Code\" sound name \"Glass\"'"
      }
    ],
    "Notification": [
      {
        "command": "osascript -e 'display notification \"Approval needed\" with title \"Claude Code\" sound name \"Ping\"'"
      }
    ]
  }
}
```

Replace `osascript` with `notify-send` on Linux.

---

## Verification

1. Set one real webhook URL in `DISCORD_WEBHOOK_PR`
2. Start a Claude Code session and let it complete a simple task
3. Verify the Stop hook fires and a message appears in Discord
4. Test the Notification hook by triggering a permission prompt
5. Verify `PLACEHOLDER_URL` webhooks log a warning and don't crash

---

## CLAUDE.md Section to Add

```markdown
## Hooks & Notifications

All team-relevant Claude Code events post to Discord automatically.

| Event | Hook type | Discord channel (env var) |
|-------|-----------|--------------------------|
| Session complete | Stop | DISCORD_WEBHOOK_PR |
| Needs human approval | Notification | DISCORD_WEBHOOK_APPROVALS |
| git push detected | PostToolUse (Bash) | DISCORD_WEBHOOK_PR |
| git merge detected | PostToolUse (Bash) | DISCORD_WEBHOOK_MERGES |
| Scheduled task complete | Stop (VPS) | DISCORD_WEBHOOK_SCHEDULED |

**Team hooks:** `.claude/settings.json` in repo (shared via git)
**Personal hooks:** `~/.claude/settings.json` (per-dev, not committed)
**Shell hooks:** use on local dev machines
**HTTP hooks:** use for headless VPS sessions
```