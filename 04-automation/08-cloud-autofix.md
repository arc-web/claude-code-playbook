# 08 - Cloud Auto-Fix

GitHub Actions workflow that automatically fixes unambiguous CI failures and green review items, so devs don't manually address every lint error or simple reviewer comment.

---

## Prerequisites

- Claude GitHub App installed (`/install-github-app` in Claude Code terminal)
- GitHub repo with branch protection on `main` (require 1 approval)
- Agent Teams configured (Guide 05) for the green/yellow/red tier system
- Discord hooks configured (Guide 03)

---

## Install the Claude GitHub App

In a Claude Code terminal:
```
/install-github-app
```

This installs the Claude GitHub App on your repo and configures the required secrets. Verify it has Read & Write access to: Contents, Issues, Pull Requests.

---

## GitHub Actions Workflow

Create `.github/workflows/claude-autofix.yml`:

```yaml
name: Claude Auto-Fix

on:
  pull_request:
    types: [opened, synchronize]
  issue_comment:
    types: [created]
  check_suite:
    types: [completed]

jobs:
  auto-fix:
    runs-on: ubuntu-latest
    if: |
      (github.event_name == 'check_suite' && github.event.check_suite.conclusion == 'failure') ||
      (github.event_name == 'issue_comment' && contains(github.event.comment.body, '@claude'))

    steps:
      - uses: actions/checkout@v4

      - name: Run Claude Auto-Fix
        uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          prompt: |
            You are reviewing a CI failure or responding to a @claude comment.

            For CI failures:
            - Read the failure log
            - Identify the root cause
            - If fix is unambiguous (missing import, type error, lint failure, obvious test assertion): apply the fix, commit, push to PR branch
            - If fix requires judgment: post a yellow or red review card as a PR comment
            - After any auto-fix: post a PR comment summarizing what changed and why

            For @claude comments:
            - Parse what's being requested
            - If simple directive: execute, commit, push
            - If ambiguous: reply asking for clarification
            - Never auto-merge
          github_token: ${{ secrets.GITHUB_TOKEN }}
```

---

## Safeguard Rules

Add to `CLAUDE.md` and encode in the workflow:

### Never Auto-Fix
- Anything in auth, security, or access control files
- Database schemas or migrations
- Environment variables or secrets files
- CI/CD pipeline configuration
- File deletions or functionality removal
- Any fix touching more than 3 files simultaneously

### Always Auto-Fix
- Missing imports
- Type errors with one obvious resolution
- Lint/format failures
- Test snapshot updates where expected value is clearly wrong
- Simple null checks flagged by review agent

### Gray Area - Post Card, Let Human Decide
- Anything not in the above two lists

---

## Green Auto-Apply Command

Create `.claude/commands/auto-apply-green.md`:

```markdown
After the review team lead produces card output, parse it for green RECOMMENDED items.

For each green item:
- Execute the recommended fix
- Stage the change

After all green items are applied:
- Commit with message: "auto-fix: [N] recommended items from review"
- Push to the PR branch
- Post to Discord (DISCORD_WEBHOOK_REVIEWS): list which items were auto-applied
- Post remaining yellow and red cards to Discord for human decision

If zero green items: skip auto-apply, post all cards for human review.
```

---

## Runaway Loop Prevention

Track auto-fix attempts via PR labels. Add this logic to the workflow:

1. On each auto-fix attempt: add label `auto-fix-1`, `auto-fix-2`, or `auto-fix-3`
2. If `auto-fix-3` label already exists: stop all automation, add `needs-human` label
3. Post a red card to Discord: "Auto-fix exhausted after 3 attempts. Human intervention required."

```yaml
- name: Check auto-fix attempt count
  run: |
    LABELS=$(gh pr view ${{ github.event.pull_request.number }} --json labels -q '.labels[].name')
    if echo "$LABELS" | grep -q "auto-fix-3"; then
      gh pr edit ${{ github.event.pull_request.number }} --add-label "needs-human"
      echo "SKIP_AUTOFIX=true" >> $GITHUB_ENV
    fi
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

---

## CLAUDE.md Section to Add

```markdown
## Cloud Auto-Fix

Triggers: CI failure, @claude PR comment, green review items.

Never auto-fix: auth/security files, migrations, env vars, CI config, deletions, >3 files.
Always auto-fix: missing imports, type errors, lint failures, snapshot updates, simple null checks.
Gray area: post as card, let human decide.

Max 3 auto-fix attempts per PR. After 3: adds "needs-human" label and stops.
Auto-fix always posts to Discord. Auto-fix never merges - only commits and pushes.
Green items auto-apply. Yellow and red items require human selection.
```
