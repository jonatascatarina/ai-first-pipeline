# Changelog

Todas as mudanças notáveis neste projeto são documentadas aqui.

O formato segue [Keep a Changelog](https://keepachangelog.com/pt-BR/1.0.0/).

---

## [Não lançado]

### Adicionado
- Comando `/pr-checklist` (FEATURE-002): gera checklist de revisão categorizado (Segurança, Testes, Arquitetura, Documentação) a partir do título e descrição de um PR. Detecção de contexto por keywords, itens obrigatórios fixos, output copiável em bloco Markdown.
- Comando `/speckit.changelog` (FEATURE-003): gera seção de changelog automaticamente a partir do git log cruzado com specs referenciadas nos commits. Classifica por prefixo convencional, agrupa múltiplos commits por spec e exige confirmação antes de escrever no CHANGELOG.md.

### Planejado
- Comando `/speckit.review` para revisão de código guiada por spec
- Exemplo EXAMPLE-002 cobrindo autenticação OAuth2
- Integração com GitHub Issues via template de spec

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
