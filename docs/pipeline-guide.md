# Como o Pipeline Funciona

Este guia é para quem quer entender o que acontece em segundo plano quando você diz ao agente o que quer construir.

---

## As 6 Fases

O pipeline executa em sequência, sempre na mesma ordem.

### 1. Constitution

**Comando:** `/sdd.constitution`  
**Arquivo gerado:** `constitution.md`

Define os princípios imutáveis do projeto: nome, stack, público-alvo, regras de governança e restrições técnicas. Executado uma vez por projeto. Todos os artefatos subsequentes herdam essas decisões.

### 2. Specify

**Comando:** `/sdd.specify`  
**Arquivo gerado:** `specs/<FEATURE>/spec.md`

Transforma a descrição do usuário em uma spec estruturada com três seções obrigatórias:

- **OUTCOMES** — o que muda no mundo quando a feature existir
- **SCOPE** — o que está dentro e fora do escopo
- **BEHAVIOR** — regras em notação EARS (Event-Action-Response-System)

### 3. Clarify

**Comando:** `/sdd.clarify`  
**Arquivo gerado:** `specs/<FEATURE>/perguntas-respondidas.md`

Lê a spec e levanta as perguntas que bloqueariam o planejamento. O usuário responde antes de qualquer código ser escrito — elimina suposições que geram retrabalho.

### 4. Plan

**Comando:** `/sdd.plan`  
**Arquivo gerado:** `specs/<FEATURE>/plan.md`

Com a spec clarificada, define estrutura de arquivos e módulos, abordagem de implementação, riscos e dependências.

### 5. Tasks

**Comando:** `/sdd.tasks`  
**Arquivo gerado:** `specs/<FEATURE>/tasks.md`

Decompõe o plano em tasks atômicas, cada uma com critério de conclusão claro e código de risco:

- 🟢 baixo — implementação direta, sem ambiguidade
- 🟡 médio — requer atenção a edge cases ou dependências
- 🔴 alto — risco de bloqueio, decisão arquitetural ou integração externa

### 6. Analyze

**Comando:** `/sdd.analyze`  
**Arquivo gerado:** `specs/<FEATURE>/analysis-report.md`

Após a implementação, compara o código com a spec original e gera um relatório de conformidade: o que foi implementado, o que divergiu e o que ficou fora do escopo.

---

## Detecção de Drift

**Comando:** `/sdd.drift feature=<FEATURE>` ou `/sdd.drift layer=<LAYER>`

Detecta divergências entre spec e implementação atual via `git log`. Produz um drift report com veredicto (ALINHADO / DIVERGENTE) e lista os itens que precisam de atenção.

Útil para projetos em andamento que acumularam mudanças sem atualização de spec.

---

## Subagentes

Os subagentes executam fases especializadas das Camadas 2 (TDD) e 3 (OPS). São ativados explicitamente pedindo ao agente que leia o arquivo de definição:

```
Leia .claude/agents/tdd-test-writer.md e specs/FEATURE-NNN/spec.md e execute.
```

| Agente | Fase | Responsabilidade |
|--------|------|-----------------|
| `tdd-test-writer` | 5 | Escreve testes baseados nos critérios de aceite da spec |
| `tdd-implementer` | 6 | Implementa o código mínimo para os testes passarem |
| `refactor` | 7 | Limpa o código sem quebrar os testes |
| `security-auditor` | 9 | Audita vulnerabilidades e gera relatório com BLOCKERs |
| `onboarding` | — | Contextualiza um agente novo sobre o estado do projeto |

---

## Model Routing

O pipeline distribui tarefas entre modelos por complexidade:

| Modelo | Usado para |
|--------|-----------|
| `claude-sonnet-4-6` | Spec, clarificação, plano, análise, revisão, TDD, auditoria |
| `claude-haiku-4-5` | Tasks, formatação, scaffold, refatoração simples |

Se uma tarefa rotulada para Haiku exigir raciocínio sobre trade-offs ou ambiguidade, o agente escala para Sonnet e registra o motivo no output.

O modelo recomendado está declarado no cabeçalho de cada arquivo em `.claude/commands/`.

---

Veja também: [`AGENTS.md`](../AGENTS.md) · [`docs/tdd-ops-guide.md`](tdd-ops-guide.md) · [`constitution.md`](../constitution.md)
