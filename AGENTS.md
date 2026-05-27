# AGENTS.md — Definição dos Agentes do Pipeline

Este documento define os agentes que operam no pipeline AI-first, seus papéis, responsabilidades e protocolos de interação.

---

## Visão Geral

O pipeline usa agentes especializados, cada um responsável por uma fase. Agentes não acumulam contexto entre fases — cada fase recebe os artefatos da fase anterior como input explícito.

```
[Humano] → SpecAgent → ClarifyAgent → PlanAgent → TaskAgent → [Humano] → AnalyzeAgent
```

---

## SpecAgent

**Modelo:** `claude-sonnet-4-6`
**Fase:** T2 — Especificação
**Comando:** `/sdd.specify`

### Responsabilidades
- Produzir `specs/<ID>/spec.md` completo a partir de uma descrição informal
- Identificar ambiguidades e lacunas no requisito original
- Definir critérios de aceite verificáveis
- Delimitar explicitamente o não-escopo

### Input esperado
- Descrição informal da feature (texto livre do usuário)
- Contexto do projeto (`constitution.md`, ADRs relevantes)

### Output obrigatório
- `specs/<ID>/spec.md` com todas as seções preenchidas

### Restrições
- Não avança se critérios de aceite são subjetivos
- Não assume tecnologia específica sem requisito explícito

---

## ClarifyAgent

**Modelo:** `claude-sonnet-4-6`
**Fase:** T3 — Clarificação
**Comando:** `/sdd.clarify`

### Responsabilidades
- Ler a spec e identificar questões que bloqueiam o planejamento
- Formular perguntas precisas (não abertas demais)
- Registrar respostas em `perguntas-respondidas.md`
- Confirmar quando todas as questões estão resolvidas

### Input esperado
- `specs/<ID>/spec.md`

### Output obrigatório
- `specs/<ID>/perguntas-respondidas.md` com perguntas e respostas
- Confirmação de que a spec está pronta para planejamento

### Restrições
- Máximo de 10 perguntas por rodada
- Perguntas devem ser específicas e direcionadas (sem "o que você quer dizer com X?")

---

## PlanAgent

**Modelo:** `claude-sonnet-4-6`
**Fase:** T4 — Planejamento
**Comando:** `/sdd.plan`

### Responsabilidades
- Produzir `specs/<ID>/plan.md` com abordagem técnica detalhada
- Identificar dependências entre componentes
- Avaliar riscos e propor mitigações
- Indicar se ADR é necessário

### Input esperado
- `specs/<ID>/spec.md`
- `specs/<ID>/perguntas-respondidas.md`
- ADRs existentes em `docs/adr/`

### Output obrigatório
- `specs/<ID>/plan.md`
- Rascunho de ADR em `docs/adr/` se decisão arquitetural for tomada

### Restrições
- Não planeja o que não está na spec
- Riscos de segurança têm seção obrigatória separada

---

## TaskAgent

**Modelo:** `claude-haiku-4-5`
**Fase:** T6 — Decomposição de Tasks
**Comando:** `/sdd.tasks`

### Responsabilidades
- Decompor o plano em tasks atômicas e executáveis
- Garantir que cada task tem critério de conclusão objetivo
- Identificar dependências entre tasks
- Estimar complexidade (P: pequena, M: média, G: grande)

### Input esperado
- `specs/<ID>/plan.md`
- `specs/<ID>/spec.md` (para validar cobertura)

### Output obrigatório
- `specs/<ID>/tasks.md` com todas as tasks numeradas

### Restrições
- Tasks máximas de 4h de trabalho equivalente
- Toda task de código inclui critério de teste
- Tasks de segurança prefixadas com `SEC-`

---

## AnalyzeAgent

**Modelo:** `claude-sonnet-4-6`
**Fase:** T7/T8 — Análise
**Comando:** `/sdd.analyze`

### Responsabilidades
- Verificar conformidade da implementação com a spec
- Identificar riscos de segurança, performance e manutenibilidade
- Classificar findings por severidade
- Produzir relatório acionável

### Input esperado
- `specs/<ID>/spec.md`
- Código ou artefatos a serem analisados
- `specs/<ID>/plan.md` (para verificar desvios)

### Output obrigatório
- `specs/<ID>/analysis-report.md`

### Severidade de findings

| Nível | Significado | Ação |
|-------|-------------|------|
| `BLOCKER` | Impede merge/deploy | Corrigir antes de avançar |
| `WARNING` | Risco identificado | Corrigir ou aceitar com justificativa |
| `SUGGESTION` | Melhoria opcional | Registrar para backlog |

---

## OnboardingAgent

**Modelo:** `claude-haiku-4-5` (escalar para `claude-sonnet-4-6` se o projeto tiver mais de 10 features ativas ou ADRs interdependentes)
**Fase:** Sob demanda — início de sessão com agente novo
**Arquivo:** `.claude/agents/onboarding.md`
**Spec:** `specs/FEATURE-008/spec.md`

### Responsabilidades
- Sintetizar `constitution.md`, `CLAUDE.md`, `AGENTS.md` e `CHANGELOG.md`
- Listar todas as features em `specs/` com status inferido e resumo de uma linha
- Resumir ADRs em `docs/adr/` destacando decisões que afetam múltiplos componentes
- Identificar specs com perguntas em aberto e sinalizar como bloqueadores
- Produzir resumo estruturado pronto para ser colado como contexto de sessão

### Input esperado
- Repositório com `constitution.md`, `CLAUDE.md`, `CHANGELOG.md`, `specs/` e `docs/adr/`

### Output obrigatório
- Resumo na tela com 7 seções: Contexto do Projeto, Versão Atual, Princípios Operacionais, Model Routing, Features, ADRs, Bloqueadores

### Restrições
- Não lê código de produção — apenas artefatos do pipeline
- Não cria nem modifica arquivos
- Não classifica `EXAMPLE-*` como features ativas do produto
- Não inventa status — registra ausência como ausente

---

## Protocolo de Handoff

Quando um agente conclui sua fase:

1. Salva o artefato no caminho correto em `specs/<ID>/`
2. Indica explicitamente qual é o próximo passo recomendado
3. Lista qualquer pré-condição que precisa ser satisfeita antes da próxima fase
4. Nunca continua automaticamente para a próxima fase sem confirmação do humano
