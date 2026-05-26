# Changelog

Todas as mudanças notáveis neste projeto são documentadas aqui.

O formato segue [Keep a Changelog](https://keepachangelog.com/pt-BR/1.0.0/).

---

## [Não lançado]

### Adicionado
- Comando `/pr-checklist` (FEATURE-002): gera checklist de revisão categorizado (Segurança, Testes, Arquitetura, Documentação) a partir do título e descrição de um PR. Detecção de contexto por keywords, itens obrigatórios fixos, output copiável em bloco Markdown.
- Comando `/speckit.changelog` (FEATURE-003): gera seção de changelog automaticamente a partir do git log cruzado com specs referenciadas nos commits. Classifica por prefixo convencional, agrupa múltiplos commits por spec e exige confirmação antes de escrever no CHANGELOG.md.
- Comando `/speckit.standup` (FEATURE-004): gera mensagem de standup diário pronta para colar no Slack/Discord, traduzindo commits técnicos para linguagem de negócio. Trata segunda-feira automaticamente com `--since="last friday"`.
- Comando `/speckit.review` (FEATURE-005): conduz revisão de código guiada por spec junto ao revisor humano. Apresenta cada critério de aceite com perguntas específicas, coleta avaliações (OK/PARCIAL/AUSENTE) e gera comentário de revisão formatado para colar no PR com veredicto (APROVADO/BLOQUEADO/CONDICIONALMENTE APROVADO).

### Planejado
- Exemplo EXAMPLE-002 cobrindo autenticação OAuth2
- Integração com GitHub Issues via template de spec

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
