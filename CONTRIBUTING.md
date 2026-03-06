# Contributing to Claude Agents Forge

## Agent Quality Standards

Every agent in this project must meet these requirements. No exceptions.

### Required Frontmatter Fields

```yaml
---
name: agent-name          # Required: lowercase, hyphens
description: >            # Required: when to use this agent
  Clear description of role and capabilities
tools:                    # Required: explicit tool list
  - Read
  - Glob
  - Grep
  - Bash
disallowedTools:          # Required: what the agent CANNOT do
  - Write                 # Quality agents must block Write/Edit
  - Edit
model: sonnet             # Required: sonnet for workers, opus for orchestration
permissionMode: bypassPermissions  # Required: explicit permission mode
maxTurns: 60              # Required: turn limit with safety margin
memory: local             # Required: user | project | local
---
```

### Turn Budget

Every agent must have an internal turn budget table:

```
| Phase            | Max turns |
|------------------|-----------|
| Stack detection  | N         |
| File reading     | N         |
| Core work        | N         |
| Memory update    | 1         |
| Report           | N         |
| Reserve          | N         |
```

The reserve should be ~10-15% of maxTurns. Include a "turn limit safety" instruction that triggers partial reporting at ~75% of maxTurns.

### Bash Timeout Table

Every agent that uses Bash must have explicit timeout values:

```
| Command           | Timeout (ms) |
|-------------------|-------------|
| git commands      | 30000       |
| static analysis   | 60000       |
| file checks       | 5000        |
```

### Quality Agents Must Be Read-Only

Agents in `agents/quality/` must have `Write`, `Edit`, `NotebookEdit` in `disallowedTools`. Exceptions: dual-mode agents (test-engineer, doc-writer, monitoring-engineer) that have an explicit write mode.

### Dev Agents Must Declare File Scope

Agents in `agents/development/` must have a "File Scope" section listing exactly which directories they can modify and which are off-limits.

### Memory Strategy

Every agent must have a "Memory Strategy" section explaining:
- What goes in MEMORY.md (hot cache, loaded every session)
- What to store in Memory Service (if configured)
- What NOT to store

## Submitting Changes

1. Fork the repo
2. Create a feature branch
3. Make your changes
4. Test the agent on a real project (not just reading the prompt)
5. Open a PR with:
   - What the agent does
   - How you tested it
   - Before/after comparison if modifying an existing agent

## What We Accept

- New agents that fill real gaps (not duplicates of existing ones)
- Improvements to existing agents backed by testing
- Bug fixes
- Documentation improvements

## What We Don't Accept

- Generic prompts without `maxTurns`, `memory`, `disallowedTools`
- "Awesome list" padding — agents must be tested on real projects
- Agents that duplicate existing quality gates
- Breaking changes to the conductor pipeline without discussion
