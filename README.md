# ai-first-pipeline

Template de pipeline AI-first para desenvolvimento de software com SDD (Spec-Driven Development), TDD e DevSecOps integrados. Publique no GitHub e use como ponto de partida para qualquer projeto.

Zero dependências de runtime. Apenas Markdown versionado.

---

## Quickstart

**1. Clone ou use este template**

```
gh repo create meu-projeto --template jonatas/ai-first-pipeline
```

**2. Defina a constituição do projeto**

```
/speckit.constitution
```

**3. Escreva a spec da primeira feature**

```
/speckit.specify
```

**4. Responda as perguntas de clarificação e gere o plano**

```
/speckit.clarify
/speckit.plan
/speckit.tasks
```

**5. Implemente guiado pelas tasks**

Cada task em `specs/<FEATURE>/tasks.md` é uma unidade de trabalho autônoma. Execute com seu agente preferido.

---

## Pipeline

```
SPEC → CLARIFY → PLAN → TASKS → IMPLEMENT → ANALYZE
  ↑                                              |
  └──────────────────────────────────────────────┘
```

| Fase | Comando | Agente | Saída |
|------|---------|--------|-------|
| Especificar | `/speckit.specify` | Sonnet | `specs/<ID>/spec.md` |
| Clarificar | `/speckit.clarify` | Sonnet | `specs/<ID>/perguntas-respondidas.md` |
| Planejar | `/speckit.plan` | Sonnet | `specs/<ID>/plan.md` |
| Detalhar tasks | `/speckit.tasks` | Haiku | `specs/<ID>/tasks.md` |
| Analisar | `/speckit.analyze` | Sonnet | `specs/<ID>/analysis-report.md` |

---

## Estrutura

```
ai-first-pipeline/
├── README.md               ← você está aqui
├── constitution.md         ← princípios e regras do projeto
├── CLAUDE.md               ← instruções e model routing
├── AGENTS.md               ← definição dos agentes
├── CONTRIBUTING.md         ← como contribuir
├── CHANGELOG.md            ← histórico de versões
├── .claude/commands/       ← comandos /speckit.*
├── specs/                  ← specs de features
│   └── EXAMPLE-001/        ← exemplo: rate limiting
└── docs/adr/               ← Architecture Decision Records
```

---

## Exemplo incluído

`specs/EXAMPLE-001/` contém um exemplo completo de spec para **rate limiting por usuário** (API REST, sliding window, Redis). Use como referência de formato e profundidade esperada.

---

## Princípios

- **Spec before code** — nenhuma linha de código sem spec aprovada
- **Questions before assumptions** — clarificar é mais barato que refatorar
- **Docs as source of truth** — o Markdown versiona as decisões, o código implementa
- **AI-native** — cada fase é projetada para ser executada por um agente

Veja `constitution.md` para o contrato completo do projeto.
