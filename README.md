# Claude Agents Forge

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-agents-blue)](https://code.claude.com)

**Production-hardened agent system for Claude Code that actually uses the full agent spec.**

Most Claude Code agent collections are just system prompts with `tools: [Read, Write, Edit, Bash]`. They ignore 80% of the agent specification — no turn limits, no permission modes, no memory, no MCP integration, no tool restrictions, no real orchestration.

This project is different. Every agent is built for real work, not demos.

---

## What Makes This Different

| Feature | Other collections | Forge |
|---------|:-:|:-:|
| `maxTurns` + turn budgets | - | Per-agent limits with phase budgets |
| `permissionMode` | - | `bypassPermissions` for CI, `default` for orchestration |
| `disallowedTools` | - | Read-only quality agents can't write files |
| `memory` scopes | - | `user` / `project` / `local` per agent role |
| `mcpServers` in agents | - | ESLint, PHPStan, Semgrep, Memory Service |
| `Task()` orchestration | - | Conductor spawns specific agents, not "any" |
| Bash timeout tables | - | Per-command timeouts to prevent hangs |
| Quality gate pipeline | - | Mandatory review after every code change |
| Verdict-driven flow | - | Agent verdicts control pipeline progression |

> We reviewed the top Claude Code agent repos on GitHub (including ones with 30k+ and 60k+ stars). **None** use `maxTurns`, `permissionMode`, `disallowedTools`, or `Task()` syntax. [Full comparison](docs/comparison.md)

---

## Architecture

```
User Task
    |
    v
[Conductor] ── orchestrates, never writes code
    |
    |── delegates to ──> [Dev Agent]     (backend / frontend / devops)
    |                         |
    |                         v
    |                    code changes
    |                         |
    |── quality gates ──> [Code Reviewer]      always
    |                     [Security Auditor]   if auth/input/API
    |                     [Test Engineer]       if logic changed
    |                     [Doc Writer]          if behavior changed
    |                     [Monitoring Engineer] if infra changed
    |
    v
Structured Report + Verdict
```

**Key rules:**
- Conductor **never** writes code — `Write`/`Edit` are in `disallowedTools`
- Quality agents are **read-only** — they can't modify your codebase
- Dev agents run **sequentially** — no file conflicts
- Quality gates run **in parallel** — fast feedback
- Every agent has a **turn budget** — no infinite loops
- Pipeline stops on **CRITICAL** verdicts — no broken code gets through

---

## Agents

### Orchestration

| Agent | Model | Turns | Memory | Role |
|-------|-------|-------|--------|------|
| [conductor](agents/orchestration/conductor.md) | opus | 200 | user | Decomposes tasks, delegates, runs quality gates |

### Quality (read-only)

| Agent | Model | Turns | Memory | Role |
|-------|-------|-------|--------|------|
| [code-reviewer](agents/quality/code-reviewer.md) | sonnet | 60 | local | Bugs, design, performance, tech debt |
| [security-auditor](agents/quality/security-auditor.md) | sonnet | 60 | local | OWASP, injection, auth, secrets, SAST |
| [test-engineer](agents/quality/test-engineer.md) | sonnet | 90 | local | Coverage gaps + writes tests (dual mode) |
| [doc-writer](agents/quality/doc-writer.md) | sonnet | 80 | local | README, API docs, changelog (dual mode) |
| [monitoring-engineer](agents/quality/monitoring-engineer.md) | sonnet | 70 | local | Logging, metrics, alerts, health checks |

### Development

| Agent | Model | Turns | Memory | Role |
|-------|-------|-------|--------|------|
| [backend-dev](agents/development/backend-dev.md) | sonnet | 120 | project | Backend APIs, models, migrations, services |
| [frontend-dev](agents/development/frontend-dev.md) | sonnet | 120 | project | Components, pages, state, routing, styling |
| [devops-engineer](agents/development/devops-engineer.md) | sonnet | 100 | project | Docker, CI/CD, Nginx, deployment |

---

## Quick Start

### 1. Install agents globally

```bash
# Clone the repo
git clone https://github.com/Mationetap/claude-agents-forge.git

# Copy agents to Claude Code's global agent directory
cp claude-agents-forge/agents/**/*.md ~/.claude/agents/
```

### 2. Run the conductor

```bash
# Start Claude Code with the conductor as the main agent
claude --agent conductor
```

The conductor will:
1. Detect your project (reads `package.json`, `composer.json`, `CLAUDE.md`)
2. Decompose your task into subtasks
3. Delegate to the right dev agent
4. Run quality gates automatically
5. Report results with structured verdicts

### 3. Or use agents individually

```bash
# Direct code review
claude --agent code-reviewer

# Security audit
claude --agent security-auditor

# Just write code with a dev agent
claude --agent backend-dev
```

---

## Production Hardening Features

### Turn Budgets

Every agent has a `maxTurns` limit and an internal phase budget:

```
| Phase            | Max turns |
|------------------|-----------|
| Stack detection  | 2         |
| File reading     | 20        |
| Static analysis  | 5         |
| Analysis         | 14        |
| Memory update    | 1         |
| Report           | 2         |
| Reserve          | 7         |
```

If an agent reaches ~75% of its turn limit without starting the report phase, it immediately compiles a partial report. A partial report is always better than no report.

### Bash Timeout Tables

Every Bash command has an explicit timeout to prevent hangs:

```
| Command                    | Timeout (ms) |
|----------------------------|-------------|
| git diff/log/status        | 30000       |
| phpstan/eslint             | 60000       |
| semgrep                    | 120000      |
| test execution             | 120000      |
| file checks (test -f, wc)  | 5000        |
```

### Read-Only Quality Agents

Quality agents use `disallowedTools` to enforce read-only behavior:

```yaml
disallowedTools:
  - Write
  - Edit
  - NotebookEdit
  - Agent       # can't spawn sub-agents
  - WebSearch   # no network access
  - WebFetch    # no network access
```

This is a hard guarantee — not a prompt instruction that can be ignored.

### Verdict-Driven Pipeline

Quality agents return structured verdicts that control pipeline flow:

| Verdict | Action |
|---------|--------|
| `PASS` | Continue / mark done |
| `PASS WITH NOTES` | Ask user: fix or accept? |
| `NEEDS CHANGES` | Route back to dev agent |
| `CRITICAL ISSUES` | Block — route back for fix |

Max 3 correction round-trips per pipeline to prevent infinite loops.

---

## Customization

### Adding Your Own Dev Agent

Create a new file in `agents/development/`:

```yaml
---
name: rails-dev
description: >
  Ruby on Rails backend developer. Implements APIs, models, migrations,
  background jobs, and admin panels. Only modifies Ruby files.
tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - Bash
disallowedTools:
  - Agent
  - WebSearch
  - WebFetch
  - NotebookEdit
model: sonnet
permissionMode: bypassPermissions
maxTurns: 120
memory: project
---

# Rails Dev — Backend Developer

Your system prompt here...
```

Then add it to the conductor's `Task()` list:

```yaml
tools:
  - Task(backend-dev, frontend-dev, rails-dev, ...)
```

### Adjusting Quality Gates

In `conductor.md`, the quality gate matrix controls which gates run:

```
| Change type              | security | tests | review | docs | monitoring |
|--------------------------|:---:|:---:|:---:|:---:|:---:|
| Auth/login/permissions   | YES | YES | YES | YES | -   |
| New API endpoint         | YES | YES | YES | YES | -   |
| Frontend component       | -   | if complex | YES | - | - |
| Docker/CI/CD             | -   | -   | YES | -   | YES |
| CSS/styling only         | -   | -   | YES | -   | -   |
```

Edit this matrix to match your workflow.

### Memory Scopes

| Scope | Location | Best for |
|-------|----------|----------|
| `user` | `~/.claude/agent-memory/<agent>/` | Cross-project knowledge (conductor) |
| `project` | `.claude/agent-memory/<agent>/` | Project-specific, shared via git (dev agents) |
| `local` | `.claude/agent-memory-local/<agent>/` | Project-specific, private (quality agents) |

Quality agents use `local` memory — they learn your project's conventions, linting config, and common patterns across sessions without polluting git.

---

## MCP Integration

Agents can declare MCP servers in their frontmatter:

```yaml
mcpServers:
  - eslint      # JS/TS linting
  - phpstan     # PHP static analysis
  - semgrep     # Security scanning
  - memory      # MCP Memory Service
```

MCP tools are tried first; if they fail, agents fall back to CLI equivalents. This "MCP first, Bash fallback" pattern handles known MCP reliability issues in subagents.

### Setting Up MCP Servers

Configure your MCP servers in `~/.claude.json`. The exact package names depend on which MCP server implementations you use. Example structure:

```json
{
  "mcpServers": {
    "eslint": {
      "command": "npx",
      "args": ["-y", "your-eslint-mcp-package"]
    },
    "semgrep": {
      "command": "npx",
      "args": ["-y", "your-semgrep-mcp-package"]
    }
  }
}
```

See the [Claude Code MCP documentation](https://code.claude.com/docs/en/mcp) for available servers and setup instructions.

---

## File Structure

```
claude-agents-forge/
  agents/
    orchestration/
      conductor.md          # Central orchestrator (opus, 200 turns)
    quality/
      code-reviewer.md      # Read-only code analysis (sonnet, 60 turns)
      security-auditor.md   # Read-only security audit (sonnet, 60 turns)
      test-engineer.md      # Dual-mode: review + write tests (sonnet, 90 turns)
      doc-writer.md         # Dual-mode: review + write docs (sonnet, 80 turns)
      monitoring-engineer.md # Dual-mode: review + write configs (sonnet, 70 turns)
    development/
      backend-dev.md        # Backend development (sonnet, 120 turns)
      frontend-dev.md       # Frontend development (sonnet, 120 turns)
      devops-engineer.md    # Infrastructure & CI/CD (sonnet, 100 turns)
  docs/
    getting-started.md      # Detailed setup guide
    agent-spec.md           # Full agent YAML spec reference
    comparison.md           # Side-by-side comparison with other repos
  LICENSE
  CONTRIBUTING.md
```

---

## FAQ

**Q: Why not 135 agents like other repos?**
A: Because 9 well-configured agents that actually work beat 135 generic prompts. Each of our agents uses features that none of theirs do. Quality over quantity.

**Q: Do I need all the MCP servers?**
A: No. MCP integration is optional — agents fall back to CLI tools automatically. Start without MCP, add servers when you need them.

**Q: Can I use this with my own stack?**
A: Yes. The quality agents (code-reviewer, security-auditor, test-engineer) are stack-agnostic. Dev agents are templates — fork `backend-dev.md` and customize for your stack.

**Q: Does this work on Windows?**
A: Yes. Agents use Bash tool timeouts (not the `timeout` CLI command), POSIX paths in Git Bash, and handle Windows-specific edge cases.

**Q: Why does the conductor use opus?**
A: The conductor makes complex routing decisions and manages multi-agent pipelines. Opus handles this better than sonnet. Dev and quality agents use sonnet — they execute, not decide.

---

## Roadmap

<table>
<tr>
<td><b>v1.0</b> Foundation</td>
<td>:white_check_mark: Done</td>
<td>9 agents, full spec coverage, verdict pipeline</td>
</tr>
<tr>
<td><b>v1.1</b> More Agents</td>
<td>:construction: Planned</td>
<td>database-engineer, api-designer, refactoring-agent, and more</td>
</tr>
<tr>
<td><b>v2.0</b> Agent Teams</td>
<td>:construction: Planned</td>
<td>Parallel agents with shared tasks and mailbox</td>
</tr>
<tr>
<td><b>v2.2</b> Memory</td>
<td>:construction: Planned</td>
<td>MCP Memory Service, cross-session learning, auto-calibration</td>
</tr>
<tr>
<td><b>v3.0</b> Plugin System</td>
<td>:construction: Planned</td>
<td>One-command install, agent marketplace</td>
</tr>
<tr>
<td><b>v3.1</b> CI/CD</td>
<td>:construction: Planned</td>
<td>GitHub Actions, PR comment bot, status checks</td>
</tr>
</table>

**[Full Roadmap →](ROADMAP.md)**

---

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

Key rules:
- Every agent MUST use `maxTurns`, `memory`, and `disallowedTools`
- Quality agents MUST be read-only (no `Write`/`Edit`)
- Dev agents MUST declare their file scope
- No "awesome list" padding — only agents you've tested in real work

---

## Keywords

`claude-code` `claude-code-agents` `claude-code-subagents` `ai-agents` `multi-agent` `orchestration` `code-review` `security-audit` `devops` `claude` `anthropic` `llm-agents` `agentic-coding` `claude-code-skills` `production-ready`

---

## License

MIT
