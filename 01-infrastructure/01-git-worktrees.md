# 01 - Git Worktrees

Run 2-4 parallel Claude Code sessions simultaneously without context switching, branch conflicts, or lost work. Each worktree is an independent checkout on its own branch.

---

## Prerequisites

- Claude Code installed
- Git repo initialized
- `claude --version` reports a recent build

---

## Why Worktrees

Without worktrees, switching tasks means stashing, checking out a new branch, losing your terminal state, and context-switching Claude mid-task. With worktrees, each task has its own directory, its own branch, and its own Claude session running concurrently.

A two-developer team running 2-4 worktrees each means 4-8 parallel streams of work. This only works reliably with the infrastructure below.

---

## Setup

### 1. Audit for untracked config files

Before creating worktrees, find files that are gitignored but required to run the app (`.env`, `.env.local`, secrets, local config). These need to be copied to each new worktree.

```bash
git ls-files --others --exclude-standard
# also check:
find . -name ".env*" -not -path "./.git/*"
```

List every file found. These go in `.worktreeinclude`.

### 2. Create `.worktreeinclude`

In the repo root, create `.worktreeinclude` using `.gitignore` syntax. List the untracked files that should be copied when a new worktree is created.

```
# Files to copy into new worktrees (gitignored but required to run the app)
.env.local
.env.development
config/local.json
```

If no untracked config files exist, create the file anyway with a comment explaining its purpose.

### 3. Gitignore worktree directory

Add this to `.gitignore` to prevent worktree contents from appearing as untracked files:

```
.claude/worktrees/
```

### 4. Create the launcher script

Create `.claude/scripts/tw.sh`:

```bash
#!/usr/bin/env bash
# Worktree launcher - usage: tw <task-name>
# Variants: tw-auto (long-running, uses --channels), tw-sched (headless scheduled)

set -e

TASK_NAME="${1:?Usage: tw <task-name>}"
WORKTREE_DIR=".claude/worktrees/$TASK_NAME"
BRANCH="worktree/$(git config user.initials 2>/dev/null || echo dev)/$TASK_NAME"

# tw: standard interactive session
tw() {
  git worktree add "$WORKTREE_DIR" -b "$BRANCH"
  # Copy files listed in .worktreeinclude
  if [ -f .worktreeinclude ]; then
    rsync -av --include-from=.worktreeinclude --exclude='*' . "$WORKTREE_DIR/" 2>/dev/null || true
  fi
  cd "$WORKTREE_DIR"
  claude
}

# tw-auto: long-running autonomous session with permission relay
tw_auto() {
  git worktree add "$WORKTREE_DIR" -b "$BRANCH"
  if [ -f .worktreeinclude ]; then
    rsync -av --include-from=.worktreeinclude --exclude='*' . "$WORKTREE_DIR/" 2>/dev/null || true
  fi
  cd "$WORKTREE_DIR"
  claude --channels
}

# tw-sched: headless scheduled session
tw_sched() {
  local PROMPT="${2:?Usage: tw-sched <task-name> <prompt>}"
  git worktree add "$WORKTREE_DIR" -b "$BRANCH"
  cd "$WORKTREE_DIR"
  claude --channels -p "$PROMPT"
}

case "$0" in
  *tw-auto) tw_auto "$@" ;;
  *tw-sched) tw_sched "$@" ;;
  *) tw "$@" ;;
esac
```

Make it executable and create symlinks for variants:

```bash
chmod +x .claude/scripts/tw.sh
ln -s tw.sh .claude/scripts/tw
ln -s tw.sh .claude/scripts/tw-auto
ln -s tw.sh .claude/scripts/tw-sched
```

Source from your shell profile:

```bash
export PATH="$PATH:/path/to/repo/.claude/scripts"
```

### 5. Document conventions in CLAUDE.md

Add a `## Worktrees` section to your `CLAUDE.md`:

```markdown
## Worktrees

**Naming:** `worktree/<initials>/<task-name>` (e.g. `worktree/jd/auth-refactor`)
**Max concurrent per dev:** 4
**Directory:** `.claude/worktrees/<task-name>/`

**Rules:**
- Each worktree targets independent files/modules to minimize merge conflicts
- Always commit or stash before switching context; never leave uncommitted changes
- Worktrees with no changes auto-clean on session end
- Worktrees with changes persist for review

**Launchers:**
- `tw <task>` - standard interactive session
- `tw-auto <task>` - long-running autonomous (adds --channels)
- `tw-sched <task> "<prompt>"` - headless scheduled session
```

---

## Resource Considerations

Each parallel Claude Code session consumes RAM and CPU. Before running 4+ parallel sessions on a shared server, check available headroom:

```bash
free -h        # RAM
uptime         # CPU load average
```

On a 2-vCPU / 8GB RAM server already running other services, 2-3 concurrent worktree sessions is a safe starting point. Monitor and adjust.

---

## Verification

```bash
# Create a test worktree
tw test-worktree-setup

# Verify it was created
ls .claude/worktrees/

# Verify branch exists
git branch | grep test-worktree-setup

# Clean up
git worktree remove .claude/worktrees/test-worktree-setup
git branch -d worktree/dev/test-worktree-setup
```