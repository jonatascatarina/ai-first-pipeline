# Perguntas Respondidas — FEATURE-002

## Rodada 1 — 2026-05-26

### P1 (Dev) — Os itens do checklist devem ser fixos por categoria ou gerados dinamicamente pelo contexto do PR?

**Contexto:** Um checklist fixo é mais previsível mas menos útil — itens sobre banco de dados aparecem mesmo em PRs de CSS. Um checklist dinâmico é mais relevante mas pode errar o contexto.

**Resposta:** Híbrido. Dois itens são fixos e obrigatórios em todo PR (CHANGELOG e Spec). O restante é gerado dinamicamente a partir de palavras-chave no título e na descrição. Se nenhuma keyword de segurança for detectada, a seção Segurança aparece com uma observação "nenhum risco identificado neste PR" em vez de itens genéricos.

**Impacto na spec:** EARS-5 define os dois itens obrigatórios fixos. EARS-2, EARS-3 e EARS-4 definem geração condicional por keyword. Adicionado comportamento para seção vazia explícita.

---

### P2 (QA) — Como garantir que o checklist gerado é específico e não genérico? Qual é o critério de qualidade de um item?

**Contexto:** "Verificar se os testes passam" é inútil — qualquer revisor já sabe disso. O valor está em itens como "Verificar se o novo endpoint de autenticação tem teste de token expirado".

**Resposta:** Um item é considerado específico se referencia pelo menos um elemento do contexto do PR (nome do endpoint, tipo de dado, tecnologia mencionada, caso de uso descrito). O prompt do comando deve incluir exemplos de itens ruins vs bons para calibrar o modelo. CA-6 torna este critério verificável.

**Impacto na spec:** CA-6 formalizado: "os itens gerados são específicos ao contexto do PR, não genéricos". Adicionado na seção de Behavior que o prompt deve incluir exemplos contrastantes (few-shot).

---

### P3 (Tech Lead) — O output do comando deve ser apenas o checklist ou pode incluir comentário introdutório do agente?

**Contexto:** Se o agente gera texto introdutório ("Aqui está seu checklist..."), o revisor precisa deletar antes de colar no PR. Se gera apenas o Markdown puro, é copiável diretamente mas parece abrupto em conversas interativas.

**Resposta:** O output deve começar com um bloco Markdown copiável delimitado por ` ```markdown ` e ` ``` `, precedido por uma linha introdutória curta fora do bloco. O revisor copia apenas o conteúdo dentro do bloco. Isso equilibra usabilidade conversacional com praticidade de cópia.

**Impacto na spec:** EARS-7 atualizado para especificar que o checklist é entregue dentro de bloco de código Markdown. Adicionado exemplo de output em `plan.md`.
