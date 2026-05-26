# CLAUDE.md — Instruções para Agentes Claude

Este arquivo configura o comportamento dos agentes Claude neste repositório. Leia integralmente antes de executar qualquer tarefa.

---

## Model Routing

| Modelo | Tarefas | Fases do Pipeline |
|--------|---------|-------------------|
| `claude-haiku-4-5` | Geração de tasks, formatação, checklists, scaffold de arquivos, refatoração simples | T1 (init), T6 (tasks), T7 (refactor), T8 (lint/format) |
| `claude-sonnet-4-6` | Especificação, clarificação, análise, planejamento, revisão, TDD, auditoria de segurança | T2 (specify), T3 (clarify), T4 (plan), T5 (write tests + implement), T9 (sec audit), T10 (review) |

**Regra de escalonamento:** Se uma tarefa rotulada para Haiku exigir raciocínio sobre trade-offs ou ambiguidade de requisitos, escalone para Sonnet e registre o motivo no output.

---

## Agentes Especializados

| Agente | Arquivo | Fase |
|--------|---------|------|
| TDDTestWriter | `.claude/agents/tdd-test-writer.md` | Passo 5 — escrever testes antes da implementação |
| TDDImplementer | `.claude/agents/tdd-implementer.md` | Passo 6 — implementar código para testes passarem |
| RefactorAgent | `.claude/agents/refactor.md` | Passo 7 — limpar código com testes verdes |
| SecurityAuditor | `.claude/agents/security-auditor.md` | Passo 9 — auditar segurança antes do merge |
| OnboardingAgent | `.claude/agents/onboarding.md` | Sob demanda — resumo de contexto para agente novo |

Cada agente tem escopo estrito definido em seu arquivo. Cruzamento de responsabilidades é anti-padrão.

---

## Contexto do Repositório

- Este repositório é um **template de processo**, não um produto de software
- Todos os arquivos são Markdown puro — sem frontmatter YAML, sem código executável
- A estrutura de `specs/` é a fonte de verdade do que deve ser construído
- `constitution.md` define as regras que você deve seguir sem exceção

---

## Comportamento Esperado

### Ao receber uma task de spec

1. Leia `constitution.md` completo
2. Leia a spec existente em `specs/<ID>/spec.md` se houver
3. Identifique ambiguidades antes de produzir output
4. Use o comando `/sdd.clarify` para formalizar perguntas

### Ao receber uma task de plano

1. Verifique que `perguntas-respondidas.md` está completo (sem `TBD` em aberto)
2. Identifique riscos de segurança e privacidade explicitamente
3. Referencie ADRs existentes em `docs/adr/` quando relevante
4. Proponha novo ADR se a decisão tiver impacto arquitetural

### Ao receber uma task de análise

1. Analise conformidade com a spec, não apenas com boas práticas genéricas
2. Liste findings com severidade: `BLOCKER`, `WARNING`, `SUGGESTION`
3. Findings `BLOCKER` impedem merge — liste ação corretiva obrigatória
4. Salve resultado em `specs/<ID>/analysis-report.md`

---

## Regras de Formatação

- Headings com `#` a `###` apenas (sem `####` ou mais profundo em specs)
- Listas com `-` (não `*` ou `+`)
- Blocos de código sempre com linguagem especificada
- Tabelas com header separado por `---`
- Sem emojis em documentos de spec ou plano
- Sem frontmatter YAML em nenhum arquivo

---

## O Que Não Fazer

- Não invente requisitos que não estão na spec
- Não tome decisões de arquitetura sem registrar ADR
- Não modifique `constitution.md` sem instrução explícita do usuário
- Não crie arquivos fora da estrutura definida em `README.md`
- Não use linguagem vaga ("talvez", "pode ser", "geralmente") em critérios de aceite
- Não avance para a fase seguinte se a fase atual tiver itens em aberto

---

## Comandos Disponíveis

| Comando | Descrição |
|---------|-----------|
| `/sdd.constitution` | Define ou atualiza a constituição do projeto |
| `/sdd.specify` | Cria spec detalhada de uma feature |
| `/sdd.clarify` | Gera perguntas de clarificação para uma spec |
| `/sdd.plan` | Cria plano de implementação a partir da spec |
| `/sdd.tasks` | Decompõe o plano em tasks executáveis |
| `/sdd.analyze` | Analisa código ou spec e gera relatório |
| `/sdd.lite` | Pipeline enxuto para features pequenas (specify + plan + tasks em uma execução) |
| `/sdd.drift` | Detecta divergência entre spec aprovada e implementação via commits |
| `/sdd.review` | Conduz revisão de código guiada por spec junto ao revisor humano |
| `/sdd.issue` | Publica spec como GitHub Issue ou converte Issue em scaffold de spec (bidirecional) |
| `/sdd.adr` | Guia a criação interativa de um Architecture Decision Record |
| `/sdd.epic` | Decompõe uma iniciativa grande em features com mapa de dependências e sequência de entrega |
| `/sdd.changelog` | Gera seção de changelog a partir do git log e specs referenciadas |
| `/sdd.standup` | Gera resumo de standup diário a partir dos commits git do dia anterior |
| `/pr-checklist` | Gera checklist de revisão de PR a partir de título e descrição |
