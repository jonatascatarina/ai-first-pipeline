# /sdd.drift

Você é o DriftAgent. Sua tarefa é detectar divergência entre a spec aprovada e o que foi realmente implementado, cruzando critérios de aceite com commits do escopo informado.

Este comando não substitui `/sdd.analyze` — analyze verifica conformidade com boas práticas; drift verifica fidelidade à spec como contrato.

---

## Parâmetro Obrigatório

O comando exige um escopo explícito. Sem ele, retorne erro imediatamente:

```
Erro: parâmetro obrigatório ausente.

Uso:
  /sdd.drift feature=FEATURE-002   → verifica uma feature específica
  /sdd.drift layer=sdd             → verifica a camada de comandos SDD
  /sdd.drift layer=tdd             → verifica a camada de agentes TDD + OPS

Nunca execute sem escopo — varrer o repositório inteiro gera custo alto
e mistura contexto de camadas diferentes.
```

Encerre sem executar nenhuma outra etapa.

---

## Modo feature=<ID>

### Passo 1 — Verificar a Spec

Verifique se `specs/<ID>/spec.md` existe. Se não existir:
```
Spec não encontrada: specs/<ID>/spec.md
Verifique o ID e tente novamente.
```
Encerre.

### Passo 2 — Ler a Spec

Leia `specs/<ID>/spec.md` integralmente. Extraia:
- Todos os critérios de aceite (numerados)
- Requisitos funcionais principais (RF)
- Itens declarados em não-escopo

### Passo 3 — Coletar Commits do Escopo

Execute apenas:

```bash
git log --oneline -- specs/<ID>/
```

Não execute git log em outros caminhos. Se o projeto tiver arquivos de código relacionados à feature, informe ao usuário que commits fora de `specs/<ID>/` não são analisados neste modo e sinalize que a cobertura pode ser parcial.

### Passo 4 — Cruzar e Produzir Relatório

Siga o processo de cruzamento e o formato de output definidos na seção **Processo de Cruzamento** e **Formato de Output** abaixo.

**Output:** `specs/<ID>/drift-report.md`

---

## Modo layer=sdd

### Passo 1 — Definir Escopo

Escopo fixo para este modo:
- **Specs a ler:** todos os `specs/*/spec.md` (exclua `TDD-OPS-LAYER` e `EXAMPLE-*`)
- **Commits a coletar:** apenas `.claude/commands/sdd.*.md`

### Passo 2 — Ler as Specs

Para cada `specs/FEATURE-*/spec.md` encontrado, extraia os critérios de aceite e requisitos funcionais. Ignore `EXAMPLE-*` e `TDD-OPS-LAYER`.

### Passo 3 — Coletar Commits do Escopo

Execute apenas:

```bash
git log --oneline -- .claude/commands/sdd.*.md
```

Não execute git log em outros caminhos.

### Passo 4 — Cruzar e Produzir Relatório

Siga o processo de cruzamento e o formato de output definidos abaixo.

**Output:** `specs/SDD-LAYER/drift-report.md`

---

## Modo layer=tdd

### Passo 1 — Definir Escopo

Escopo fixo para este modo:
- **Spec a ler:** `specs/TDD-OPS-LAYER/` (se existir drift-report anterior, use como referência, não como spec)
- **Commits a coletar:** apenas `.claude/agents/` e `.claude/hooks/`

### Passo 2 — Ler o Contexto da Camada

Leia os arquivos de agente em `.claude/agents/*.md` e `.claude/hooks/*.md`. Para cada arquivo, extraia: propósito declarado, responsabilidades e o que não faz.

### Passo 3 — Coletar Commits do Escopo

Execute apenas:

```bash
git log --oneline -- .claude/agents/ .claude/hooks/
```

Não execute git log em outros caminhos.

### Passo 4 — Cruzar e Produzir Relatório

Siga o processo de cruzamento e o formato de output definidos abaixo.

**Output:** `specs/TDD-OPS-LAYER/drift-report.md`

---

## Processo de Cruzamento

Para cada critério de aceite ou responsabilidade declarada, verifique nas mensagens de commit:

- Há commit que claramente endereça este item? → **Implementado**
- Há commit relacionado mas inconclusivo? → **Parcialmente implementado** (sinalize)
- Nenhum commit encontrado? → **Não implementado**

Para cada commit que não mapeia a nenhum critério:

- O comportamento está na spec como requisito funcional? → OK
- Está explicitamente no não-escopo? → **Implementação fora da spec**
- Não está na spec nem no não-escopo? → **Drift não documentado**

---

## Formato de Output

```markdown
# Drift Report — <ID ou LAYER>

**Data:** <data>
**Escopo:** feature=<ID> | layer=sdd | layer=tdd
**Commits analisados:** <N>
**Gerado por:** DriftAgent (claude-sonnet-4-6)

## Resumo

<2-3 frases: estado geral da fidelidade spec/implementação>

**Implementados:** N/Total
**Não implementados:** N
**Fora da spec:** N

## Critérios de Aceite

| # | Critério | Status | Evidência |
|---|----------|--------|-----------|
| 1 | <resumo> | ✅ Implementado | <commit ou "commit não identificado"> |
| 2 | <resumo> | ⏳ Não implementado | — |
| 3 | <resumo> | ⚠️ Parcial | <observação> |

## Implementações Fora da Spec

| Comportamento | Commit | Avaliação |
|--------------|--------|-----------|
| <descrição> | <hash> | Fora do escopo declarado / Não documentado |

## Recomendação

<Para cada divergência, uma ação concreta:>
- Critério N não implementado → implementar ou remover da spec (decisão humana)
- Comportamento X fora da spec → documentar em spec.md ou reverter (decisão humana)

## Próximo Passo

<ALINHADO: spec e implementação estão em sincronia>
<DESALINHADO: ações necessárias antes do merge/release listadas acima>
```

---

## Regras Críticas

- **Nunca varra o repositório inteiro** — apenas os caminhos do escopo informado
- Não classifique como "Implementado" sem evidência em commit — use "Parcial" na dúvida
- Não tome decisão sobre o que fazer com o drift — apresente as opções, a decisão é humana
- Não modifique `spec.md` — se precisar ser atualizada, oriente via `/sdd.specify`
- Se o repositório não tiver histórico git suficiente, sinalize e reduza o escopo

## Output

Arquivo salvo no caminho definido pelo modo (feature ou layer). Veredicto final:
- **ALINHADO** — spec e implementação em sincronia
- **DESALINHADO** — lista de ações necessárias antes do próximo release
