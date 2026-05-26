# Tasks — FEATURE-004

## Dependências entre Tasks

```
T1 → T2 → T3 → T4
               T4 → T5
```

---

## 🟢 T1 — Criar estrutura base do comando `/speckit.standup` [SETUP] [P]

**O que fazer:**
Criar `.claude/commands/speckit.standup.md` com estrutura base: cabeçalho, contexto, seção de input e placeholders para o processo. Seguir o padrão dos outros comandos `/speckit.*`.

**Critério de conclusão:**
- [ ] Arquivo criado com estrutura consistente com os outros comandos do projeto
- [ ] Seções: contexto, input esperado, processo, output, regras

**Testes esperados:**
- Manual: formato visualmente idêntico ao de `speckit.changelog.md`

**Não inclui:**
Lógica de leitura do git log ou tradução de commits.

---

## 🟡 T2 — Implementar leitura do git log e coleta de input [CORE] [M]

**O que fazer:**
Escrever a seção de "Processo" com:
1. Instrução para executar `git log --since="yesterday" --until="today" --oneline --no-merges`
2. Instrução de fallback: se sem commits, perguntar "O que você fez ontem?" antes de continuar
3. Instrução para fazer as 3 perguntas sequencialmente:
   - "Há algo além dos commits para incluir no Ontem? (opcional)"
   - "O que você vai fazer hoje? (obrigatório)"
   - "Há algum bloqueio? (opcional, padrão: Nenhum)"
4. Instrução para não avançar sem resposta para "Hoje"

**Critério de conclusão:**
- [ ] Comando git explícito e copiável no prompt
- [ ] Fallback para ausência de commits documentado
- [ ] 3 perguntas com indicação de obrigatório/opcional
- [ ] CA-2 coberto: sem commits → pergunta antes de gerar

**Testes esperados:**
- Manual: executar em repositório sem commits de ontem → agente pergunta antes de gerar

**Não inclui:**
Tradução e agrupamento de commits — isso é T3.

---

## 🟡 T3 — Implementar tradução, agrupamento e few-shot [CORE] [M]

**O que fazer:**
Adicionar ao prompt:
1. Tabela de exemplos de tradução (commits técnicos → linguagem de negócio)
2. Regra de agrupamento: commits do mesmo tema → 1 item
3. Regra de limite: máximo 5 itens por seção, priorizar `feat:` e `fix:`
4. Instrução de tom: conversacional, primeira pessoa, sem jargão técnico
5. Instrução de idioma: sempre português, independente do idioma dos commits

**Critério de conclusão:**
- [ ] Tabela com pelo menos 5 exemplos de tradução (bom vs ruim)
- [ ] Regra de agrupamento documentada com exemplo
- [ ] Limite de 5 itens por seção explícito
- [ ] CA-1 coberto: commits relacionados agrupados em 1 item
- [ ] CA-6 coberto: `chore:` traduzido, não omitido nem copiado literal

**Testes esperados:**
- Manual: executar com commits mistos e verificar que saída é em português e sem prefixos técnicos

**Não inclui:**
Formatação final do output — isso é T4.

---

## 🟢 T4 — Implementar formato de output e header de data [CORE] [P]

**O que fazer:**
Adicionar ao prompt:
1. Template exato do output (com emojis, asteriscos Slack, bullets •)
2. Instrução para gerar o header com dia da semana e data em `DD/MM/YYYY` em português
3. Instrução para entregar dentro de bloco copiável
4. Instrução para seção Bloqueios: usar "Nenhum" se não informado

**Critério de conclusão:**
- [ ] Template do output incluso no prompt como few-shot
- [ ] Header com dia da semana em português (ex: "Terça-feira, 26/05/2026")
- [ ] Output dentro de bloco copiável
- [ ] CA-3, CA-4, CA-5, CA-7 cobertos

**Testes esperados:**
- Manual: verificar que o header tem formato correto para a data de hoje
- Manual: não informar bloqueios → seção aparece com "Nenhum"

**Não inclui:**
Atualização de CLAUDE.md e CHANGELOG — isso é T5.

---

## 🟢 T5 — Atualizar CLAUDE.md e CHANGELOG [DOCS] [P]

**O que fazer:**
1. Adicionar `/speckit.standup` na tabela de comandos do `CLAUDE.md`
2. Registrar a feature no `CHANGELOG.md` na seção `### Adicionado` de `[Não lançado]`

**Critério de conclusão:**
- [ ] Linha adicionada em `CLAUDE.md`: `| /speckit.standup | Gera resumo de standup diário a partir dos commits git do dia anterior |`
- [ ] Entrada no CHANGELOG referenciando FEATURE-004

**Não inclui:**
Criar ADR — sem impacto arquitetural.

---

## Resumo

| Task | Área | Risco | Estimativa |
|------|------|-------|-----------|
| T1 | SETUP | 🟢 Baixo | P |
| T2 | CORE | 🟡 Médio | M |
| T3 | CORE | 🟡 Médio | M |
| T4 | CORE | 🟢 Baixo | P |
| T5 | DOCS | 🟢 Baixo | P |
