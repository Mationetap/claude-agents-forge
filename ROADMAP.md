# Roadmap

> **Contributions welcome** — pick any item, [open an issue](https://github.com/Mationetap/claude-agents-forge/issues), submit a PR.

<br>

```mermaid
timeline
    title Claude Agents Forge — Roadmap
    section v1.0
        Foundation : 9 agents
                   : Full spec coverage
                   : Verdict pipeline
    section v1.x
        More Agents : database-engineer
                    : api-designer
                    : refactoring-agent
        Stack Devs : rails, django
                   : nestjs, go
                   : flutter, unity
    section v2.x
        Agent Teams : Parallel agents
                    : Shared tasks
                    : Mailbox
        Orchestration : Multi-stage pipelines
                      : Conditional gates
                      : Cost tracking
        Memory : MCP Memory Service
               : Cross-session learning
               : Project fingerprinting
    section v3.x
        Plugin System : One-command install
                      : Agent marketplace
        CI/CD : GitHub Actions
              : PR comment bot
              : Status checks
        Analytics : Dashboard
                  : Trend tracking
```

<br>

## v1.0 — Foundation ![Status](https://img.shields.io/badge/status-done-brightgreen?style=flat-square)

> 9 production-hardened agents with full spec coverage

- [x] Conductor orchestrator (opus, 200 turns)
- [x] 5 quality agents — read-only enforcement via `disallowedTools`
- [x] 3 dev agents — backend, frontend, devops
- [x] Turn budgets + Bash timeout tables
- [x] Verdict-driven pipeline (PASS → CRITICAL)
- [x] Memory scopes — user / project / local
- [x] MCP first, Bash fallback pattern

<br>

## v1.1 — More Agents ![Status](https://img.shields.io/badge/status-planned-blue?style=flat-square)

> Expand coverage without bloat — every agent earns its place

| Agent | Purpose |
|-------|---------|
| `database-engineer` | Schema design, migrations, query optimization, index analysis |
| `api-designer` | OpenAPI spec review, REST/GraphQL best practices, versioning |
| `refactoring-agent` | Code smell detection, extract method/class, dead code removal |
| `dependency-auditor` | Outdated packages, license compliance, vulnerability scanning |
| `i18n-reviewer` | Missing translations, hardcoded strings, locale consistency |
| `accessibility-auditor` | WCAG compliance, aria attributes, keyboard navigation |

<br>

## v1.2 — Stack-Specific Dev Agents ![Status](https://img.shields.io/badge/status-planned-blue?style=flat-square)

> Fork `backend-dev.md`, customize for your stack

| Agent | Stack |
|-------|-------|
| `rails-dev` | Ruby on Rails — models, migrations, jobs, admin |
| `django-dev` | Python / Django |
| `nestjs-dev` | NestJS / TypeScript |
| `go-dev` | Go — net/http, gin, fiber |
| `flutter-dev` | Dart / Flutter mobile |
| `unity-dev` | C# / Unity game dev |

<br>

## v2.0 — Agent Teams ![Status](https://img.shields.io/badge/status-planned-blue?style=flat-square)

> Parallel agents that talk to each other — not just to the conductor

```mermaid
graph LR
    TL[Team Lead] -->|task| A1[Backend Dev]
    TL -->|task| A2[Frontend Dev]
    TL -->|task| A3[DevOps]
    A1 <-->|mailbox| A2
    A2 <-->|mailbox| A3
    A1 -->|done| TL
    A2 -->|done| TL
    A3 -->|done| TL
```

- [ ] Agent Teams support (experimental Claude Code feature)
- [ ] Team Lead agent coordinating parallel dev agents
- [ ] Shared task list between teammates
- [ ] Direct mailbox communication between agents
- [ ] Conflict resolution for overlapping file changes

<br>

## v2.1 — Advanced Orchestration ![Status](https://img.shields.io/badge/status-planned-blue?style=flat-square)

> Smarter pipelines, fewer wasted turns

```mermaid
graph LR
    P[Plan] --> I[Implement] --> T[Test] --> R[Review] --> D[Deploy]
    R -->|NEEDS CHANGES| I
    T -->|FAIL| I
```

- [ ] Multi-stage pipelines — plan → implement → test → review → deploy
- [ ] Conditional gates — skip security audit for CSS-only changes
- [ ] Parallel dev agents with file locking
- [ ] Retry policies — configurable count, exponential backoff
- [ ] Pipeline templates — presets for bug fix, feature, refactor
- [ ] Cost tracking — estimate token usage per pipeline run

<br>

## v2.2 — Memory & Learning ![Status](https://img.shields.io/badge/status-planned-blue?style=flat-square)

> Agents that get smarter with every run

```mermaid
graph TD
    A[Agent runs] --> L{Learned something?}
    L -->|yes| M[MCP Memory Service]
    M --> S[(SQLite-vec)]
    S -->|semantic search| N[Next session]
    N --> A
    L -->|no| D[Done]
```

- [ ] **MCP Memory Service** — structured long-term memory (SQLite-vec, semantic search)
- [ ] **Memory-aware prompts** — consult memory before work, save findings after
- [ ] **Project fingerprinting** — auto-detect stack, conventions, config → persist
- [ ] **Cross-session learning** — remember patterns, past findings, recurring issues
- [ ] **Team knowledge base** — shared memory across quality agents
- [ ] **Memory scoping strategy** — guidelines for user / project / local
- [ ] **Auto-calibration** — adjust review depth based on past false positive rate
- [ ] **Pattern library** — reusable checklists per framework, stored in memory
- [ ] **Memory export/import** — share learned patterns between projects
- [ ] **Memory hygiene** — auto-cleanup of stale/conflicting memories, dedup
- [ ] **Onboarding memory** — first-run scan populates project structure & deps

<br>

## v3.0 — Plugin System ![Status](https://img.shields.io/badge/status-planned-blue?style=flat-square)

> One-command install, community marketplace

- [ ] Package as Claude Code plugin (`plugin.json` manifest)
- [ ] One-command installation via plugin registry
- [ ] Settings UI for selecting which agents to enable
- [ ] Agent marketplace — community agents with ratings
- [ ] Plugin hooks for custom pre/post agent actions

<br>

## v3.1 — CI/CD Integration ![Status](https://img.shields.io/badge/status-planned-blue?style=flat-square)

> Quality gates in your pipeline, not just your terminal

```mermaid
graph LR
    PR[Pull Request] --> GA[GitHub Actions]
    GA --> QG{Quality Gates}
    QG --> CR[Code Review]
    QG --> SA[Security Audit]
    QG --> TE[Tests]
    CR --> V{Verdict}
    SA --> V
    TE --> V
    V -->|PASS| M[Merge]
    V -->|CRITICAL| B[Block]
```

- [ ] **GitHub Actions** — run quality gates on PR
- [ ] **GitLab CI** — equivalent template
- [ ] **Pre-commit hook** — local quality check before push
- [ ] **PR comment bot** — post agent verdicts as review comments
- [ ] **Status checks** — block merge on CRITICAL verdicts
- [ ] **Diff-only mode** — review only changed lines

<br>

## v3.2 — Reporting & Analytics ![Status](https://img.shields.io/badge/status-planned-blue?style=flat-square)

> See how your code quality evolves

- [ ] HTML report — visual quality report with charts
- [ ] Trend tracking — quality score over time
- [ ] Verdict history — track recurring issues
- [ ] Agent performance metrics — turns, time, accuracy
- [ ] Dashboard — web UI for monitoring runs

<br>

## Ongoing

> Continuous improvements across all versions

- [ ] Turn budget optimization (measure actual usage per phase)
- [ ] More MCP servers — Prettier, Biome, Ruff, mypy, Clippy
- [ ] Windows-native improvements (PowerShell compatibility)
- [ ] Agent prompt compression (reduce tokens without losing quality)
- [ ] Benchmark suite — reproducible test cases for agent quality
- [ ] Multilingual docs — Russian, Chinese, Japanese, Spanish

<br>

---

<p align="center">
  <b>Want to contribute?</b><br>
  Pick any item, <a href="https://github.com/Mationetap/claude-agents-forge/issues">open an issue</a>, submit a PR.<br>
  See <a href="CONTRIBUTING.md">CONTRIBUTING.md</a> for quality standards.
</p>
