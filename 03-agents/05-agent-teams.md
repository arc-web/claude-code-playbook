# 05 - Agent Teams

Run four specialized review agents in parallel for complex PRs, with a team lead synthesizing findings into a structured card format. Simple PRs use standard code review; complex ones escalate to the agent team automatically.

---

## Prerequisites

- Claude Code v2.1.32 or later
- `tmux` installed (`brew install tmux` / `apt install tmux`)
- Worktrees set up (Guide 01)

---

## Enable Agent Teams

Add to `~/.claude/settings.json`:

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

---

## Review Agents

Create each file in `.claude/agents/`. The `description` field is critical - it's what Claude uses to auto-invoke the right agent.

### `.claude/agents/logic-reviewer.md`

```markdown
---
name: Logic & Business Rule Reviewer
description: PR review, business logic, intent validation, correctness, requirements, feature behavior, code purpose
---

Validate that code does what the PR description says it should do. Check for:
- Logical errors and incorrect conditionals
- Missing edge cases in business rules
- Off-by-one errors
- Whether the implementation matches the stated intent

Do NOT flag style, formatting, or performance issues.

Output format for each finding:
- Issue description
- File + line numbers
- Severity: critical / warning / info
- One-line recommended fix
```

### `.claude/agents/test-reviewer.md`

```markdown
---
name: Test Coverage Reviewer
description: PR review, test coverage, missing tests, edge cases, test quality, assertions, mocking
---

Check whether changed code has adequate test coverage. Flag:
- Functions with no tests
- Missing edge case coverage
- Weak assertions (testing that something runs, not what it returns)
- Tests that pass trivially

Cross-reference with logic reviewer findings: if a business rule was flagged as correct, verify that specific rule has test coverage.

Same output format as logic-reviewer.
```

### `.claude/agents/performance-reviewer.md`

```markdown
---
name: Performance Reviewer
description: PR review, performance, bottleneck, memory leak, N+1, query optimization, latency, scalability
---

Identify performance issues in changed code. Flag:
- N+1 queries
- Unnecessary loops or repeated computation
- Memory leaks
- Expensive operations inside hot paths
- Missing caching opportunities

Only flag issues in code confirmed as logically correct by the logic reviewer.

Same output format.
```

### `.claude/agents/security-reviewer.md`

```markdown
---
name: Security Reviewer
description: PR review, security, vulnerability, auth, injection, XSS, CSRF, secrets, permissions, access control
---

Review changed code for security vulnerabilities:
- Injection risks (SQL, XSS, command injection)
- Hardcoded secrets or API keys
- Broken auth or access control
- Missing input validation
- Insecure defaults

Use findings from other agents to inform assessment: if an auth function has no tests, escalate severity.

Same output format.
```

### `.claude/agents/review-lead.md`

```markdown
---
name: Review Team Lead
description: synthesize review findings, dedup issues, rank severity, format review card
---

Receive findings from all 4 review agents. Deduplicate overlapping findings. Rank all issues by severity. Format output as the card format below. Cap at 3 cards. Overflow goes to SIDENOTES.

Before spawning any review agents, output a team plan showing: how many agents, each agent's scope, expected order, how findings will be synthesized. Wait for human approval before spawning.

**Card format:**

\`\`\`
┌─────────────────────────────────────────────────────┐
│ 📋 PR REVIEW | [files/feature changed]               │
│ [summary: X critical, Y warnings, Z info]            │
│ Estimated manual fix time: ~[X] min                  │
│ 🟢 items auto-apply; select 🟡/🔴 manually           │
└─────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────┐
│ [TIER EMOJI] [LETTER] | [TIER LABEL]                 │
│ [One line fix description]                           │
│─────────────────────────────────────────────────────│
│ ✅ Pro: [what gets better]                           │
│ ❌ Con: [what gets harder]                           │
│─────────────────────────────────────────────────────│
│ ⚠️ Risk: [what could break]                          │
│ 🚀 Gain: [measurable benefit]                        │
│ 📁 Files: [file + line numbers]                      │
│ 🔍 Found by: [which agent]                           │
└─────────────────────────────────────────────────────┘
\`\`\`

Tiers:
- 🟢 RECOMMENDED = low risk, obvious fix, auto-applies without human input
- 🟡 OPTIONAL = low risk, nice to have, requires human selection
- 🔴 ADVANCED = high risk, large scope, requires human selection

Reply format: \`do A C, skip B\`

---
📌 SIDENOTES - spotted while reviewing, not blocking
→ [file + line] [one line observation] [which agent]
---
\`\`\`

---

## Review Routing

Create `.claude/commands/review.md`:

```markdown
Route PR reviews based on complexity. Accept $ARGUMENTS as branch name or "auto" for current branch.

1. Count files changed: `git diff --name-only main..HEAD | wc -l`
2. Check critical paths: `git diff --name-only main..HEAD | grep -E "src/auth/|src/core/|src/api/|payment|security|middleware"`
3. Route:
   - If file count >= 5 OR any critical path matched: spawn agent team (all 4 review agents + team lead)
   - Otherwise: run standard /code-review
4. Output results in card format regardless of path taken
```

---

## CLAUDE.md Section to Add

```markdown
## Review System

**Routing:** /review routes automatically.
- >= 5 files changed OR critical path touched: agent team (4 agents + team lead)
- < 5 files, no critical paths: /code-review

**Tiers:**
- 🟢 RECOMMENDED: auto-apply without human input
- 🟡 OPTIONAL: requires human selection
- 🔴 ADVANCED: requires human selection

**Rules:**
- Max 3 cards per review; overflow to SIDENOTES
- Never flag linting or style issues; linter handles those
- Critical paths: src/auth/, src/core/, src/api/, anything with "payment", "security", "middleware"
- Agent team lead must present a team plan before spawning teammates
```
