# Getting Started

## Prerequisites

- [Claude Code CLI](https://code.claude.com) installed and authenticated
- A project directory with source code

## Installation

### Option 1: Global (all projects)

```bash
git clone https://github.com/Mationetap/claude-agents-forge.git
cp claude-agents-forge/agents/**/*.md ~/.claude/agents/
```

### Option 2: Per-project

```bash
# From your project root
mkdir -p .claude/agents
cp path/to/claude-agents-forge/agents/**/*.md .claude/agents/
```

### Option 3: Symlinks (recommended for development)

```bash
# Linux/macOS
ln -s /path/to/claude-agents-forge/agents/quality/*.md ~/.claude/agents/
ln -s /path/to/claude-agents-forge/agents/development/*.md ~/.claude/agents/
ln -s /path/to/claude-agents-forge/agents/orchestration/*.md ~/.claude/agents/

# Windows (requires admin or developer mode)
mklink ~/.claude/agents/conductor.md C:\path\to\claude-agents-forge\agents\orchestration\conductor.md
```

## Verify Installation

```bash
claude --agent conductor
```

The conductor should detect your project and report available agents.

## Usage Modes

### Full Pipeline (recommended)

```bash
claude --agent conductor
# Give any task — conductor will delegate and run quality gates
```

### Individual Agents

```bash
# Code review
claude --agent code-reviewer

# Security audit
claude --agent security-auditor

# Write tests
claude --agent test-engineer

# Any dev work
claude --agent backend-dev
claude --agent frontend-dev
claude --agent devops-engineer
```

### With MCP Servers (optional)

Add to `~/.claude.json`:

```json
{
  "mcpServers": {
    "eslint": {
      "command": "npx",
      "args": ["-y", "@anthropic/eslint-mcp"]
    },
    "semgrep": {
      "command": "npx",
      "args": ["-y", "@anthropic/semgrep-mcp"]
    }
  }
}
```

Agents will use MCP tools when available and fall back to CLI equivalents.

## Customizing for Your Stack

### Replace the Backend Dev Agent

The default `backend-dev.md` uses Laravel patterns. To customize for your stack:

1. Copy `agents/development/backend-dev.md` to `agents/development/rails-dev.md`
2. Update the frontmatter (`name`, `description`)
3. Replace Laravel patterns with your framework's patterns
4. Keep the workflow structure, turn budgets, and report format
5. Add to the conductor's `Task()` list

### Adjust Quality Gate Matrix

In `conductor.md`, find the quality gate matrix and adjust which gates run for which change types. For example, if your project doesn't need monitoring reviews, remove the monitoring-engineer column.

### Change Models

- Conductor uses `opus` for complex orchestration decisions
- All other agents use `sonnet` for balanced speed/quality
- Change to `haiku` for faster but less thorough agents
- Change to `opus` for critical agents that need maximum quality

## Troubleshooting

**Agent not found:** Verify files are in `~/.claude/agents/` or `.claude/agents/`

**Conductor can't delegate:** Check that all agents referenced in `Task()` are installed

**MCP tools failing:** Agents fall back to Bash CLI automatically. Check your `~/.claude.json` MCP config if you want MCP tools to work.

**Agent runs too long:** Each agent has a `maxTurns` limit. If it's still too long, reduce `maxTurns` in the frontmatter.
