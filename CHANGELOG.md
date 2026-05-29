# Changelog

Todas as mudanças notáveis neste projeto são documentadas aqui.

O formato segue [Keep a Changelog](https://keepachangelog.com/pt-BR/1.0.0/).

---

## [Não lançado]

---

## [4.5.0] — 2026-05-29

### Alterado
- `README.md` e `README.pt-BR.md`: seção `## Prerequisites` (tabela simples) substituída por `## Quickstart` completo com 4 passos numerados — pré-requisitos com links, criação do repositório, abertura do agente, definição da constituição e início da primeira feature.
- Comando `gh repo create` corrigido: adicionada flag `--public` obrigatória (repositórios privados não suportam templates no GitHub); nota sobre alternativa `--private` adicionada.
- Passo 3 do Quickstart: descrição atualizada para blockquote explicando que o agente lê `constitution.md` e define os princípios imutáveis do projeto.
- Passo 4 do Quickstart renomeado para "Write your first feature spec" com blockquote descritivo (OUTCOMES, SCOPE, BEHAVIOR em notação EARS).
- Passo 5 expandido: cada comando com seu próprio bloco e blockquote — `/sdd.clarify` resolve perguntas em aberto, `/sdd.plan` define estrutura e arquitetura, `/sdd.tasks` decompõe em tasks com código de risco por cor (🟢🟡🔴).
- Passo 6 adicionado: "Implement guided by tasks" — instrução de como passar cada task ao agente uma por vez.
- `/sdd.lite` movido para nota ao final do Quickstart como atalho opcional para features pequenas ou triviais.

---

## [4.4.0] — 2026-05-27

### Adicionado
- `.claudeignore`: exclui do contexto do agente arquivos de baixo sinal (`.git/`, `*.lock`, `node_modules/`, `CHANGELOG.md`, `*.log`, `releases/`, `specs/*/drafts/`). Reduz tokens carregados por sessão.
- Model routing declarado no cabeçalho de todos os 15 comandos em `.claude/commands/`: `**Modelo recomendado:**` + `**Justificativa:**` em uma linha. Elimina consulta ao `CLAUDE.md` para saber qual modelo usar.

### Alterado
- `CLAUDE.md` enxugado de 107 para 69 linhas: seção "Comportamento Esperado" removida (detalhes movidos para `constitution.md` com nota de leitura sob demanda), coluna "Modelo" adicionada na tabela de comandos, seção "Referências — Leia Sob Demanda" adicionada com tabela de quando ler cada arquivo de suporte.
- `sdd.tasks.md`: removida menção de modelo inline no texto do agente (substituída pelo cabeçalho padronizado).

---

## [4.3.0] — 2026-05-26

### Adicionado
- `specs/TDD-OPS-LAYER/drift-report.md`: drift report da camada TDD + OPS. Veredicto ALINHADO — todos os 7 arquivos exigidos conformes com a spec original. `onboarding.md` identificado como único drift, benéfico e sem conflito.
- `specs/FEATURE-008/spec.md` e `specs/FEATURE-008/lite.md`: formalização retroativa do OnboardingAgent como cidadão de primeira classe do pipeline. Resolve drift documentado no relatório da camada TDD + OPS.
- Seção `OnboardingAgent` adicionada em `AGENTS.md` com modelo, fase, responsabilidades, input, output e restrições no padrão dos demais agentes.

---

## [4.2.0] — 2026-05-26

### Adicionado
- Badges de status nos READMEs (EN e PT-BR): licença MIT, último commit e GitHub stars via shields.io. Spec em `specs/FEATURE-007/lite.md` (fluxo `/sdd.lite`).
- `specs/FEATURE-002/drift-report.md`: primeiro drift report do projeto, gerado com `/sdd.drift`. Veredicto ALINHADO — 5/6 critérios confirmados, CA-4 pendente de validação.

### Alterado
- `specs/FEATURE-002/spec.md` atualizado para refletir drifts benignos: fallback de seção vazia documentado em EARS-1, `test-results.md` registrado como artefato opcional, CA-4 marcado como pendente de validação.

---

## [4.1.0] — 2026-05-26

### Alterado
- Migração completa de prefixo: os 6 comandos restantes com prefixo `speckit.` renomeados para `sdd.`: `sdd.review`, `sdd.changelog`, `sdd.standup`, `sdd.issue`, `sdd.adr`, `sdd.epic`. Todos os 15 comandos do pipeline agora usam o prefixo `sdd.*`.
- `lint.yml` corrigido: paths de verificação atualizados de `speckit.*` para `sdd.*` (os antigos causavam falha no CI por referenciar arquivos inexistentes)
- `CONTRIBUTING.md`, `README.md`, `README.pt-BR.md`, `docs/tdd-ops-guide.md`: referências `speckit.*` restantes removidas

---

## [4.0.0] — 2026-05-26

### Breaking Changes
- Renomeados 6 comandos do prefixo `speckit.` para `sdd.`: `sdd.constitution`, `sdd.specify`, `sdd.clarify`, `sdd.plan`, `sdd.tasks`, `sdd.analyze`. Comandos de output e utilitários (`speckit.review`, `speckit.changelog`, `speckit.standup`, `speckit.issue`, `speckit.adr`, `speckit.epic`) mantêm o prefixo `speckit.`.

### Adicionado
- `README.md` em inglês com tagline de posicionamento, tabela comparativa e link para versão em português
- `README.pt-BR.md` com conteúdo completo em português e link para versão em inglês
- Comando `/sdd.lite`: pipeline enxuto para features pequenas (specify + plan + tasks em uma execução). Output único: `specs/<ID>/lite.md` com outcomes, até 3 tasks e risco geral. Sem clarify, sem analyze.
- Comando `/sdd.drift`: detecta divergência entre spec e implementação cruzando critérios de aceite com commits git. Classifica por status (✅ Implementado / ⏳ Não implementado / ⚠️ Fora da spec). Gera `specs/<ID>/drift-report.md`.
- Tabela comparativa em ambos os READMEs: ai-first-pipeline vs Spec Kit vs Kiro

### Alterado
- `CLAUDE.md`, `AGENTS.md`, `CONTRIBUTING.md`, `docs/tdd-ops-guide.md`: todas as referências a comandos renomeados atualizadas

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
