# 12 - Plugins

Plugins extend Claude Code with additional capabilities. The risk: each plugin consumes context window space, and too many plugins degrades performance. Treat plugins as install-and-evaluate, not install-and-forget.

---

## Context Budget Rule

**Plugins should not exceed 15% of your available context window.**

Before installing any plugin:
1. Start a fresh Claude Code session
2. Run `/context` to see baseline usage
3. Record the number

After installing:
1. Run `/context` again
2. Calculate overhead: (new usage) - (baseline)
3. If total plugin overhead exceeds 15%: remove the highest-cost plugin

---

## Recommended Plugins

### Context7

Keeps Claude current on library and framework documentation. Without it, Claude may reference outdated API calls from training data.

```bash
# Install (verify exact command with /plugin help)
/plugin marketplace add context7
```

Test it: ask Claude about a library in your stack and verify it references current docs rather than training-cutoff versions.

### Code Review Plugin

Adds review capabilities. **Important:** evaluate overlap with your custom review agents (Guide 05) before keeping.

After installing, compare its output against your custom `/review` command on the same PR. Keep whichever produces better findings; remove the other. Duplicate review agents inflate context with no benefit.

---

## Overlap Analysis

After installing each plugin, build this matrix:

| Capability | Custom Agent | Plugin | Redundant? | Keep Which? |
|-----------|-------------|--------|------------|-------------|
| Security review | security-reviewer.md | Code Review plugin | ? | ? |
| Test coverage | test-reviewer.md | Code Review plugin | ? | ? |
| Logic validation | logic-reviewer.md | Code Review plugin | ? | ? |
| Live docs | (none) | Context7 | No | Both |
| Coding standards | CLAUDE.md rules | dev-skills | ? | ? |

For each redundancy:
- Custom agent better: remove plugin capability (or entire plugin if that's all it does)
- Plugin better: remove custom agent
- Complementary: keep both
- Exact duplicate: remove the one with higher context cost

---

## Plugin Evaluation Scorecard

Create `.claude/docs/plugin-evaluation.md`. Fill in after 1 week of real use:

| Plugin | Usefulness (1-5) | Context cost (1-5) | Overlap (1-5) | Reliability (1-5) | Verdict |
|--------|-----------------|-------------------|---------------|-------------------|---------|
| Context7 | | | | | |
| Code Review | | | | | |
| [other] | | | | | |

Scoring guide:
- **Usefulness:** Does it add value beyond custom agents? (1=duplicates everything, 5=fills real gaps)
- **Context cost:** How much context does it consume? (1=very heavy, 5=very light)
- **Overlap:** How much does it duplicate custom work? (1=total overlap, 5=no overlap)
- **Reliability:** Did it work correctly in testing? (1=broken, 5=flawless)
- **Verdict:** KEEP / REMOVE / REVISIT IN 1 WEEK

---

## Plugin Management

```bash
# List installed plugins
/plugin list

# Install a plugin
/plugin marketplace add <plugin-name>

# Uninstall a plugin
/plugin remove <plugin-name>

# Check context usage
/context
```

If a plugin breaks something: uninstall immediately, document what happened in `.claude/docs/plugin-evaluation.md`.

---

## Rules

- Check context usage before and after every new plugin install
- Install one plugin at a time; evaluate individually before adding the next
- Custom agents always take priority over plugin equivalents when there's overlap
- Any dev can remove a plugin that's causing issues - just update the evaluation doc
- `dev-skills` and `claude-code-workflows` (shinpr) share skills; installing both causes duplicate skill descriptions. If you install both, check for duplicates and remove `dev-skills` if conflicts arise.

---

## CLAUDE.md Section to Add

```markdown
## Plugins

Installed plugins: [list after evaluation]
Context budget: plugins must not exceed 15% of available context window.
Custom agents take priority over plugin equivalents for any overlapping capability.
Evaluation scorecard: .claude/docs/plugin-evaluation.md
```