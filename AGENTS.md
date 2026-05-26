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
**Comando:** `/speckit.specify`

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
**Comando:** `/speckit.clarify`

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
**Comando:** `/speckit.plan`

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
**Comando:** `/speckit.tasks`

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
**Comando:** `/speckit.analyze`

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

## Protocolo de Handoff

Quando um agente conclui sua fase:

1. Salva o artefato no caminho correto em `specs/<ID>/`
2. Indica explicitamente qual é o próximo passo recomendado
3. Lista qualquer pré-condição que precisa ser satisfeita antes da próxima fase
4. Nunca continua automaticamente para a próxima fase sem confirmação do humano
