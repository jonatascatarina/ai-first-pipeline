# Drift Report — TDD + OPS Layer

**Data:** 2026-05-26
**Arquivos analisados:** 7 (4 agentes + 1 hook + 1 workflow + 1 guia)
**Arquivo extra fora da spec:** 1 (onboarding.md)
**Gerado por:** DriftAgent (claude-sonnet-4-6)

## Resumo

A camada TDD + OPS está integralmente alinhada com a spec original. Todos os 7 arquivos exigidos existem e atendem aos requisitos definidos. Os 4 agentes obrigatórios (tdd-test-writer, tdd-implementer, refactor, security-auditor) têm propósito, modelo, responsabilidades (máx 5), isolamento explícito e prompt base. O hook, o workflow e o guia cobrem todos os requisitos funcionais. Um arquivo adicional (`onboarding.md`) foi implementado fora da spec — benéfico, sem conflito com nenhum requisito.

**Implementados:** 7/7 arquivos exigidos
**Faltando:** 0
**Fora da spec:** 1 (onboarding.md) + enhancements menores em 3 arquivos

---

## Estrutura de Arquivos

| Arquivo | Spec | Status |
|---------|------|--------|
| `.claude/agents/tdd-test-writer.md` | ✅ exigido | ✅ Implementado |
| `.claude/agents/tdd-implementer.md` | ✅ exigido | ✅ Implementado |
| `.claude/agents/refactor.md` | ✅ exigido | ✅ Implementado |
| `.claude/agents/security-auditor.md` | ✅ exigido | ✅ Implementado |
| `.claude/agents/onboarding.md` | ⚠️ não previsto | ⚠️ Fora da spec |
| `.claude/hooks/pre-tool-use.md` | ✅ exigido | ✅ Implementado |
| `.github/workflows/quality-gate.yml` | ✅ exigido | ✅ Implementado |
| `docs/tdd-ops-guide.md` | ✅ exigido | ✅ Implementado |

---

## Requisitos por Agente

| Requisito da spec | tdd-test-writer | tdd-implementer | refactor | security-auditor |
|-------------------|----------------|----------------|----------|-----------------|
| Propósito (1 linha) | ✅ | ✅ | ✅ | ✅ |
| Modelo recomendado | ✅ sonnet | ✅ sonnet | ✅ haiku (escala para sonnet) | ✅ sonnet |
| Responsabilidades (máx 5) | ✅ 5 itens | ✅ 5 itens | ✅ 5 itens | ✅ 5 itens |
| O que NÃO faz (isolamento) | ✅ 5 itens | ✅ 5 itens | ✅ 5 itens | ✅ 5 itens |
| Prompt base para ativar | ✅ | ✅ | ✅ | ✅ |

---

## pre-tool-use.md

| Requisito da spec | Status | Observação |
|-------------------|--------|-----------|
| Quando bloquear | ✅ Implementado | 2 categorias BLOQUEAR: arquivos protegidos + credenciais/secrets |
| Padrão de decisão: PERMITIR / ALERTAR / BLOQUEAR | ✅ Implementado | Seção "Padrão de Decisão" com as 3 saídas explicadas |
| Exemplo de regra preenchida | ✅ Implementado | Script bash completo para `constitution.md` com `exit 1` e mensagem de erro |

Enhancements além da spec (benignos):
- Integração com `.claude/settings.json` documentada — como registrar o hook na configuração do Claude Code
- Seção "Adicionando Novas Regras" com template padronizado (Alvo / Saída / Motivo / Condição)
- 2 categorias ALERTAR (operações em specs aprovadas e operações fora do escopo da feature ativa) — spec pedia o padrão de decisão, não enumerava os casos de alerta

---

## quality-gate.yml

| Requisito da spec | Status | Observação |
|-------------------|--------|-----------|
| Etapas: lint → test → security-scan → build | ✅ Implementado | 4 jobs em ordem correta com `needs:` garantindo sequência |
| Comentários explicando cada etapa | ✅ Implementado | Blocos de comentário antes de cada job com propósito e referência à spec/constituição |
| Não executável (placeholder) | ✅ Implementado | `workflow_dispatch` (manual) + `echo` + `TODO:` em todos os steps |

Enhancements além da spec (benignos):
- Exemplos de comando real por linguagem (Node, Python, Go) nos comentários — facilita adoção
- Etapa `security-scan` decomposta em 2 sub-steps: scan de dependências + scan de secrets
- Step de cobertura mínima dentro da etapa de teste, referenciando `tdd-test-writer`

---

## tdd-ops-guide.md

| Requisito da spec | Status | Observação |
|-------------------|--------|-----------|
| Fluxo completo TDD com subagents | ✅ Implementado | Passos 5–9 com critérios de avanço por agente |
| Como o quality gate se conecta ao SDD | ✅ Implementado | Seção "Como o Quality Gate se Conecta ao Pipeline" com diagrama |
| Referência aos 13 passos | ✅ Implementado | Tabela completa dos 13 passos com responsável e artefato de saída |

Enhancements além da spec (benignos):
- "Regras de Avanço Entre Camadas" (SDD→TDD, TDD→OPS, OPS→Merge) — critérios de transição não previstos na spec original
- "Isolamento de Responsabilidades" — tabela cruzando agente × escopo (escreve testes / escreve produção / refatora / audita)

---

## Implementações Fora da Spec

| Arquivo | Commit de origem | Avaliação |
|---------|-----------------|-----------|
| `.claude/agents/onboarding.md` | Adicionado junto com a camada TDD+OPS | Não previsto na spec original. Resolve problema real (cold-start de agente novo). Sem conflito com nenhum requisito. Drift não documentado mas benéfico. |

---

## Recomendação

- **onboarding.md** → sem ação obrigatória. Se o projeto quiser formalizar este agente, criar `specs/FEATURE-OPS-ONBOARDING/spec.md` com os requisitos do agente de onboarding — transformando drift em spec retroativa. Decisão humana.
- **Enhancements menores** (exemplos de comando, seções extras no guia) → nenhuma ação necessária. Todos são aditivos e não contradizem a spec original.
- **CA implícito não testado** → nenhum critério de aceite formal foi definido para a camada TDD+OPS. Considerar adicionar um `test-results.md` neste diretório após execução manual dos agentes em uma feature real (FEATURE-007 pode ser o candidato).

---

## Próximo Passo

**ALINHADO** — todos os 7 arquivos exigidos implementados e conformes. O único item de acompanhamento é a formalização opcional do `onboarding.md` como spec retroativa.
