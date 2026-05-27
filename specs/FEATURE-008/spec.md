# FEATURE-008 — OnboardingAgent

## Contexto

Agentes que entram em um projeto no meio do desenvolvimento perdem tempo lendo todos os artefatos do pipeline para construir contexto. Quanto maior o projeto (mais features, ADRs e decisões acumuladas), maior o cold-start. O OnboardingAgent resolve esse problema gerando um resumo estruturado e pronto para ser colado como contexto inicial de qualquer novo agente.

Este agente foi implementado junto com a camada TDD + OPS (v3.0.0) e formalizado como spec retroativa após identificação como drift benéfico no relatório `specs/TDD-OPS-LAYER/drift-report.md`.

## Problema

Sem um agente de onboarding:
- Novos agentes precisam ler `constitution.md`, `CLAUDE.md`, `AGENTS.md`, `CHANGELOG.md` e todos os diretórios de `specs/` antes de executar qualquer tarefa
- O risco de agir com base em estado desatualizado aumenta com o tamanho do projeto
- Não há formato padronizado de handoff de contexto entre agentes

## Atores

- **Humano responsável** — ativa o agente ao iniciar uma sessão com agente novo
- **Agente receptor** — recebe o resumo como contexto inicial da sessão
- **OnboardingAgent** — lê os artefatos do pipeline e produz o resumo (claude-haiku-4-5)

## Solução Proposta

Um agente (`onboarding.md`) que lê os artefatos canônicos do pipeline — constituição, CLAUDE.md, AGENTS.md, CHANGELOG.md, specs/ e docs/adr/ — e produz um resumo estruturado com: propósito do projeto, versão atual, princípios operacionais, model routing, status de cada feature e decisões arquiteturais relevantes. Output apenas na tela, sem criação de arquivos.

## Requisitos Funcionais

- RF-1: Ler e sintetizar `constitution.md`, `CLAUDE.md`, `AGENTS.md` e `CHANGELOG.md`
- RF-2: Listar todas as features em `specs/` com status inferido (em spec / em desenvolvimento / concluída) e resumo de uma linha
- RF-3: Resumir ADRs em `docs/adr/` destacando decisões que afetam múltiplos componentes
- RF-4: Identificar specs com perguntas em aberto (TBD ou TODO em `perguntas-respondidas.md`) e sinalizar como bloqueadores
- RF-5: Produzir output em formato estruturado copiável como contexto de sessão

## Requisitos Não-Funcionais

- Não lê código de produção — apenas artefatos do pipeline
- Não cria nem modifica arquivos — output apenas na tela
- Escala de haiku para sonnet quando o projeto tiver mais de 10 features ativas ou ADRs interdependentes

## Critérios de Aceite

1. Dado um projeto com 3 features (1 em spec, 1 em desenvolvimento, 1 concluída), o output classifica corretamente o status de cada uma inferindo pelos arquivos presentes
2. Dado `perguntas-respondidas.md` com `TBD` em aberto, o output lista a feature como bloqueador
3. Dado `specs/EXAMPLE-*/`, o agente não classifica exemplos didáticos como features ativas do produto
4. O output contém as 7 seções obrigatórias: Contexto do Projeto, Versão Atual, Princípios Operacionais, Model Routing, Features, Decisões Arquiteturais (ADRs), Bloqueadores
5. Dado repositório sem ADRs, o agente registra "Nenhum ADR encontrado" sem erro
6. O output não contém informações inventadas — status ausente é registrado como ausente, não inferido

## Não-Escopo

- Não lê código de produção (src/, lib/, etc.)
- Não toma decisões nem sugere direção — apenas descreve o estado atual
- Não substitui a leitura dos documentos originais para tarefas críticas
- Não resume specs de exemplos (`EXAMPLE-*`) como features ativas
- Não persiste o resumo em arquivo — apenas exibe na tela

## Origem

**Status:** Implementado antes da spec — formalizado via `/sdd.drift`

O `onboarding.md` foi criado junto com a camada TDD + OPS (v3.0.0) sem spec prévia. Identificado como drift benéfico no relatório `specs/TDD-OPS-LAYER/drift-report.md` com a recomendação: *"criar spec retroativa transformando drift em spec retroativa — decisão humana"*. Esta spec foi gerada a partir da leitura do agente implementado, sem alterar seu comportamento.

## Dependências

- Nenhuma dependência técnica — agente é inteiramente um prompt Markdown em `.claude/agents/`
- Implementado em `.claude/agents/onboarding.md`

## Riscos Conhecidos

- Estado desatualizado: o resumo reflete o estado dos arquivos no momento da execução. Se arquivos forem modificados durante a sessão sem reexecutar o agente, o contexto fica stale.
- Inferência de status: o agente infere status (em spec / em desenvolvimento / concluída) pela presença de arquivos — pode divergir do estado real se arquivos estiverem incompletos ou ausentes por erro.
