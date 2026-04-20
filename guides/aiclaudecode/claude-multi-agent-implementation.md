# 🛠️ Claude Multi-Agent Implementation Guide

## Quick Start: Multi-Agent Development Environment

### 1. Install Core Tools

```bash
# Install Claude Code CLI
npm install -g @anthropic-ai/claude-code

# Install Claude Flow for orchestration
npm install -g claude-flow@2.0.0

# Install Claude Swarm for team coordination
gem install claude_swarm

# Install Context7 for documentation
npm install -g @upstash/context7-mcp
```

### 2. Configure Cursor MCP Integration

```json
// ~/.cursor/mcp.json
{
  "mcpServers": {
    "context7": {
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp"]
    },
    "claude-flow": {
      "command": "npx",
      "args": ["-y", "claude-flow@2.0.0", "mcp", "start"]
    },
    "codemcp": {
      "command": "/Users/username/.local/bin/uvx",
      "args": [
        "--from",
        "git+https://github.com/ezyang/codemcp@prod",
        "codemcp"
      ]
    }
  }
}
```

### 3. Create Multi-Agent Development Swarm

```yaml
# claude-swarm.yml
version: 1
swarm:
  name: "Full Stack Development Team"
  main: architect
  before:
    - "npm install"
    - "docker-compose up -d"
  instances:
    architect:
      description: "System architect coordinating development"
      directory: .
      model: opus
      connections: [frontend_lead, backend_lead, devops]
      prompt: "You coordinate system design and ensure code quality"
      allowed_tools: [Read, Edit, WebSearch]
      
    frontend_lead:
      description: "Frontend team lead"
      directory: ./frontend
      model: opus
      connections: [react_dev, css_expert]
      prompt: "You lead frontend development with React and TypeScript"
      allowed_tools: [Edit, Write, Bash]
      
    react_dev:
      description: "React developer"
      directory: ./frontend/src
      model: opus
      prompt: "You specialize in React components and state management"
      allowed_tools: [Edit, Write, Bash]
      
    backend_lead:
      description: "Backend team lead"
      directory: ./backend
      model: opus
      connections: [api_dev, database_expert]
      prompt: "You lead API development and backend architecture"
      allowed_tools: [Edit, Write, Bash]
      
    api_dev:
      description: "API developer"
      directory: ./backend/src
      model: opus
      prompt: "You develop REST API endpoints"
      allowed_tools: [Edit, Write, Bash]
      
    database_expert:
      description: "Database specialist"
      directory: ./backend/db
      model: sonnet
      prompt: "You handle database schema and migrations"
      allowed_tools: [Edit, Write, Bash]
      
    devops:
      description: "DevOps engineer"
      directory: ./infrastructure
      model: opus
      prompt: "You handle CI/CD and infrastructure"
      allowed_tools: [Read, Edit, Bash]
```

### 4. Start Multi-Agent Development

```bash
# Start the development swarm
claude-swarm --worktree feature-auth

# In the architect session:
> Coordinate the development of a user authentication system with JWT tokens, password hashing, and rate limiting. The system should include user registration, login, password reset, and session management.

# The architect will delegate tasks to specialized agents:
# - Frontend Lead: React components and UI
# - Backend Lead: API endpoints and business logic
# - Database Expert: User schema and migrations
# - DevOps: Deployment and monitoring
```

---

## Advanced Multi-Terminal Setup

### 1. Create Multiple Claude Code Terminals

```bash
# Terminal 1: Main Development (Architect)
claude --system-prompt "You are a senior system architect. Coordinate development efforts and ensure code quality."

# Terminal 2: Frontend Development
claude --system-prompt "You are a senior React developer. Focus on modern React patterns, TypeScript, and user experience."

# Terminal 3: Backend Development
claude --system-prompt "You are a senior backend engineer. Focus on API design, security, and performance."

# Terminal 4: Database & Infrastructure
claude --system-prompt "You are a database architect and DevOps engineer. Focus on data modeling, migrations, and deployment."

# Terminal 5: Testing & Quality Assurance
claude --system-prompt "You are a QA engineer. Focus on testing strategies, code review, and quality assurance."
```

### 2. Inter-Terminal Coordination

```bash
# Create shared coordination files
mkdir .claude-coordination
touch .claude-coordination/tasks.json
touch .claude-coordination/progress.md
touch .claude-coordination/decisions.md

# Terminal 1 (Architect) - Define tasks
echo '{
  "tasks": [
    {
      "id": "auth-system",
      "title": "User Authentication System",
      "description": "Implement JWT-based authentication",
      "status": "in-progress",
      "assigned": "backend",
      "dependencies": []
    },
    {
      "id": "auth-ui",
      "title": "Authentication UI",
      "description": "Create login/register forms",
      "status": "pending",
      "assigned": "frontend",
      "dependencies": ["auth-system"]
    }
  ]
}' > .claude-coordination/tasks.json

# Terminal 2 (Frontend) - Check tasks
cat .claude-coordination/tasks.json | jq '.tasks[] | select(.assigned=="frontend")'

# Terminal 3 (Backend) - Update progress
echo "✅ JWT implementation complete" >> .claude-coordination/progress.md
```

### 3. Git Worktree Integration

```bash
# Create worktrees for parallel development
git worktree add ../feature-auth -b feature-auth
git worktree add ../feature-dashboard -b feature-dashboard
git worktree add ../bugfix-123 bugfix-123

# Run Claude Code in each worktree
cd ../feature-auth && claude --system-prompt "Authentication feature development"
cd ../feature-dashboard && claude --system-prompt "Dashboard feature development"
cd ../bugfix-123 && claude --system-prompt "Bug fix development"
```

---

## Claude Flow Orchestration

### 1. Initialize Claude Flow Swarm

```bash
# Initialize swarm
claude-flow swarm init --topology hierarchical --max-agents 8

# Spawn specialized agents
claude-flow agent spawn architect "System Designer" --capabilities "architecture,design"
claude-flow agent spawn coder "API Developer" --capabilities "api,backend"
claude-flow agent spawn coder "Frontend Dev" --capabilities "react,ui"
claude-flow agent spawn analyst "DB Designer" --capabilities "database,schema"
claude-flow agent spawn tester "QA Engineer" --capabilities "testing,qa"
claude-flow agent spawn coordinator "Lead" --capabilities "coordination,planning"
```

### 2. Orchestrate Complex Tasks

```bash
# Orchestrate full-stack development
claude-flow task orchestrate "Build a complete e-commerce platform with user authentication, product catalog, shopping cart, and payment processing" --strategy parallel --priority high

# Monitor progress
claude-flow swarm monitor --live --neural-insights

# Check agent status
claude-flow agent list --verbose
```

### 3. MCP Integration with Claude Code

```bash
# Add Claude Flow MCP server to Claude Code
claude mcp add claude-flow npx claude-flow mcp start

# Use Claude Flow tools in Claude Code
mcp__claude-flow__swarm_init { topology: "hierarchical", maxAgents: 8, strategy: "parallel" }
mcp__claude-flow__agent_spawn { type: "architect", name: "System Designer" }
mcp__claude-flow__agent_spawn { type: "coder", name: "API Developer" }
mcp__claude-flow__agent_spawn { type: "coder", name: "Frontend Dev" }
mcp__claude-flow__agent_spawn { type: "analyst", name: "DB Designer" }
mcp__claude-flow__agent_spawn { type: "tester", name: "QA Engineer" }
mcp__claude-flow__agent_spawn { type: "coordinator", name: "Lead" }
mcp__claude-flow__task_orchestrate { task: "Build REST API", strategy: "parallel" }
```

---

## Production Workflow Examples

### 1. Code Review Automation

```yaml
# code-review-swarm.yml
version: 1
swarm:
  name: "Code Review Team"
  main: lead_reviewer
  instances:
    lead_reviewer:
      description: "Lead code reviewer coordinating the review process"
      directory: .
      model: opus
      connections: [security_reviewer, performance_reviewer, doc_reviewer]
      prompt: "You coordinate comprehensive code reviews"
      allowed_tools: [Read, Edit]
      
    security_reviewer:
      description: "Security specialist reviewing for vulnerabilities"
      directory: .
      model: opus
      prompt: "You focus on security vulnerabilities and best practices"
      allowed_tools: [Read, Grep, Glob]
      
    performance_reviewer:
      description: "Performance specialist reviewing for optimization"
      directory: .
      model: opus
      prompt: "You focus on performance bottlenecks and optimization"
      allowed_tools: [Read, Grep, Glob]
      
    doc_reviewer:
      description: "Documentation specialist reviewing code documentation"
      directory: .
      model: sonnet
      prompt: "You focus on code documentation and comments"
      allowed_tools: [Read, Edit]
```

```bash
# Start code review
claude-swarm --config code-review-swarm.yml

# In lead_reviewer session:
> Review pull request #123 for security vulnerabilities, performance issues, and documentation quality. Provide actionable feedback for each area.
```

### 2. Research & Analysis Team

```yaml
# research-swarm.yml
version: 1
swarm:
  name: "Research Team"
  main: lead_researcher
  instances:
    lead_researcher:
      description: "Lead researcher coordinating analysis"
      directory: ~/research
      model: opus
      connections: [data_analyst, literature_reviewer, writer]
      prompt: "You coordinate research efforts and synthesize findings"
      allowed_tools: [Read, WebSearch, WebFetch]
      
    data_analyst:
      description: "Data analyst processing research data"
      directory: ~/research/data
      model: opus
      prompt: "You analyze data and create visualizations"
      allowed_tools: [Read, Write, Bash]
      
    literature_reviewer:
      description: "Literature reviewer analyzing papers"
      directory: ~/research/papers
      model: opus
      prompt: "You review academic papers and extract key insights"
      allowed_tools: [Read, WebSearch, Edit]
      
    writer:
      description: "Technical writer preparing documentation"
      directory: ~/research/docs
      model: opus
      prompt: "You write clear, technical documentation"
      allowed_tools: [Edit, Write, Read]
```

### 3. Performance Investigation Team

```yaml
# performance-swarm.yml
version: 1
swarm:
  name: "Performance Investigation"
  main: coordinator
  instances:
    coordinator:
      description: "Coordinates performance investigation"
      directory: ~/projects/webapp
      model: opus
      connections: [metrics_analyst, code_reviewer, fix_implementer]
      allowed_tools: [Read]
      prompt: |
        You coordinate a performance investigation team.
        1. Use metrics_analyst to identify when/where issues occur
        2. Use code_reviewer to find root causes
        3. Use fix_implementer to create solutions
        
    metrics_analyst:
      description: "Analyzes performance metrics"
      directory: ~/projects/webapp/logs
      model: sonnet
      allowed_tools: [Read, "Bash(grep:*, awk:*, sort:*)"]
      prompt: |
        You analyze logs and metrics for performance issues.
        Focus on response times, error rates, and patterns.
        
    code_reviewer:
      description: "Reviews code for performance issues"
      directory: ~/projects/webapp/src
      model: opus
      allowed_tools: [Read, Grep, Glob]
      prompt: |
        You review code for performance bottlenecks.
        Look for N+1 queries, missing indexes, inefficient algorithms.
        
    fix_implementer:
      description: "Implements performance fixes"
      directory: ~/projects/webapp
      model: opus
      allowed_tools: [Read, Edit, Write, Bash]
      prompt: |
        You implement optimizations and fixes.
        Always add tests and measure improvements.
```

---

## Advanced Coordination Patterns

### 1. Memory-Based Coordination

```javascript
// Store coordination data in shared memory
mcp__claude-flow__memory_usage {
  action: "store",
  key: "swarm-auth-system/architect/design",
  value: {
    timestamp: Date.now(),
    decision: "Use JWT tokens for authentication",
    implementation: "JWT-based auth system",
    nextSteps: ["Implement JWT middleware", "Create user endpoints"],
    dependencies: ["Database schema", "Password hashing"]
  }
}

// Retrieve coordination data
mcp__claude-flow__memory_usage {
  action: "retrieve",
  key: "swarm-auth-system/architect/design"
}

// Check all swarm progress
mcp__claude-flow__memory_usage {
  action: "list",
  pattern: "swarm-auth-system/*"
}
```

### 2. Batch Operations for Efficiency

```javascript
// ✅ CORRECT: Single message with ALL operations
[BatchTool - Message 1]:
  mcp__claude-flow__swarm_init { topology: "hierarchical", maxAgents: 8, strategy: "parallel" }
  mcp__claude-flow__agent_spawn { type: "architect", name: "System Designer" }
  mcp__claude-flow__agent_spawn { type: "coder", name: "API Developer" }
  mcp__claude-flow__agent_spawn { type: "coder", name: "Auth Expert" }
  mcp__claude-flow__agent_spawn { type: "analyst", name: "DB Designer" }
  mcp__claude-flow__agent_spawn { type: "tester", name: "Test Engineer" }
  mcp__claude-flow__agent_spawn { type: "coordinator", name: "Lead" }
  TodoWrite { todos: [
    { id: "design", content: "Design API architecture", status: "in_progress", priority: "high" },
    { id: "auth", content: "Implement authentication", status: "pending", priority: "high" },
    { id: "db", content: "Design database schema", status: "pending", priority: "high" },
    { id: "api", content: "Build REST endpoints", status: "pending", priority: "high" },
    { id: "tests", content: "Write comprehensive tests", status: "pending", priority: "medium" }
  ]}
  mcp__claude-flow__task_orchestrate { task: "Build REST API", strategy: "parallel" }
  mcp__claude-flow__memory_usage { action: "store", key: "project/init", value: { started: Date.now() } }

[BatchTool - Message 2]:
  Bash("mkdir -p test-app/{src,tests,docs,config}")
  Bash("mkdir -p test-app/src/{models,routes,middleware,services}")
  Write("test-app/package.json", packageJsonContent)
  Write("test-app/.env.example", envContent)
  Write("test-app/README.md", readmeContent)
```

### 3. Agent Coordination Hooks

```bash
# Before starting work
npx claude-flow hook pre-task --description "[agent task]" --auto-spawn-agents false
npx claude-flow hook session-restore --session-id "swarm-[id]" --load-memory true

# During work (after each file operation)
npx claude-flow hook post-edit --file "[filepath]" --memory-key "swarm/[agent]/[step]"

# Store decisions and findings
npx claude-flow hook notification --message "[what was done]" --telemetry true

# Check coordination with other agents
npx claude-flow hook pre-search --query "[what to check]" --cache-results true

# After completing work
npx claude-flow hook post-task --task-id "[task]" --analyze-performance true
```

---

## Monitoring & Observability

### 1. Session Monitoring

```bash
# Monitor active sessions
claude-swarm ps

# Show detailed session information
claude-swarm show 20250617_143022

# Watch live logs
claude-swarm watch 20250617_143022

# List all sessions
claude-swarm list-sessions --limit 20

# Clean old sessions
claude-swarm clean --days 30
```

### 2. Performance Monitoring

```bash
# Enable OpenTelemetry
export CLAUDE_CODE_ENABLE_TELEMETRY=1
export OTEL_METRICS_EXPORTER=otlp
export OTEL_LOGS_EXPORTER=otlp
export OTEL_EXPORTER_OTLP_PROTOCOL=grpc
export OTEL_EXPORTER_OTLP_ENDPOINT=http://localhost:4317

# Monitor usage
claude-code-usage-monitor --real-time --predictions

# Export metrics
claude-code-usage-monitor --export metrics.json
```

### 3. Cost Optimization

```bash
# Set token limits
export CLAUDE_CODE_MAX_OUTPUT_TOKENS=4000
export MAX_THINKING_TOKENS=1000

# Use smaller models for background tasks
export ANTHROPIC_SMALL_FAST_MODEL="claude-3-5-haiku-20241022"

# Enable conversation compaction
echo "# Summary instructions" >> CLAUDE.md
echo "When you are using compact, please focus on test output and code changes" >> CLAUDE.md
```

---

## Troubleshooting

### 1. Common Issues

```bash
# Permission denied errors
chmod +x ~/.local/bin/uvx
chmod +x ~/.npm-global/bin/claude

# MCP server connection issues
claude mcp list
claude mcp test context7

# Session restoration issues
claude-swarm list-sessions
claude-swarm show session_id

# Memory coordination issues
mcp__claude-flow__memory_usage { action: "list", pattern: "*" }
```

### 2. Performance Issues

```bash
# Check agent status
claude-flow agent list --verbose

# Reset coordination
claude-flow swarm reset --preserve-memory
claude-flow swarm init --topology adaptive
claude-flow coordination sync --force-resync

# Monitor resource usage
top -p $(pgrep -f "claude")
```

### 3. Security Issues

```bash
# Review permissions
cat settings.json | jq '.permissions'

# Check allowed tools
claude --allowedTools "Read,Write,Bash"

# Verify MCP servers
cat ~/.cursor/mcp.json | jq '.mcpServers'
```

---

## 🎯 Implementation Checklist

### ✅ Basic Setup
- [ ] Claude Code CLI installed
- [ ] Claude Flow installed
- [ ] Claude Swarm installed
- [ ] Cursor MCP configured
- [ ] Environment variables set

### ✅ Multi-Agent Configuration
- [ ] Development swarm configured
- [ ] Agent roles defined
- [ ] Tool permissions set
- [ ] Inter-agent connections established
- [ ] Memory coordination working

### ✅ Advanced Features
- [ ] Git worktree integration
- [ ] Multiple terminal setup
- [ ] Batch operations implemented
- [ ] Coordination hooks active
- [ ] Monitoring configured

### ✅ Production Ready
- [ ] Code review automation
- [ ] Research team setup
- [ ] Performance investigation
- [ ] Security best practices
- [ ] Cost optimization

This implementation guide provides practical examples for setting up sophisticated multi-agent Claude development environments. Start with the basic setup and gradually add advanced features as you become comfortable with each component.
