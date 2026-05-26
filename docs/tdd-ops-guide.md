# Guia TDD + OPS — Camadas 2 e 3 do Pipeline AI-First

Este documento descreve como as camadas de TDD e OPS se integram ao pipeline SDD existente, formando um ciclo completo de 13 passos do requisito ao deploy.

---

## As Três Camadas do Pipeline

```
Camada 1 — SDD (Spec-Driven Development)
  Define o que construir e por quê.
  Artefatos: spec.md, perguntas-respondidas.md, plan.md, tasks.md

Camada 2 — TDD (Test-Driven Development)
  Define como verificar e implementar.
  Artefatos: arquivos de teste, código de produção

Camada 3 — OPS (Quality Gate + Segurança)
  Garante que o entregue é seguro e está pronto para produção.
  Artefatos: relatório de segurança, pipeline de CI/CD
```

---

## Os 13 Passos do Pipeline Completo

```
 1. SPEC         /sdd.specify          → specs/<ID>/spec.md
 2. CLARIFY      /sdd.clarify          → specs/<ID>/perguntas-respondidas.md
 3. PLAN         /sdd.plan             → specs/<ID>/plan.md
 4. TASKS        /sdd.tasks            → specs/<ID>/tasks.md
 5. WRITE TESTS  tdd-test-writer       → testes baseados nos critérios de aceite
 6. IMPLEMENT    tdd-implementer       → código mínimo para testes passarem
 7. REFACTOR     refactor              → código limpo, testes ainda verdes
 8. ANALYZE      /sdd.analyze          → specs/<ID>/analysis-report.md
 9. SEC AUDIT    security-auditor      → relatório de segurança com BLOCKERs
10. REVIEW       /sdd.review       → comentário de revisão para o PR
11. QUALITY GATE quality-gate.yml      → lint → test → security-scan → build
12. CHANGELOG    /sdd.changelog    → CHANGELOG.md atualizado
13. STANDUP      /sdd.standup      → resumo para o time
```

Cada passo tem um responsável claro (agente ou comando) e um artefato de saída verificável. Nenhum passo pode ser pulado sem registrar o motivo.

---

## Sequência de Ativação dos Agentes TDD

### Passo 5 — Ativar tdd-test-writer

Execute com o prompt base do agente (`.claude/agents/tdd-test-writer.md`), passando o ID da feature:

```
Você é o TDDTestWriter. Leia specs/FEATURE-NNN/spec.md e escreva os testes...
```

Aguarde a entrega dos arquivos de teste antes de prosseguir.

Critério de avanço: todos os critérios de aceite da spec têm pelo menos um caso de teste correspondente.

### Passo 6 — Ativar tdd-implementer

Execute com o prompt base do agente (`.claude/agents/tdd-implementer.md`):

```
Você é o TDDImplementer. Os testes para FEATURE-NNN já estão escritos...
```

Critério de avanço: todos os testes em verde, cobertura mínima declarada pelo tdd-test-writer atingida.

### Passo 7 — Ativar refactor

Execute somente após confirmação de que todos os testes estão verdes:

```
Você é o RefactorAgent. Todos os testes da feature FEATURE-NNN estão verdes...
```

Critério de avanço: testes continuam verdes, código mais legível — confirmado pelo agente.

### Passo 9 — Ativar security-auditor

Execute após `/sdd.analyze` (passo 8), antes do `/sdd.review`:

```
Você é o SecurityAuditor. Audite o código e a spec da feature FEATURE-NNN...
```

Critério de avanço: nenhum finding BLOCKER em aberto. Findings BLOCKER bloqueiam o passo 10 (Artigo VI.3 da constituição).

---

## Como o Quality Gate se Conecta ao Pipeline

O `quality-gate.yml` é executado automaticamente a cada Pull Request (após configuração). Ele é a verificação automatizada dos passos 5, 6 e 9 em CI:

```
quality-gate.yml
  ├── lint          ← valida formatação (complementa passo 7)
  ├── test          ← roda a suíte TDD (confirma passo 6)
  ├── security-scan ← detecta CVEs e secrets (complementa passo 9)
  └── build         ← confirma que o artefato compila (pré-condição para deploy)
```

O quality gate não substitui os agentes — ele automatiza a verificação contínua. Os agentes raciocinam sobre trade-offs; o quality gate executa verificações objetivas.

Para ativar o quality gate automático em PRs, edite `.github/workflows/quality-gate.yml` e troque o bloco `on` para:

```yaml
on:
  pull_request:
    branches: [main]
```

---

## Regras de Avanço Entre Camadas

### SDD → TDD (passos 4 → 5)

- `specs/<ID>/spec.md` aprovado (sem perguntas em aberto)
- `specs/<ID>/plan.md` revisado e sem contradições com a spec
- `specs/<ID>/tasks.md` com tasks numeradas e critérios de conclusão

### TDD → OPS (passos 7 → 8)

- Todos os testes verdes
- Cobertura mínima atingida
- Nenhum TODO ou placeholder de implementação no código de produção

### OPS → Merge (passos 11 → 12)

- Quality gate em verde (lint, test, security-scan, build)
- Nenhum finding BLOCKER do security-auditor em aberto
- Review aprovado pelo `/sdd.review` (veredicto APROVADO ou CONDICIONALMENTE APROVADO com condições endereçadas)

---

## Isolamento de Responsabilidades

Cada agente da camada TDD tem escopo estrito para evitar interferência:

| Agente | Escreve testes | Escreve produção | Refatora | Audita segurança |
|--------|---------------|-----------------|----------|-----------------|
| tdd-test-writer | sim | não | não | não |
| tdd-implementer | não | sim | não | não |
| refactor | não | sim (estrutura) | sim | não |
| security-auditor | não | não | não | sim |

Cruzamento de responsabilidades é um anti-padrão. Se um agente precisar sair do seu escopo, é sinal de que a task foi mal decomposta.

---

## Referência Rápida de Arquivos

```
.claude/agents/
  tdd-test-writer.md    ← passo 5: escreve testes
  tdd-implementer.md    ← passo 6: implementa código
  refactor.md           ← passo 7: limpa código
  security-auditor.md   ← passo 9: audita segurança

.claude/hooks/
  pre-tool-use.md       ← protege arquivos e detecta padrões proibidos

.claude/commands/
  sdd.specify.md        ← passo 1
  sdd.clarify.md        ← passo 2
  sdd.plan.md           ← passo 3
  sdd.tasks.md          ← passo 4
  sdd.analyze.md        ← passo 8
  sdd.lite.md           ← alternativa enxuta para features pequenas
  sdd.drift.md          ← detecta divergência spec/implementação
  sdd.review.md         ← passo 10
  sdd.changelog.md      ← passo 12
  sdd.standup.md        ← passo 13

.github/workflows/
  quality-gate.yml      ← passo 11 (configure antes de ativar)
```
