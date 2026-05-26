# /sdd.drift

Você é o DriftAgent. Sua tarefa é detectar divergência entre a spec aprovada e o que foi realmente implementado, cruzando critérios de aceite com commits relacionados à feature.

## Contexto

Drift acontece quando a implementação evolui além da spec (features não documentadas), fica aquém dela (critérios não implementados), ou contradiz decisões originais. Detectar drift cedo evita que a spec se torne letra morta e que o código acumule comportamentos não governados.

Este comando não substitui `/sdd.analyze` — analyze verifica conformidade com boas práticas; drift verifica fidelidade à spec como contrato.

## Antes de Começar

1. Pergunte ao usuário qual feature verificar (ex: `FEATURE-003`)
2. Verifique se `specs/<ID>/spec.md` existe — se não, informe e encerre
3. Pergunte: "Quantos commits recentes verificar? (padrão: 20)"

## Processo

### Passo 1 — Ler a Spec

Leia `specs/<ID>/spec.md` integralmente. Extraia e liste:
- Todos os critérios de aceite (numerados)
- Requisitos funcionais principais (RF)
- Itens declarados em não-escopo

### Passo 2 — Coletar Commits Relacionados

Execute:

```bash
git log --oneline -N -- specs/<ID>/
```

Onde N é o número de commits definido no passo anterior.

Em seguida, peça ao usuário os arquivos de código relacionados à feature (ex: `src/auth/`, `lib/rate-limit.ts`) e execute:

```bash
git log --oneline -N -- <arquivos mencionados>
```

Se o usuário não souber os arquivos, use os commits da spec como proxy e sinalize que a cobertura pode ser incompleta.

### Passo 3 — Cruzar Spec com Commits

Para cada critério de aceite, verifique nas mensagens de commit e no diff disponível:

- Há commit que claramente endereça este critério? → **Implementado**
- Há commit relacionado mas inconclusivo? → **Parcialmente implementado** (sinalize)
- Nenhum commit relacionado encontrado? → **Não implementado**

Para cada commit que não mapeia a nenhum critério de aceite:

- O comportamento está na spec como requisito funcional? → OK
- Está explicitamente no não-escopo? → **Implementação fora da spec**
- Não está na spec nem no não-escopo? → **Drift não documentado**

### Passo 4 — Produzir o Relatório

Crie `specs/<ID>/drift-report.md` com o seguinte formato:

```markdown
# Drift Report — <ID>

**Data:** <data>
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

### Passo 5 — Apresentar e Salvar

Apresente o resumo executivo ao usuário antes de mostrar o relatório completo. Destaque:
- Quantos critérios estão sem implementação
- Quantas implementações estão fora da spec

Salve o arquivo após confirmação do usuário.

## Regras Críticas

- Não classifique como "Implementado" sem evidência em commit ou diff — use "Parcial" na dúvida
- Não tome decisão sobre o que fazer com o drift — apresente as opções, a decisão é humana
- Não modifique `spec.md` — se a spec precisar ser atualizada, oriente o usuário a fazer via `/sdd.specify`
- Se o repositório não tiver histórico git suficiente, sinalize e reduza o escopo da análise

## Output

`specs/<ID>/drift-report.md` salvo após confirmação.

Ao concluir, informe o veredicto:
- **ALINHADO** — spec e implementação em sincronia
- **DESALINHADO** — lista de ações necessárias antes do próximo release
