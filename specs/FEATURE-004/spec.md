# FEATURE-004 — Daily Standup Summarizer

## Contexto

Desenvolvedores perdem 5-10 minutos toda manhã transformando commits técnicos em resumo legível para o standup do time. O git log já contém tudo o que foi feito — falta apenas um agente que o traduza para linguagem humana no formato esperado pelo time.

## Problema

Escrever o standup manualmente tem três custos:
- **Tempo:** 5-10 minutos por dia = ~40 horas por ano por desenvolvedor
- **Qualidade variável:** commits técnicos são difíceis de resumir sem contexto de negócio
- **Esquecimento:** sem ferramenta, o standup é frequentemente genérico ("continuei no projeto X") ou esquecido

## Solução Proposta

Um comando `/speckit.standup` que lê os commits git do dia anterior, permite ao usuário adicionar contexto não-técnico opcional (reuniões, revisões, bloqueios), e gera uma mensagem de standup copiável para Slack ou Discord no formato "Ontem / Hoje / Bloqueios".

## Atores

- **Desenvolvedor** — executa o comando toda manhã antes do standup
- **Agente** — lê git log, processa contexto adicional, gera o resumo (claude-sonnet-4-6)
- **Canal do time** — destino final da mensagem gerada (Slack, Discord, etc.)

## Requisitos Funcionais

1. O comando deve ler commits git de ontem via `git log --since="yesterday" --until="today" --oneline --no-merges`
2. O agente deve perguntar ao usuário se há contexto adicional a incluir (reuniões, revisões, conversas importantes) — resposta opcional
3. O agente deve perguntar quais são os planos para hoje — resposta obrigatória
4. O agente deve perguntar se há bloqueios — resposta opcional (padrão: "Nenhum")
5. O output deve seguir o template:
   ```
   🗓 Standup — <dia da semana>, <data>

   *Ontem:*
   • <item 1>
   • <item 2>

   *Hoje:*
   • <item 1>

   *Bloqueios:*
   • <item ou "Nenhum">
   ```
6. Commits técnicos devem ser traduzidos para linguagem de negócio (ex: `feat: implement sliding window lua script` → "Implementei o algoritmo de rate limiting com Redis")
7. Múltiplos commits relacionados ao mesmo tema devem ser agrupados em um item
8. Se não houver commits ontem (dia não trabalhado, fim de semana), o agente deve perguntar o que foi feito antes de gerar
9. O output deve ser entregue dentro de um bloco copiável

## Requisitos Não-Funcionais

- Output sempre em português, independente do idioma dos commits
- Tom conversacional e direto — sem jargão técnico no output final
- Máximo de 5 itens por seção — preferir síntese a lista longa
- O comando não modifica nenhum arquivo — apenas gera output na tela

## Critérios de Aceite

1. Dado commits `feat: add JWT auth` e `fix: token expiry returning 500`, o output agrupa em um item "Implementei autenticação JWT e corrigi bug de expiração de token"
2. Dado ausência de commits ontem, o agente pergunta o que foi feito antes de gerar o standup
3. O output contém exatamente as três seções: Ontem, Hoje, Bloqueios
4. Dado usuário não informar bloqueios, a seção aparece com "Nenhum"
5. O output está dentro de um bloco copiável (não solto no texto)
6. Commits de `chore:`, `docs:` e `test:` são traduzidos para linguagem legível (não omitidos, não copiados literalmente)
7. O formato da data no header é `DD/MM/YYYY` em português (ex: "Terça-feira, 26/05/2026")

## Não-Escopo

- Integração com Slack API ou Discord (envio automático)
- Leitura de calendário ou agenda do dia
- Persistência de standups anteriores
- Suporte a múltiplos repositórios git simultaneamente
- Notificação ou agendamento automático
- Formato em inglês ou outros idiomas

## Dependências

- Git disponível e repositório inicializado com pelo menos um commit
- Nenhuma outra dependência — feature é inteiramente um prompt em `.claude/commands/`

## Riscos Conhecidos

- **BAIXO — Commits vagos:** Mensagens como `fix: bug` não têm contexto suficiente para tradução útil. O agente usa o que tem — qualidade do output depende da qualidade dos commits
- **BAIXO — Fuso horário:** `--since="yesterday"` usa o fuso do sistema. Em times distribuídos, pode capturar commits do dia errado. Fora do escopo desta versão

## Perguntas em Aberto

Nenhuma — todas as decisões foram tomadas pelo SpecAgent com autonomia concedida pelo usuário.
