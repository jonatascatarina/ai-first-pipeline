# Tasks — FEATURE-005

## T1 — Criar o arquivo do comando `/speckit.review`

**O que fazer:** Criar `.claude/commands/speckit.review.md` com o prompt completo do ReviewAgent conforme definido em `plan.md`.

**Criterio de conclusao:** O arquivo existe em `.claude/commands/speckit.review.md` e contém as 4 fases do fluxo (coleta de input, extração de critérios, revisão por critério, geração de output).

**Testes esperados:** Executar `/speckit.review` em um projeto com spec existente e verificar que o agente conduz o fluxo completo sem desvios.

---

## T2 — Atualizar `CLAUDE.md` com o novo comando

**O que fazer:** Adicionar `/speckit.review` na tabela de "Comandos Disponíveis" em `CLAUDE.md` com descrição: "Conduz revisão de código guiada por spec junto ao revisor humano".

**Criterio de conclusao:** A tabela de comandos em `CLAUDE.md` contém a linha do `/speckit.review`.

**Testes esperados:** Leitura visual do arquivo — linha presente e formatação correta.

---

## T3 — Atualizar `README.md` com o novo comando

**O que fazer:** Adicionar `/speckit.review` na listagem de comandos disponíveis em `README.md`.

**Criterio de conclusao:** O README menciona `/speckit.review` com descrição consistente com `CLAUDE.md`.

**Testes esperados:** Leitura visual do arquivo — entrada presente e alinhada com as demais.

---

## T4 — Executar testes manuais do comando

**O que fazer:** Executar os 5 cenários de teste definidos em `plan.md`:
1. Todos os critérios OK → veredicto APROVADO
2. Um critério AUSENTE → veredicto BLOQUEADO
3. Um critério PARCIAL → veredicto CONDICIONALMENTE APROVADO com condição
4. ID inexistente → mensagem de erro sem output
5. Confirmação de critérios extraídos funciona antes da revisão iniciar

**Criterio de conclusao:** Todos os 5 cenários produzem o resultado esperado. Resultados documentados em `specs/FEATURE-005/test-results.md`.

**Testes esperados:** Os próprios cenários descritos acima.

---

## T5 — Atualizar `CHANGELOG.md`

**O que fazer:** Adicionar entrada na seção `[Não lançado]` do `CHANGELOG.md` descrevendo a adição do comando `/speckit.review` (FEATURE-005).

**Criterio de conclusao:** `CHANGELOG.md` contém entrada na seção `[Não lançado]` referenciando FEATURE-005.

**Testes esperados:** Leitura visual — entrada presente, formatação correta, sem duplicatas.
