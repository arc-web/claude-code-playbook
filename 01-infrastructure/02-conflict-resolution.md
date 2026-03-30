# 02 - Conflict Resolution

A three-layer system to handle merge conflicts on teams with no strict file ownership and high parallel branch activity.

**Layer 1 - Prevention:** Active awareness of what each branch is touching before you start work.
**Layer 2 - Detection:** Test for conflicts before merge, not after.
**Layer 3 - Resolution:** Auto-resolve simple conflicts; generate structured cards for complex ones.

---

## Prerequisites

- Git worktrees set up (Guide 01)
- `CLAUDE.md` exists in the repo root
- Discord hooks configured (Guide 03) - needed for conflict card notifications

---

## Layer 1: Prevention

### /branches command

Create `.claude/commands/branches.md`:

```markdown
Show all active branches and their file overlap. Run: `git fetch origin` first, then for each local and remote branch, run `git diff --name-only main..<branch>`. Output a table:

| Branch | Last commit | Dev | Files changed | Collision warning |
|--------|-------------|-----|---------------|-------------------|

Highlight any file that appears in more than one active branch with a ⚠️ warning. At the bottom, recommend: if you are about to work on a file another active branch is touching, coordinate first or choose different files.
```

### SessionStart hook

Create `.claude/hooks/pre-work-check.sh`:

```bash
#!/usr/bin/env bash
# Runs on every session start
# Fetches latest, checks if branch is behind main, warns on file collisions

git fetch origin --quiet

# Check if behind main
BEHIND=$(git rev-list --count HEAD..origin/main 2>/dev/null || echo 0)
if [ "$BEHIND" -gt 0 ]; then
  echo "⚠️  Your branch is $BEHIND commits behind main. Consider rebasing before starting work."
fi

# Check git identity
NAME=$(git config user.name)
EMAIL=$(git config user.email)
if [ -z "$NAME" ] || [ -z "$EMAIL" ]; then
  echo "⚠️  Git identity not configured. Run:"
  echo "  git config --global user.name \"Your Name\""
  echo "  git config --global user.email \"you@example.com\""
fi
```

Register in `.claude/settings.json`:

```json
{
  "hooks": {
    "SessionStart": [
      { "command": "bash .claude/hooks/pre-work-check.sh" }
    ]
  }
}
```

---

## Layer 2: Detection

### /conflict-check command

Create `.claude/commands/conflict-check.md`:

```markdown
Test if the current branch would conflict with main before merging.

1. Run `git merge --no-commit --no-ff origin/main` to simulate the merge
2. If clean: output "✅ No conflicts with main. Safe to merge."
3. If conflicts: list every conflicting file with the conflict type (both modified / deleted vs modified)
4. Always abort: `git merge --abort`
5. Also test against all other open PR branches using `gh pr list --json headRefName` and repeat for each

Output a full conflict report. This command is read-only - it never commits anything.
```

Add to `CLAUDE.md`:

```markdown
**Rule:** Run /conflict-check before approving any PR.
```

---

## Layer 3: Resolution

### /resolve command

Create `.claude/commands/resolve.md`:

```markdown
Resolve merge conflicts in the current branch. Parse all conflicting files from `git diff --name-only --diff-filter=U`.

**Auto-resolvable (fix automatically):**
- Import ordering conflicts (both branches added different imports)
- Non-overlapping changes in the same file
- Whitespace or formatting-only conflicts
- Package lock file conflicts (delete and regenerate: `npm install` or equivalent)
- Auto-generated files (rebuild from source)
- Changelog or version bump conflicts (combine both entries)

**Requires human decision (generate a conflict card):**
- Same function or code block modified differently in both branches
- One branch deleted code the other branch modified
- Structural conflicts (file moved in one branch, modified in another)
- Any conflict in critical path files (define in CLAUDE.md under `## Critical Paths`)

For auto-resolvable: fix, stage, and report what was done.

For human-required: generate a conflict card for each conflict in this format and post to Discord (DISCORD_WEBHOOK_REVIEWS):

```
┌─────────────────────────────────────────────────────┐
│ ⚔️ MERGE CONFLICT | [filename]                       │
│ [branch-ours] vs [branch-theirs]                     │
│─────────────────────────────────────────────────────│
│ OURS (current branch):                               │
│ [3-5 lines of relevant code]                         │
│─────────────────────────────────────────────────────│
│ THEIRS (incoming branch):                            │
│ [3-5 lines of relevant code]                         │
│─────────────────────────────────────────────────────│
│ [A] Keep ours                                        │
│ [B] Keep theirs                                      │
│ [C] Merge both - [proposed combined version]         │
│ [D] Skip; I'll handle manually                       │
│ 📁 [full path + line range]                          │
└─────────────────────────────────────────────────────┘
```

Reply format: `do A for file1, B for file2, C for file3`
Apply the selected resolutions.
```

---

## Git Identity Enforcement

Every dev must configure their git identity on their machine. This is how commits are attributed regardless of shared service accounts.

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

---

## CLAUDE.md Section to Add

```markdown
## Critical Paths

Files in these paths always require human decision for conflict resolution and trigger agent team escalation for reviews:
- src/auth/
- src/core/
- src/api/
- Any file containing: payment, security, middleware

**Rule:** Run /conflict-check before approving any PR.
**Rule:** Lockfile and auto-generated file conflicts always auto-resolve.
**Rule:** Critical path file conflicts always require human decision.
**Rule:** Git identity must be configured per machine; SessionStart hook enforces this.
```
