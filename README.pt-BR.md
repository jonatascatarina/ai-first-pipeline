# ai-first-pipeline

[![Release](https://img.shields.io/badge/release-v4.4.0-blue)](https://github.com/jonatascatarina/ai-first-pipeline/releases/tag/v4.4.0) [![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT) [![Last commit](https://img.shields.io/github/last-commit/jonatascatarina/ai-first-pipeline)](https://github.com/jonatascatarina/ai-first-pipeline/commits/main) [![Stars](https://img.shields.io/github/stars/jonatascatarina/ai-first-pipeline?style=social)](https://github.com/jonatascatarina/ai-first-pipeline/stargazers) [🇺🇸 Read in English](./README.md)

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
/sdd.constitution
```

**3. Escreva a spec da primeira feature**

```
/sdd.specify
```

**4. Responda as perguntas de clarificação e gere o plano**

```
/sdd.clarify
/sdd.plan
/sdd.tasks
```

**5. Implemente guiado pelas tasks**

Cada task em `specs/<FEATURE>/tasks.md` é uma unidade de trabalho autônoma. Execute com seu agente preferido.

**6. (Opcional) Integre com GitHub Issues**

Para times que usam o issue tracker do GitHub:
- Abra Issues usando o template em `.github/ISSUE_TEMPLATE/feature-spec.md`
- Use `/sdd.issue` para publicar specs como Issues ou converter Issues em specs

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

### Camada 1 — SDD

| Passo | Comando | Agente | Saída |
|-------|---------|--------|-------|
| 1. Especificar | `/sdd.specify` | Sonnet | `specs/<ID>/spec.md` |
| 2. Clarificar | `/sdd.clarify` | Sonnet | `specs/<ID>/perguntas-respondidas.md` |
| 3. Planejar | `/sdd.plan` | Sonnet | `specs/<ID>/plan.md` |
| 4. Detalhar tasks | `/sdd.tasks` | Haiku | `specs/<ID>/tasks.md` |

### Camada 2 — TDD

| Passo | Agente | Saída |
|-------|--------|-------|
| 5. Escrever testes | `tdd-test-writer` | Suíte de testes baseada na spec |
| 6. Implementar | `tdd-implementer` | Código mínimo para testes passarem |
| 7. Refatorar | `refactor` | Código limpo, testes ainda verdes |

### Camada 3 — OPS

| Passo | Comando / Agente | Saída |
|-------|-----------------|-------|
| 8. Analisar | `/sdd.analyze` | `specs/<ID>/analysis-report.md` |
| 9. Auditar segurança | `security-auditor` | Relatório com BLOCKERs |
| 10. Revisar | `/sdd.review` | Comentário de PR copiável |
| 11. Quality gate | `quality-gate.yml` | lint → test → sec-scan → build |
| 12. Changelog | `/sdd.changelog` | `CHANGELOG.md` atualizado |
| 13. Standup | `/sdd.standup` | Resumo para o time |

Veja `docs/tdd-ops-guide.md` para o guia completo de ativação das camadas 2 e 3.

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
├── .claude/
│   ├── commands/           ← comandos /sdd.*
│   ├── agents/             ← agentes: TDD, refactor, security-auditor, onboarding
│   └── hooks/              ← pre-tool-use: regras de proteção de arquivos
├── .github/
│   ├── ISSUE_TEMPLATE/     ← template de Issue para features (SDD)
│   └── workflows/
│       └── quality-gate.yml ← pipeline CI: lint → test → sec-scan → build (configure antes de ativar)
├── docs/
│   ├── adr/                ← Architecture Decision Records
│   └── tdd-ops-guide.md    ← guia das camadas TDD e OPS
├── specs/                  ← specs de features e epics
│   ├── EXAMPLE-001/        ← exemplo: rate limiting (sliding window + Redis)
│   └── EXAMPLE-002/        ← exemplo: autenticação OAuth2 com GitHub (PKCE)
└── docs/adr/               ← Architecture Decision Records
```

---

## O que está em `specs/`

O diretório `specs/` contém dois tipos de artefato:

**Exemplos didáticos (`EXAMPLE-*`)** — specs fictícias de produtos reais, criadas para demonstrar o formato e a profundidade esperada. Use como referência ao escrever suas próprias specs. Podem ser deletadas ao iniciar um projeto real.

- `EXAMPLE-001/` — rate limiting por usuário (API REST, sliding window, Redis)
- `EXAMPLE-002/` — autenticação OAuth2 com GitHub (Authorization Code Flow + PKCE)

**Specs do próprio pipeline (`FEATURE-*`)** — este projeto foi desenvolvido usando o próprio processo que documenta. As specs `FEATURE-002` a `FEATURE-006` registram como cada comando `/sdd.*` foi especificado, planejado e implementado. São o histórico real de desenvolvimento do template — e também servem como exemplos de como o pipeline funciona na prática.

---

## Comparação

| Recurso | ai-first-pipeline | Spec Kit | Kiro |
|---------|-------------------|----------|------|
| Zero dependências | ✅ Markdown only | ❌ Python CLI | ❌ IDE completa |
| Agnóstico de agente | ✅ | ✅ | ❌ Bloqueado |
| Detecção de drift | ✅ `/sdd.drift` | ❌ | ❌ |
| Modo lite | ✅ `/sdd.lite` | ❌ | ❌ |
| Event hooks | ❌ | ❌ | ✅ |
| Docs bilíngues | ✅ EN + PT-BR | ❌ EN | ❌ EN |

---

## Princípios

- **Spec before code** — nenhuma linha de código sem spec aprovada
- **Questions before assumptions** — clarificar é mais barato que refatorar
- **Docs as source of truth** — o Markdown versiona as decisões, o código implementa
- **AI-native** — cada fase é projetada para ser executada por um agente

Veja `constitution.md` para o contrato completo do projeto.
