# Tasks — FEATURE-006

## T1 — Criar `.github/ISSUE_TEMPLATE/feature-spec.md`

**O que fazer:** Criar o arquivo de template de Issue com frontmatter YAML (`name`, `about`, `title: "[SPEC] "`, `labels: ["spec"]`) e as seções: Contexto, Problema, Solução Proposta, Critérios de Aceite e Não-Escopo. Cada seção deve ter instruções em comentário HTML explicando o que preencher.

**Critério de conclusão:** Arquivo existe em `.github/ISSUE_TEMPLATE/feature-spec.md`. Ao abrir um novo Issue no repositório via GitHub UI, o template aparece como opção.

**Testes esperados:** Abrir Issue no GitHub → template listado como opção → selecionar → formulário exibe seções corretas.

---

## T2 — Criar `.claude/commands/speckit.issue.md`

**O que fazer:** Criar o prompt completo do comando `/speckit.issue` com os dois modos definidos em `plan.md`:
- Modo `publish`: lê spec, extrai campos, apresenta preview, cria Issue via `gh issue create`
- Modo `scaffold`: lê Issue via `gh issue view`, extrai campos por heurística, propõe ID, cria `specs/<ID>/spec.md`, adiciona comentário no Issue via `gh issue comment`

Incluir tratamento de todos os casos de erro: spec não encontrada, Issue não encontrado, label inexistente, spec já existente no modo scaffold.

**Critério de conclusão:** Arquivo existe em `.claude/commands/speckit.issue.md` e cobre os dois modos completos com fluxos de erro.

**Testes esperados:** Execução manual dos 6 cenários de teste definidos em `plan.md`.

---

## T3 — Atualizar `CLAUDE.md` com o novo comando

**O que fazer:** Adicionar `/speckit.issue` na tabela de "Comandos Disponíveis" com descrição: "Publica spec como GitHub Issue ou converte Issue em scaffold de spec (bidirecional)".

**Critério de conclusão:** Linha presente na tabela com formatação correta.

**Testes esperados:** Leitura visual do arquivo.

---

## T4 — Atualizar `README.md`

**O que fazer:** Dois ajustes:
1. Adicionar `.github/ISSUE_TEMPLATE/` na árvore de estrutura do projeto
2. Mencionar o Issue Template na seção "Quickstart" como passo opcional para times que usam GitHub Issues

**Critério de conclusão:** Estrutura e quickstart refletem a existência do template.

**Testes esperados:** Leitura visual — tree e quickstart coerentes.

---

## T5 — Atualizar `CHANGELOG.md`

**O que fazer:** Adicionar entrada na seção `[Não lançado]` descrevendo FEATURE-006. Remover "Integração com GitHub Issues via template de spec" da lista de Planejado.

**Critério de conclusão:** Entrada presente em `[Não lançado]`, item removido de Planejado (lista de Planejado fica vazia ou é removida).

**Testes esperados:** Leitura visual — sem duplicatas, sem item planejado obsoleto.

---

## T6 — Executar e documentar testes manuais

**O que fazer:** Executar os 6 cenários de teste definidos em `plan.md` e documentar os resultados em `specs/FEATURE-006/test-results.md`.

**Critério de conclusão:** Todos os 6 cenários executados com resultado documentado. Qualquer desvio do comportamento esperado registrado como finding.

**Testes esperados:** Os próprios cenários descritos em `plan.md`.
