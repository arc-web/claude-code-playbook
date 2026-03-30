# 06 - Custom Subagents

Specialists Claude auto-invokes based on task description keywords. Three categories: review agents (Guide 05), implementation agents (stack-specific), and ops agents (infrastructure).

---

## Prerequisites

- `.claude/agents/` directory exists (created in Guide 05)
- `CLAUDE.md` with architecture notes so agents know the tech stack

---

## How Auto-Invoke Works

Claude reads each agent's `description` field and selects the matching agent when a task contains those keywords. The `description` is the most important field - bad keywords mean the wrong agent (or no agent) gets selected.

**Good description:** specific trigger terms that appear in natural task language
```
description: Docker, container, Compose, docker-compose, image, Dockerfile, container health
```

**Bad description:** generic terms that match everything
```
description: code, files, implementation, help
```

---

## Agent File Format

Every agent in `.claude/agents/` uses this format:

```markdown
---
name: [Human-readable name]
description: [comma-separated trigger keywords]
---

[Instructions for the agent. Be specific about:]
- What this agent is responsible for
- What it is NOT responsible for (scope boundaries)
- What to check before implementing (docs, live APIs, config files)
- Output format expected
- Any validation steps required before applying changes
```

---

## Implementation Agents

These cover your application's tech stack. Create one per major integration or domain area. Examples:

**For a team using a project management API:**
```markdown
---
name: Project Management API Specialist
description: [your PM tool], task management, issue, project API, task state, workflow state, task dependency
---

Expert in [your PM tool]'s API. Before implementing anything, fetch current API docs - do not rely on training data alone. Handles: creating/updating tasks via API, syncing state, building webhook handlers, querying data.

Validate against the live API endpoint before committing. Output changes as a diff with API call examples.
```

**For a team using a messaging/notification system:**
```markdown
---
name: Messaging Integration Specialist
description: [your messaging tool], webhook, channel, message, embed, notification, bot, OAuth, rate limit
---

Expert in [your messaging tool]'s API. Knows embed formats, rate limit handling (429 responses, retry-after), OAuth flow, and webhook registration. Always use embeds over plain text for structured data. Handle rate limits gracefully.
```

---

## Ops Agents

Cover infrastructure. One per service type. Examples:

**Docker:**
```markdown
---
name: Docker & Container Specialist
description: Docker, container, Compose, docker-compose, image, volume, network, service, Dockerfile, container health, restart policy
---

Expert in Docker and Docker Compose. Before modifying config, check current resource usage. Always: pin image versions (never :latest in production), include health checks in service definitions, set resource limits. Validate with `docker-compose config` before applying.
```

**Reverse Proxy:**
```markdown
---
name: Nginx & Reverse Proxy Specialist
description: Nginx, reverse proxy, proxy_pass, upstream, SSL, TLS, certificate, subdomain, rate limit, CORS, redirect
---

Expert in Nginx config. Always validate with `nginx -t` before reloading. Always include security headers: X-Frame-Options, X-Content-Type-Options, Content-Security-Policy, Strict-Transport-Security. Coordinate with DNS specialist when SSL changes are involved.
```

---

## Agent Registry

Create `.claude/docs/agent-registry.md` as a master list:

```markdown
# Agent Registry

| Agent | Category | File | Top Trigger Keywords | File Scope |
|-------|----------|------|---------------------|------------|
| Logic Reviewer | Review | logic-reviewer.md | PR review, business logic, correctness | PR diff |
| Test Reviewer | Review | test-reviewer.md | PR review, test coverage, missing tests | PR diff |
| Performance Reviewer | Review | performance-reviewer.md | PR review, N+1, bottleneck, latency | PR diff |
| Security Reviewer | Review | security-reviewer.md | PR review, security, injection, auth | PR diff |
| Review Team Lead | Review | review-lead.md | synthesize, dedup, card format | All agent outputs |
| [Your Stack Agent] | Implementation | [file].md | [keywords] | [scope] |
| Docker Specialist | Ops | docker-specialist.md | Docker, Compose, container | Dockerfile*, docker-compose* |
| Nginx Specialist | Ops | nginx-specialist.md | Nginx, reverse proxy, SSL | nginx.conf, sites-* |
```

Update this file whenever you add or modify an agent.

---

## CLAUDE.md Section to Add

```markdown
## Subagents

See `.claude/docs/agent-registry.md` for the full agent list.

Auto-invoke: Claude selects agents by matching task keywords to agent descriptions. No manual invocation needed.
Force a specific agent: "use the docker-specialist agent for this"

**Rules:**
- Agents stay within their defined file scope unless the task explicitly requires cross-cutting changes
- Implementation agents must check live API docs before implementing; never rely on training data alone
- Ops agents must validate config changes before applying (nginx -t, docker-compose config, etc.)
- Multi-scope tasks (e.g. add new service = Docker + Nginx + DNS): Claude proposes an agent team
```

---

## Testing Auto-Invoke

For each agent, give a task that contains its trigger keywords and verify the right agent activates:

```
"Add a health check to the Docker container" → docker-specialist
"Fix the Nginx CORS headers for the API subdomain" → nginx-specialist
"The [PM tool] API call for task creation is returning 404" → your PM specialist
```

If an agent fails to trigger, refine its `description` field with better keywords.