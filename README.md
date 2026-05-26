# ai-first-pipeline

[![Release](https://img.shields.io/badge/release-v4.2.0-blue)](https://github.com/jonatascatarina/ai-first-pipeline/releases/tag/v4.2.0) [![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT) [![Last commit](https://img.shields.io/github/last-commit/jonatascatarina/ai-first-pipeline)](https://github.com/jonatascatarina/ai-first-pipeline/commits/main) [![Stars](https://img.shields.io/github/stars/jonatascatarina/ai-first-pipeline?style=social)](https://github.com/jonatascatarina/ai-first-pipeline/stargazers) [🇧🇷 Leia em português](./README.pt-BR.md)

Lightweight SDD pipeline for AI coding agents.
Zero runtime dependencies, agent-agnostic, Markdown-only.

---

## Quickstart

**1. Clone or use this template**

```
gh repo create my-project --template jonatascatarina/ai-first-pipeline
```

**2. Define your project constitution**

```
/sdd.constitution
```

**3. Write your first feature spec**

```
/sdd.specify
```

**4. Clarify, plan and break into tasks**

```
/sdd.clarify
/sdd.plan
/sdd.tasks
```

**5. Implement guided by tasks**

Each task in `specs/<FEATURE>/tasks.md` is a self-contained unit of work. Run it with your preferred agent.

**6. (Optional) Integrate with GitHub Issues**

- Open Issues using `.github/ISSUE_TEMPLATE/feature-spec.md`
- Use `/sdd.issue` to publish specs as Issues or convert Issues into spec scaffolds

---

## Pipeline

```
 1. SPEC      2. CLARIFY   3. PLAN      4. TASKS
      ↓             ↓           ↓            ↓
 5. WRITE     6. IMPLEMENT  7. REFACTOR  8. ANALYZE
   TESTS
      ↓             ↓           ↓            ↓
 9. SEC AUDIT 10. REVIEW  11. QUALITY  12. CHANGELOG
                              GATE
                                           ↓
                                      13. STANDUP
```

### Layer 1 — SDD

| Step | Command | Model | Output |
|------|---------|-------|--------|
| 1. Specify | `/sdd.specify` | Sonnet | `specs/<ID>/spec.md` |
| 2. Clarify | `/sdd.clarify` | Sonnet | `specs/<ID>/perguntas-respondidas.md` |
| 3. Plan | `/sdd.plan` | Sonnet | `specs/<ID>/plan.md` |
| 4. Tasks | `/sdd.tasks` | Haiku | `specs/<ID>/tasks.md` |

### Layer 2 — TDD

| Step | Agent | Output |
|------|-------|--------|
| 5. Write tests | `tdd-test-writer` | Test suite based on acceptance criteria |
| 6. Implement | `tdd-implementer` | Minimal code to make tests pass |
| 7. Refactor | `refactor` | Clean code, tests still green |

### Layer 3 — OPS

| Step | Command / Agent | Output |
|------|----------------|--------|
| 8. Analyze | `/sdd.analyze` | `specs/<ID>/analysis-report.md` |
| 9. Security audit | `security-auditor` | Report with BLOCKERs |
| 10. Review | `/sdd.review` | PR review comment (copy-paste ready) |
| 11. Quality gate | `quality-gate.yml` | lint → test → sec-scan → build |
| 12. Changelog | `/sdd.changelog` | Updated `CHANGELOG.md` |
| 13. Standup | `/sdd.standup` | Team standup summary |

See `docs/tdd-ops-guide.md` for the complete activation guide for layers 2 and 3.

---

## All Commands

| Command | Description |
|---------|-------------|
| `/sdd.constitution` | Define or update the project constitution |
| `/sdd.specify` | Write a detailed feature spec |
| `/sdd.clarify` | Generate clarification questions for a spec |
| `/sdd.plan` | Create an implementation plan from the spec |
| `/sdd.tasks` | Break the plan into executable tasks |
| `/sdd.analyze` | Analyze spec or code conformance and produce a report |
| `/sdd.lite` | Lightweight pipeline for small/trivial features (specify + plan + tasks in one run) |
| `/sdd.drift` | Detect divergence between spec and implementation |
| `/sdd.review` | Spec-guided code review with human reviewer |
| `/sdd.changelog` | Generate changelog section from git log |
| `/sdd.standup` | Generate daily standup summary |
| `/sdd.issue` | Bidirectional GitHub Issues integration |
| `/sdd.adr` | Interactively create an Architecture Decision Record |
| `/sdd.epic` | Decompose a large initiative into features with dependency map |
| `/pr-checklist` | Generate PR review checklist from title and description |

---

## Structure

```
ai-first-pipeline/
├── README.md               ← you are here (English)
├── README.pt-BR.md         ← Portuguese version
├── constitution.md         ← project principles and governance rules
├── CLAUDE.md               ← agent instructions and model routing
├── AGENTS.md               ← agent definitions
├── CONTRIBUTING.md         ← how to contribute
├── CHANGELOG.md            ← version history
├── .claude/
│   ├── commands/           ← /sdd.* commands
│   ├── agents/             ← TDD, refactor, security-auditor, onboarding agents
│   └── hooks/              ← pre-tool-use: file protection rules
├── .github/
│   ├── ISSUE_TEMPLATE/     ← structured Issue template for features
│   └── workflows/
│       └── quality-gate.yml ← CI pipeline template (configure before activating)
├── docs/
│   ├── adr/                ← Architecture Decision Records
│   └── tdd-ops-guide.md    ← TDD and OPS layers guide
└── specs/                  ← feature specs and epics
    ├── EXAMPLE-001/        ← example: rate limiting (sliding window + Redis)
    └── EXAMPLE-002/        ← example: OAuth2 with GitHub (PKCE)
```

---

## What's in `specs/`

**Didactic examples (`EXAMPLE-*`)** — complete specs for real-world scenarios, built to demonstrate the expected format and depth. Use as reference when writing your own specs. Safe to delete when starting a real project.

- `EXAMPLE-001/` — per-user rate limiting (REST API, sliding window, Redis)
- `EXAMPLE-002/` — OAuth2 authentication with GitHub (Authorization Code Flow + PKCE)

**Pipeline's own specs (`FEATURE-*`)** — this project was built using the process it documents. Specs `FEATURE-002` to `FEATURE-006` record how each command was specified, planned and implemented. Real development history — and additional examples of the pipeline in action.

---

## Comparison

| Feature | ai-first-pipeline | Spec Kit | Kiro |
|---------|-------------------|----------|------|
| Zero dependencies | ✅ Markdown only | ❌ Python CLI | ❌ Full IDE |
| Agent-agnostic | ✅ | ✅ | ❌ Locked |
| Drift detection | ✅ `/sdd.drift` | ❌ | ❌ |
| Lite mode | ✅ `/sdd.lite` | ❌ | ❌ |
| Event hooks | ❌ | ❌ | ✅ |
| Bilingual docs | ✅ EN + PT-BR | ❌ EN only | ❌ EN only |

---

## Principles

- **Spec before code** — no production code without an approved spec
- **Questions before assumptions** — clarifying is cheaper than refactoring
- **Docs as source of truth** — Markdown versions decisions, code implements them
- **AI-native** — every phase is designed to be executed by an agent

See `constitution.md` for the full project contract.
