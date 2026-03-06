# Agent Specification Reference

This document explains the Claude Code agent YAML frontmatter fields that Forge agents use. Most agent collections ignore these — we use all of them.

## Frontmatter Fields

### `name` (required)

```yaml
name: code-reviewer
```

Unique identifier. Lowercase with hyphens. Used in `Task()` references and `--agent` CLI flag.

### `description` (required)

```yaml
description: >
  Code quality reviewer. Analyzes code for bugs, design issues,
  performance problems. Read-only — never modifies code.
```

Claude uses this to decide when to delegate to this agent. Be specific about capabilities AND limitations.

### `tools`

```yaml
tools:
  - Read
  - Glob
  - Grep
  - Bash
```

Allowlist of tools the agent can use. Without this, the agent inherits all tools from the parent.

### `disallowedTools`

```yaml
disallowedTools:
  - Write
  - Edit
  - NotebookEdit
  - Agent
  - WebSearch
  - WebFetch
```

Denylist — these tools are blocked even if inherited. This is how we enforce read-only quality agents. This is a **hard guarantee**, not a prompt instruction.

### `model`

```yaml
model: sonnet  # sonnet | opus | haiku | inherit
```

- `opus` — best quality, highest cost. Use for orchestration and complex decisions.
- `sonnet` — good balance. Use for most work agents.
- `haiku` — fastest, cheapest. Use for simple lookup tasks.
- `inherit` — use whatever model the parent session uses.

### `permissionMode`

```yaml
permissionMode: bypassPermissions  # default | acceptEdits | dontAsk | bypassPermissions | plan
```

- `default` — ask user for permission on risky actions.
- `bypassPermissions` — skip permission prompts. Use for agents that need to work autonomously (CI, quality gates).
- `plan` — read-only mode, can only plan.

### `maxTurns`

```yaml
maxTurns: 60
```

Hard limit on agent turns. When reached, the agent is terminated. Always set this — without it, agents can run indefinitely.

**Rule of thumb:** Set `maxTurns` to your turn budget total plus 10-15% reserve.

### `memory`

```yaml
memory: local  # user | project | local
```

| Scope | Location | Shared | Best for |
|-------|----------|--------|----------|
| `user` | `~/.claude/agent-memory/<agent>/` | All projects | Orchestrators, cross-project knowledge |
| `project` | `.claude/agent-memory/<agent>/` | Via git | Dev agents, project-specific patterns |
| `local` | `.claude/agent-memory-local/<agent>/` | Not shared | Quality agents, per-machine knowledge |

The agent's `MEMORY.md` (first 200 lines) is loaded automatically every session. Use it for hot-cache knowledge.

### `mcpServers`

```yaml
mcpServers:
  - eslint
  - semgrep
  - memory
```

MCP servers available to this agent. Servers must be configured in `~/.claude.json`. Agents should always have a Bash fallback for MCP tools that might be unavailable.

### `Task()` (in tools)

```yaml
tools:
  - Task(backend-dev, frontend-dev, code-reviewer)
```

Restricts which agents the orchestrator can spawn. Without this, the agent could delegate to any agent. Use this to enforce the pipeline structure.

**Critical limitation:** Subagents cannot spawn other subagents. Only one level of depth. The conductor works around this by being launched as the main thread (`claude --agent conductor`).

## Forge Patterns

### Turn Budgets

Every agent has an internal phase budget that sums to `maxTurns`:

```
| Phase            | Turns |
|------------------|-------|
| Detection        | 2-4   |
| Reading          | 15-25 |
| Core work        | 30-55 |
| Validation       | 5-10  |
| Memory           | 1     |
| Report           | 2-5   |
| Reserve          | 7-15  |
```

At ~75% turns used, agents trigger partial reporting.

### Bash Timeout Tables

Every Bash command has an explicit `timeout` parameter:

```
| Command Type       | Timeout  |
|--------------------|----------|
| Quick checks       | 5000ms   |
| Git operations     | 30000ms  |
| Static analysis    | 60000ms  |
| Test execution     | 120000ms |
```

This prevents hanging on slow commands, network issues, or stuck processes.

### Verdict System

Quality agents return structured verdicts:

```
### Verdict: [PASS | PASS WITH NOTES | NEEDS CHANGES | CRITICAL ISSUES]
```

Each verdict maps to a conductor action. The conductor never guesses — it reads the verdict line and acts accordingly.
