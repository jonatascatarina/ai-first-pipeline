# Tasks — FEATURE-003

## Dependências entre Tasks

```
T1 → T2 → T3 → T4 → T5
```

Sequencial — cada task depende da anterior.

---

## 🟢 T1 — Criar estrutura base do comando `/speckit.changelog` [SETUP] [P]

**O que fazer:**
Criar `.claude/commands/speckit.changelog.md` com a estrutura base: cabeçalho, contexto, seção de input esperado (versão e data) e placeholders para as etapas do processo.

Seguir o padrão de formato dos outros comandos em `.claude/commands/speckit.*.md`.

**Critério de conclusão:**
- [ ] Arquivo `.claude/commands/speckit.changelog.md` criado
- [ ] Estrutura consistente com os outros comandos `/speckit.*`
- [ ] Seções: contexto, input esperado, processo, output, regras

**Testes esperados:**
- Manual: verificar que o formato é idêntico ao dos outros comandos do projeto

**Não inclui:**
A lógica de leitura do git log e processamento — apenas o boilerplate.

---

## 🟡 T2 — Implementar lógica de leitura do git log e extração de IDs [CORE] [M]

**O que fazer:**
Escrever a seção de "Processo" do prompt com:
1. Instrução para executar `git tag --sort=-version:refname | head -1` e usar o resultado como base do range
2. Instrução para executar `git log <tag>..HEAD --oneline --no-merges` (ou sem tag se não houver)
3. Instrução para extrair IDs no padrão `FEATURE-\d{3}` e `EXAMPLE-\d{3}` de cada linha do log
4. Instrução para ler `specs/<ID>/spec.md` e extrair título (linha `# `) e seção `## Solução Proposta`
5. Instrução de fallback: se spec não existir, usar a mensagem do commit

**Critério de conclusão:**
- [ ] Comandos git explicitamente listados no prompt (copiáveis pelo agente)
- [ ] Regex de extração de ID documentada no prompt
- [ ] Fallback para spec ausente especificado
- [ ] Instrução para ler apenas título e `## Solução Proposta` da spec (não o arquivo todo)

**Testes esperados:**
- Manual: executar com o repositório atual e verificar que FEATURE-002 é detectada nos commits

**Não inclui:**
Lógica de classificação e agrupamento — isso é T3.

---

## 🟡 T3 — Implementar classificação, agrupamento e formato de output [CORE] [M]

**O que fazer:**
Adicionar ao prompt:
1. Tabela de classificação: `feat:` → `### Adicionado`, `fix:` → `### Corrigido`, `chore:/docs:/test:` → `### Interno`
2. Regra de agrupamento: múltiplos commits da mesma spec → uma única entrada
3. Formato da entrada: `- **<Título da Spec>** (<ID>): <Solução Proposta resumida em 1 frase>`
4. Formato da seção: `## [X.Y.Z] — YYYY-MM-DD` acima das subseções
5. Instrução para omitir subseções vazias (ex: sem fixes → não incluir `### Corrigido`)

**Critério de conclusão:**
- [ ] Tabela de classificação presente no prompt
- [ ] Regra de agrupamento por spec ID documentada
- [ ] Exemplo de entrada bem formatada incluído no prompt (few-shot)
- [ ] Subseções vazias não aparecem no output

**Testes esperados:**
- Manual: executar com commits mistos (feat + fix + chore) e verificar classificação correta

**Não inclui:**
A etapa de confirmação e escrita em disco — isso é T4.

---

## 🔴 T4 — Implementar confirmação e escrita no CHANGELOG.md [CORE] [P]

**O que fazer:**
Adicionar ao prompt a etapa final:
1. Apresentar o rascunho completo dentro de bloco de código Markdown
2. Perguntar: "Confirma a inserção desta seção no CHANGELOG.md? (sim/não)"
3. Se sim: instruir o agente a inserir o bloco logo abaixo da linha `## [Não lançado]` no `CHANGELOG.md`
4. Se não: encerrar sem modificar nenhum arquivo
5. Confirmar com: "Seção [X.Y.Z] adicionada ao CHANGELOG.md com sucesso."

**Critério de conclusão:**
- [ ] Rascunho apresentado em bloco copiável antes da confirmação
- [ ] Confirmação explícita exigida antes de qualquer escrita
- [ ] CA-7: recusa não modifica o arquivo
- [ ] Mensagem de confirmação após escrita bem-sucedida

**Testes esperados:**
- Manual: responder "não" e verificar que CHANGELOG.md não foi alterado
- Manual: responder "sim" e verificar que a nova seção foi inserida no lugar correto

**Não inclui:**
Push para GitHub — fora do escopo desta feature.

---

## 🟢 T5 — Atualizar CLAUDE.md e CHANGELOG [DOCS] [P]

**O que fazer:**
1. Adicionar `/speckit.changelog` na tabela de comandos do `CLAUDE.md`
2. Registrar a feature no `CHANGELOG.md` na seção `### Adicionado` de `[Não lançado]`

**Critério de conclusão:**
- [ ] Linha adicionada em `CLAUDE.md`: `| /speckit.changelog | Gera seção de changelog a partir do git log e specs referenciadas |`
- [ ] Entrada no CHANGELOG referenciando FEATURE-003

**Testes esperados:**
- Manual: tabela de comandos no CLAUDE.md está consistente e em ordem

**Não inclui:**
Criar ADR — esta feature não tem impacto arquitetural além do próprio template.

---

## Resumo

| Task | Área | Risco | Estimativa |
|------|------|-------|-----------|
| T1 | SETUP | 🟢 Baixo | P |
| T2 | CORE | 🟡 Médio | M |
| T3 | CORE | 🟡 Médio | M |
| T4 | CORE | 🔴 Crítico — escrita em disco | P |
| T5 | DOCS | 🟢 Baixo | P |
