# Comparison with Other Claude Code Agent Collections

Last updated: March 2026

We reviewed the top Claude Code agent repositories to understand what features they use from the [agent specification](https://code.claude.com/docs/en/sub-agents).

## Feature Matrix

| Feature | [everything-claude-code](https://github.com/affaan-m/everything-claude-code) (62k stars) | [wshobson/agents](https://github.com/wshobson/agents) (30k stars) | [awesome-claude-agents](https://github.com/vijaythecoder/awesome-claude-agents) (4k stars) | [awesome-claude-code-toolkit](https://github.com/rohitg00/awesome-claude-code-toolkit) (640 stars) | **Forge** |
|---------|:-:|:-:|:-:|:-:|:-:|
| `maxTurns` | - | - | - | - | Per-agent limits |
| `permissionMode` | - | - | - | - | Per-agent modes |
| `disallowedTools` | - | - | - | - | Read-only quality agents |
| `memory` scopes | via hooks | - | - | 1 of 135 | All 3 scopes |
| `mcpServers` in agents | as config file | - | - | - | In frontmatter |
| `Task()` orchestration | - | - | - | - | Conductor pattern |
| Turn budgets | - | - | - | - | Phase-level budgets |
| Bash timeouts | - | - | - | - | Per-command tables |
| Quality gate pipeline | - | - | - | - | Verdict-driven flow |
| Agent count | 12 | 112 | 24 | 135 | 9 |

## Key Differences

### Turn Limits

None of the reviewed repositories set `maxTurns` on their agents. This means agents can run indefinitely, consuming tokens without producing results. Forge agents have explicit turn limits with phase budgets and safety triggers for partial reporting at 75%.

### Read-Only Enforcement

In most collections, code reviewers and security auditors have `Write` and `Edit` tools available. This means a "reviewer" agent can silently modify your code. In Forge, quality agents use `disallowedTools` to enforce read-only behavior at the tool level — not as a prompt instruction.

### Real Orchestration

Several repos describe "orchestration" in their prompts, but none use the `Task()` syntax that actually spawns subagents. Forge's conductor uses `Task(backend-dev, frontend-dev, ...)` to delegate work, with explicit scope containment and context passing.

### Memory Strategy

Forge uses all three memory scopes intentionally:
- `user` for conductor (cross-project knowledge)
- `project` for dev agents (project-specific, shared via git)
- `local` for quality agents (per-project, not in git)

### Verdict System

Quality agents return structured verdicts (`PASS`, `PASS WITH NOTES`, `NEEDS CHANGES`, `CRITICAL ISSUES`) that control pipeline flow. The conductor has explicit logic for each verdict type, including max correction rounds to prevent infinite loops.

## Methodology

We cloned each repository and searched for these frontmatter fields across all agent files:

```bash
grep -rl "maxTurns" agents/
grep -rl "permissionMode" agents/
grep -rl "disallowedTools" agents/
grep -rl "memory:" agents/
grep -rl "mcpServers" agents/
```

Results were verified by reading representative agent files from each collection.
