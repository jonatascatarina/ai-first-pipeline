# Plano de Implementação — FEATURE-004

## Resumo Técnico

Feature implementada como um único arquivo `.claude/commands/speckit.standup.md`. O agente executa um comando git para obter os commits de ontem, coleta contexto adicional do usuário em até 3 perguntas, traduz os commits para linguagem de negócio, agrupa temas relacionados e entrega o standup formatado dentro de um bloco copiável. Zero dependências além do git.

## Abordagem

### Fluxo Principal

```
/speckit.standup
  │
  ├── Executa: git log --since="yesterday" --until="today" --oneline --no-merges
  │     ├── Commits encontrados → processa
  │     └── Sem commits → pergunta "O que você fez ontem?"
  │
  ├── Pergunta 1: "Há algo além dos commits para incluir no Ontem?
  │              (reuniões, revisões, conversas)" — opcional
  │
  ├── Pergunta 2: "O que você vai fazer hoje?" — obrigatória
  │
  ├── Pergunta 3: "Há algum bloqueio?" — opcional (padrão: Nenhum)
  │
  ├── Processa commits:
  │     ├── Remove prefixos (feat:, fix:, chore:, etc.)
  │     ├── Traduz jargão técnico → linguagem de negócio
  │     ├── Agrupa commits relacionados ao mesmo tema
  │     └── Limita a 5 itens (prioriza feat: e fix:)
  │
  ├── Monta header com dia da semana e data em português
  │
  └── Entrega bloco copiável formatado para Slack/Discord
```

### Regras de Tradução de Commits

O prompt inclui uma tabela de exemplos de tradução para calibrar o modelo:

| Commit original | Tradução esperada |
|----------------|------------------|
| `feat: implement sliding window lua script` | "Implementei o algoritmo de controle de tráfego da API" |
| `fix: token expiry returning 500 instead of 401` | "Corrigi bug onde token expirado causava erro 500" |
| `chore: add LICENSE and fix README typo` | "Adicionei licença MIT e corrigi erro no README" |
| `test: add concurrency tests for rate limit` | "Escrevi testes de concorrência para o rate limiting" |
| `docs: update CHANGELOG for v1.1.0` | "Atualizei o changelog da versão 1.1.0" |

### Formato de Output

```
🗓 Standup — Terça-feira, 26/05/2026

*Ontem:*
• Implementei autenticação JWT no endpoint /api/users
• Participei de reunião de refinamento do backlog

*Hoje:*
• Implementar testes de integração para o novo middleware

*Bloqueios:*
• Nenhum
```

### Componente único

**Arquivo:** `.claude/commands/speckit.standup.md`

## Dependências

- Git instalado e repositório com histórico de commits
- Nenhuma outra dependência

## Riscos

### Riscos de qualidade
- **BAIXO — Commits vagos:** `fix: bug` não tem contexto para tradução útil. O agente usa o que tem e produz "Corrigi um bug". Qualidade depende da disciplina de commit do desenvolvedor
- **BAIXO — Agrupamento incorreto:** O modelo pode agrupar commits não relacionados. Few-shot no prompt mitiga, mas não elimina

### Riscos de fuso horário
- **BAIXO — `--since="yesterday"` usa o fuso do sistema.** Em máquinas com UTC, "yesterday" pode capturar ou omitir commits da virada do dia. Documentar no prompt como limitação conhecida

### Riscos de segurança
- **NENHUM:** Apenas lê git log local, sem acesso a rede ou dados sensíveis

## Estimativa de Complexidade

| Área | Complexidade | Justificativa |
|------|-------------|---------------|
| Prompt engineering | M | Tradução de commits + agrupamento + formato exato exigem calibração |
| Coleta de input | P | 3 perguntas sequenciais simples |
| Testes | P | 2-3 casos manuais cobrem os critérios de aceite |
