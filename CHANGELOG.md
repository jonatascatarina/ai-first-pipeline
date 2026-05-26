# Changelog

Todas as mudanças notáveis neste projeto são documentadas aqui.

O formato segue [Keep a Changelog](https://keepachangelog.com/pt-BR/1.0.0/).

---

## [Não lançado]

---

## [3.0.0] — 2026-05-26

### Adicionado
- Comando `/speckit.adr`: guia a criação interativa de ADRs com contexto, decisão, alternativas consideradas e consequências. Status inicial `Em revisão`, imutável após aprovação (Artigo VII.3).
- Comando `/speckit.epic`: decompõe iniciativas grandes em features com mapa de dependências, estimativas de complexidade (P/M/G), sequência de entrega por fases e caminho crítico. Gera `specs/EPIC-NNN/epic.md` e stubs opcionais de features filho.
- Agente `onboarding`: gera resumo de contexto completo do projeto para um agente novo — versão atual, features com status inferido, ADRs, bloqueadores e próximos passos recomendados. Zero modificação de arquivos.

### Pipeline completo em v3.0.0

13 comandos e 5 agentes especializados:

| Tipo | Nome | Responsabilidade |
|------|------|-----------------|
| Comando | `/speckit.constitution` | Define a constituição do projeto |
| Comando | `/speckit.specify` | Cria spec de feature |
| Comando | `/speckit.clarify` | Gera perguntas de clarificação |
| Comando | `/speckit.plan` | Cria plano de implementação |
| Comando | `/speckit.tasks` | Decompõe plano em tasks |
| Comando | `/speckit.analyze` | Analisa conformidade com spec |
| Comando | `/speckit.review` | Revisão guiada por spec |
| Comando | `/speckit.changelog` | Gera seção de changelog |
| Comando | `/speckit.standup` | Gera resumo de standup |
| Comando | `/speckit.issue` | Integração com GitHub Issues |
| Comando | `/speckit.adr` | Cria Architecture Decision Record |
| Comando | `/speckit.epic` | Planeja iniciativas multi-feature |
| Comando | `/pr-checklist` | Checklist de revisão de PR |
| Agente | `tdd-test-writer` | Escreve testes antes da implementação |
| Agente | `tdd-implementer` | Implementa código para testes passarem |
| Agente | `refactor` | Limpa código com testes verdes |
| Agente | `security-auditor` | Audita segurança antes do merge |
| Agente | `onboarding` | Contextualiza agentes novos |

---

## [2.1.0] — 2026-05-26

### Adicionado
- Camada TDD com quatro agentes especializados em `.claude/agents/`: `tdd-test-writer` (traduz critérios de aceite em testes), `tdd-implementer` (código mínimo para testes passarem), `refactor` (limpa código com testes verdes), `security-auditor` (OWASP Top 10, LGPD, CVEs)
- Camada OPS com `quality-gate.yml` em `.github/workflows/`: pipeline documentado de lint → test → security-scan → build com placeholders configuráveis
- Hook `pre-tool-use.md` em `.claude/hooks/`: padrão PERMITIR/ALERTAR/BLOQUEAR para proteger arquivos críticos (`constitution.md`, ADRs, workflows, secrets)
- Guia `docs/tdd-ops-guide.md`: fluxo completo dos 13 passos do pipeline, sequência de ativação de agentes, regras de avanço entre camadas e tabela de isolamento de responsabilidades

### Alterado
- `CLAUDE.md`: tabela de model routing atualizada com fases TDD/OPS; nova seção de agentes especializados com arquivo e fase de cada um
- `README.md`: pipeline reorganizado em 3 camadas (SDD → TDD → OPS) com 13 passos; estrutura de diretórios expandida

---

## [2.0.0] — 2026-05-26

### Adicionado
- Comando `/speckit.review` (FEATURE-005): conduz revisão de código guiada por spec junto ao revisor humano. Apresenta cada critério de aceite com perguntas específicas, coleta avaliações (OK/PARCIAL/AUSENTE) e gera comentário de revisão formatado para colar no PR com veredicto (APROVADO/BLOQUEADO/CONDICIONALMENTE APROVADO).
- Integração bidirecional com GitHub Issues (FEATURE-006): template de Issue estruturado em `.github/ISSUE_TEMPLATE/feature-spec.md` e comando `/speckit.issue` que publica specs como Issues ou converte Issues em scaffolds de spec.
- Exemplo `EXAMPLE-002`: spec completa de autenticação OAuth2 com GitHub (Authorization Code Flow + PKCE). Cobre fluxo frontend/backend, migration de banco, análise de riscos de segurança (client_secret, CSRF, open redirect, replay de code) e 10 tasks incluindo task SEC prioritária.

### Pipeline completo em v2.0.0

11 comandos disponíveis:

| Comando | Descrição |
|---------|-----------|
| `/speckit.constitution` | Define ou atualiza a constituição do projeto |
| `/speckit.specify` | Cria spec detalhada de uma feature |
| `/speckit.clarify` | Gera perguntas de clarificação para uma spec |
| `/speckit.plan` | Cria plano de implementação a partir da spec |
| `/speckit.tasks` | Decompõe o plano em tasks executáveis |
| `/speckit.analyze` | Analisa código ou spec e gera relatório |
| `/speckit.changelog` | Gera seção de changelog a partir do git log |
| `/speckit.standup` | Gera resumo de standup diário |
| `/speckit.review` | Revisão de código guiada por spec |
| `/speckit.issue` | Integração bidirecional com GitHub Issues |
| `/pr-checklist` | Gera checklist de revisão de PR |

---

## [1.1.0] — 2026-05-26

### Adicionado
- **PR Review Checklist Generator** (FEATURE-002): comando `/pr-checklist` que recebe título e descrição de PR e gera checklist Markdown categorizado com itens específicos ao contexto, pronto para colar no comentário de revisão
- **Gerador Automático de Changelog** (FEATURE-003): comando `/speckit.changelog` que lê o git log cruzado com specs referenciadas nos commits, classifica por prefixo convencional, agrupa múltiplos commits por spec e insere nova seção versionada no CHANGELOG.md após confirmação
- Estrutura inicial do template ai-first-pipeline com constitution, CLAUDE.md, AGENTS.md, 6 comandos `/speckit.*` e exemplo EXAMPLE-001 de rate limiting

### Interno
- LICENSE MIT adicionada, typo no quickstart do README corrigido
- Testes manuais do `/pr-checklist` documentados em `test-results.md`

---

## [1.0.0] — 2026-05-26

### Adicionado
- Estrutura inicial do template ai-first-pipeline
- `constitution.md` com oito artigos de governança
- `CLAUDE.md` com tabela de model routing (Haiku / Sonnet) e instruções de comportamento
- `AGENTS.md` definindo cinco agentes especializados por fase do pipeline
- Seis comandos `/speckit.*`:
  - `/speckit.constitution` — definir constituição do projeto
  - `/speckit.specify` — escrever spec de feature
  - `/speckit.clarify` — gerar perguntas de clarificação
  - `/speckit.plan` — criar plano de implementação
  - `/speckit.tasks` — decompor plano em tasks
  - `/speckit.analyze` — analisar conformidade com spec
- Exemplo completo `EXAMPLE-001`: rate limiting por usuário (API REST, sliding window, Redis)
  - `spec.md` com critérios de aceite e não-escopo
  - `perguntas-respondidas.md` com 8 perguntas e respostas
  - `plan.md` com abordagem técnica e análise de riscos
  - `tasks.md` com 12 tasks numeradas e estimativas
  - `analysis-report.md` com findings e recomendações
- `docs/adr/ADR-001-exemplo.md` demonstrando formato de Architecture Decision Record
- `.github/workflows/lint.yml` para validação de Markdown em PRs
- `CONTRIBUTING.md` com processo de contribuição
