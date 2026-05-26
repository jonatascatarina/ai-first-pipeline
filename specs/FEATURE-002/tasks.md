# Tasks — FEATURE-002

## Dependências entre Tasks

```
T1 → T2 → T3 → T4 → T5
```

Sequencial — cada task depende da anterior.

---

## 🟢 T1 — Criar estrutura do comando `/pr-checklist` [SETUP] [P]

**O que fazer:**
Criar o arquivo `.claude/commands/pr-checklist.md` com a estrutura base do comando: cabeçalho, contexto, instrução de coleta de input (título + descrição), e placeholder para a lógica de geração.

Seguir o mesmo padrão de formato dos outros comandos em `.claude/commands/speckit.*.md`.

**Critério de conclusão:**
- [ ] Arquivo `.claude/commands/pr-checklist.md` criado
- [ ] Estrutura consistente com os outros comandos do projeto
- [ ] Seções: contexto, input esperado, processo, output, regras

**Testes esperados:**
- Manual: abrir o arquivo e verificar que o formato é idêntico ao dos comandos `/speckit.*`

**Não inclui:**
O prompt de geração em si — apenas a estrutura e o boilerplate.

---

## 🟡 T2 — Escrever lógica de detecção de contexto e few-shot [CORE] [M]

**O que fazer:**
Escrever a seção de "Processo" do prompt com:
1. Instrução para detectar keywords por categoria (segurança, testes, arquitetura, docs)
2. Tabela de keywords → itens esperados
3. Exemplos few-shot: um item ruim (genérico) vs um item bom (específico) para cada categoria
4. Instrução para a seção vazia: se nenhuma keyword for detectada para uma categoria, escrever "Nenhum risco identificado nesta categoria para este PR."

**Critério de conclusão:**
- [ ] Tabela de keywords presente no prompt
- [ ] Pelo menos 2 exemplos few-shot (bom vs ruim) por categoria
- [ ] Instrução explícita para evitar itens genéricos como "verificar se os testes passam"
- [ ] Instrução para categoria vazia explicitamente tratada

**Testes esperados:**
- Manual: executar o comando com PR de autenticação e verificar que itens de segurança são específicos (mencionam JWT, tokens, etc.)

**Não inclui:**
Os dois itens obrigatórios fixos — isso é T3.

---

## 🟢 T3 — Adicionar itens obrigatórios e formato de output [CORE] [P]

**O que fazer:**
Adicionar ao prompt:
1. Instrução para sempre incluir os dois itens fixos (CHANGELOG e Spec) independente do contexto
2. Instrução de formato: output dentro de bloco ` ```markdown ``` `, precedido por uma linha introdutória
3. Instrução de validação: EARS-6 — se título < 5 caracteres, solicitar mais informações

**Critério de conclusão:**
- [ ] Itens fixos presentes em todo output (verificável manualmente)
- [ ] Output sempre dentro de bloco de código Markdown
- [ ] Comportamento de input insuficiente documentado no prompt

**Testes esperados:**
- Manual: executar com título "fix" (3 chars) — agente deve pedir mais informações
- Manual: verificar que CHANGELOG e Spec estão presentes em output de qualquer PR

**Não inclui:**
Testes automatizados — escopo de T5.

---

## 🟡 T4 — Validar com 3 exemplos representativos [TEST] [M]

**O que fazer:**
Executar o comando `/pr-checklist` com três PRs representativos e verificar conformidade com os critérios de aceite:

- **PR-A:** "Add JWT authentication to /api/users" (segurança dominante)
- **PR-B:** "Fix typo in README.md" (baixo risco, itens mínimos)
- **PR-C:** "Migrate users table to add plan column" (arquitetura + dados)

Para cada execução, verificar CA-1 a CA-6 da spec.

**Critério de conclusão:**
- [ ] CA-1: PR-A gera ≥ 3 itens de segurança específicos sobre JWT
- [ ] CA-2: Todos os PRs geram as 4 seções
- [ ] CA-3: Todos os PRs incluem os 2 itens fixos
- [ ] CA-4: Título "fix" solicita mais informações
- [ ] CA-5: Output é Markdown válido com `- [ ]` e `##`
- [ ] CA-6: Itens de PR-A mencionam "JWT" ou "token" explicitamente

**Testes esperados:**
- Manual: 3 execuções documentadas com resultado observado vs esperado

**Não inclui:**
Automação dos testes — apenas validação manual e registro dos resultados.

---

## 🔴 T5 — Atualizar CLAUDE.md e CHANGELOG [DOCS] [P]

**O que fazer:**
1. Adicionar `/pr-checklist` na tabela de comandos do `CLAUDE.md`
2. Registrar a feature no `CHANGELOG.md` na seção `[Não lançado]`

**Critério de conclusão:**
- [ ] Linha adicionada em CLAUDE.md: `| /pr-checklist | Gera checklist de revisão de PR a partir de título e descrição |`
- [ ] Entrada no CHANGELOG com descrição da feature

**Testes esperados:**
- Manual: verificar que a tabela de comandos no CLAUDE.md está consistente

**Não inclui:**
Criar o ADR — esta feature não tem impacto arquitetural que justifique ADR (decisão de escopo: prompt-only, sem código).

---

## Resumo

| Task | Área | Risco | Estimativa |
|------|------|-------|-----------|
| T1 | SETUP | 🟢 Baixo | P |
| T2 | CORE | 🟡 Médio | M |
| T3 | CORE | 🟢 Baixo | P |
| T4 | TEST | 🟡 Médio | M |
| T5 | DOCS | 🔴 Crítico se esquecido | P |
