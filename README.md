# ai-first-pipeline

[![Release](https://img.shields.io/badge/release-v2.0.0-blue)](https://github.com/jonatascatarina/ai-first-pipeline/releases/tag/v2.0.0)

Template de pipeline AI-first para desenvolvimento de software com SDD (Spec-Driven Development), TDD e DevSecOps integrados. Publique no GitHub e use como ponto de partida para qualquer projeto.

Zero dependências de runtime. Apenas Markdown versionado.

---

## Quickstart

**1. Clone ou use este template**

```
gh repo create meu-projeto --template jonatascatarina/ai-first-pipeline
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

**6. (Opcional) Integre com GitHub Issues**

Para times que usam o issue tracker do GitHub:
- Abra Issues usando o template em `.github/ISSUE_TEMPLATE/feature-spec.md`
- Use `/speckit.issue` para publicar specs como Issues ou converter Issues em specs

---

## Pipeline

```
SPEC → CLARIFY → PLAN → TASKS → IMPLEMENT → ANALYZE → REVIEW
  ↑                                                         |
  └─────────────────────────────────────────────────────────┘
```

| Fase | Comando | Agente | Saída |
|------|---------|--------|-------|
| Especificar | `/speckit.specify` | Sonnet | `specs/<ID>/spec.md` |
| Clarificar | `/speckit.clarify` | Sonnet | `specs/<ID>/perguntas-respondidas.md` |
| Planejar | `/speckit.plan` | Sonnet | `specs/<ID>/plan.md` |
| Detalhar tasks | `/speckit.tasks` | Haiku | `specs/<ID>/tasks.md` |
| Analisar | `/speckit.analyze` | Sonnet | `specs/<ID>/analysis-report.md` |
| Revisar | `/speckit.review` | Sonnet | Comentário de PR copiável |

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
├── .github/
│   └── ISSUE_TEMPLATE/     ← template de Issue para features (SDD)
├── specs/                  ← specs de features
│   ├── EXAMPLE-001/        ← exemplo: rate limiting (sliding window + Redis)
│   └── EXAMPLE-002/        ← exemplo: autenticação OAuth2 com GitHub (PKCE)
└── docs/adr/               ← Architecture Decision Records
```

---

## Exemplos incluídos

`specs/EXAMPLE-001/` — spec completa para **rate limiting por usuário** (API REST, sliding window, Redis). Demonstra: requisitos funcionais e não-funcionais, critérios de aceite verificáveis, perguntas de clarificação, plano técnico com decisões justificadas e tasks com estimativa.

`specs/EXAMPLE-002/` — spec completa para **autenticação OAuth2 com GitHub** (Authorization Code Flow + PKCE). Demonstra: fluxo de segurança multicamada, modelagem de dados com migration, contratos de interface frontend/backend, análise de riscos de segurança e tasks com task SEC prioritária.

---

## Princípios

- **Spec before code** — nenhuma linha de código sem spec aprovada
- **Questions before assumptions** — clarificar é mais barato que refatorar
- **Docs as source of truth** — o Markdown versiona as decisões, o código implementa
- **AI-native** — cada fase é projetada para ser executada por um agente

Veja `constitution.md` para o contrato completo do projeto.
