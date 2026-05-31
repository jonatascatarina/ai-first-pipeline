# CLAUDE.md — Instruções para Agentes Claude

Este arquivo é carregado em toda sessão — mantido conciso intencionalmente.
Para detalhes de comportamento por fase, leia `constitution.md` sob demanda, não na inicialização.

---

## Auto-start

Ao iniciar qualquer sessão, verifique:

1. Se `constitution.md` está vazio ou não foi preenchido:
   → Pergunte ao usuário: "Olá! O que você quer construir?"
   → Com a resposta, execute o pipeline completo em silêncio:
      constitution → specify → clarify → plan → tasks
   → Não mencione nomes técnicos (constitution, spec, EARS)
   → Faça perguntas naturais quando precisar de mais contexto:
      "Para quem é esse produto?"
      "Qual o maior problema que ele resolve?"
      "Tem alguma restrição técnica que eu deva saber?"
   → Ao terminar, mostre apenas:
      "Pronto! Aqui estão as primeiras tarefas para começar:"
      [lista as tasks geradas]

2. Se `constitution.md` já foi preenchido e existem specs em `specs/`:
   → Pergunte: "Bem-vindo de volta! Quer continuar uma feature existente ou começar algo novo?"
   → Mostre as features em andamento com status resumido

3. Se existem tasks pendentes em `specs/*/tasks.md`:
   → Pergunte: "Tem tasks pendentes em [FEATURE]. Quer continuar?"

---

## Model Routing

| Modelo | Tarefas | Fases do Pipeline |
|--------|---------|-------------------|
| `claude-haiku-4-5` | Tasks, formatação, scaffold, refatoração simples | T1, T6, T7, T8 |
| `claude-sonnet-4-6` | Spec, clarificação, análise, plano, revisão, TDD, auditoria | T2, T3, T4, T5, T9, T10 |

**Regra de escalonamento:** Se uma tarefa rotulada para Haiku exigir raciocínio sobre trade-offs ou ambiguidade, escalone para Sonnet e registre o motivo no output.

**Model routing por comando:** declarado no cabeçalho de cada arquivo em `.claude/commands/`.

---

## Referências — Leia Sob Demanda

| Arquivo | Quando ler |
|---------|-----------|
| `constitution.md` | Antes de qualquer task de spec, plano ou decisão arquitetural |
| `AGENTS.md` | Antes de ativar ou colaborar com um agente especializado |
| `docs/tdd-ops-guide.md` | Antes de executar passos 5–13 do pipeline |

---

## Regras de Formatação

- Headings com `#` a `###` apenas
- Listas com `-` (não `*` ou `+`)
- Blocos de código sempre com linguagem especificada
- Sem emojis em documentos de spec ou plano
- Sem frontmatter YAML em nenhum arquivo

---

## O Que Não Fazer

- Não invente requisitos que não estão na spec
- Não tome decisões de arquitetura sem registrar ADR
- Não modifique `constitution.md` sem instrução explícita do usuário
- Não crie arquivos fora da estrutura definida em `README.md`
- Não avance para a fase seguinte se a fase atual tiver itens em aberto

---

## Comandos Disponíveis

| Comando | Modelo | Descrição |
|---------|--------|-----------|
| `/sdd.start` | sonnet | Ponto de entrada único — detecta estado e conduz conversa natural |
| `/sdd.constitution` | sonnet | Define ou atualiza a constituição do projeto |
| `/sdd.specify` | sonnet | Cria spec detalhada de uma feature |
| `/sdd.clarify` | sonnet | Gera perguntas de clarificação para uma spec |
| `/sdd.plan` | sonnet | Cria plano de implementação a partir da spec |
| `/sdd.tasks` | haiku | Decompõe o plano em tasks executáveis |
| `/sdd.analyze` | sonnet | Analisa código ou spec e gera relatório |
| `/sdd.lite` | haiku | Pipeline enxuto para features pequenas |
| `/sdd.drift` | sonnet | Detecta divergência spec/implementação (requer `feature=` ou `layer=`) |
| `/sdd.review` | sonnet | Conduz revisão de código guiada por spec |
| `/sdd.issue` | haiku | Publica spec como Issue ou converte Issue em scaffold |
| `/sdd.adr` | sonnet | Guia a criação interativa de um ADR |
| `/sdd.epic` | sonnet | Decompõe iniciativa grande em features com mapa de dependências |
| `/sdd.wrap` | haiku | Gera changelog e resumo de standup ao final do ciclo |
| `/pr-checklist` | sonnet | Gera checklist de revisão de PR |
