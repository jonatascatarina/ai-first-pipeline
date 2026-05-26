# Perguntas Respondidas — FEATURE-004

## Rodada 1 — 2026-05-26

Todas as decisões foram tomadas pelo SpecAgent com autonomia total concedida pelo usuário.

---

### D1 — Qual é o input principal?

**Decisão:** Commits git do dia anterior via `git log --since="yesterday"`, complementados por contexto adicional opcional digitado pelo usuário.

**Justificativa:** Git log é a fonte mais objetiva e já disponível em qualquer projeto. Contexto adicional (reuniões, revisões) captura o trabalho não-técnico que os commits não registram.

**Impacto na spec:** RF-1 (git log) + RF-2 (contexto adicional opcional).

---

### D2 — Qual é o formato do output?

**Decisão:** Template fixo de 3 seções: "Ontem / Hoje / Bloqueios", com emojis e formatação Markdown compatível com Slack/Discord. Entregue dentro de bloco copiável.

**Justificativa:** O formato "Ontem / Hoje / Bloqueios" é o padrão mais adotado em times ágeis. Formatação Slack-compatible (asteriscos para negrito, bullet •) elimina re-formatação manual.

**Impacto na spec:** RF-5 (template), CA-3 (três seções), CA-5 (bloco copiável).

---

### D3 — Como tratar commits técnicos?

**Decisão:** Traduzir para linguagem de negócio, nunca copiar literalmente. Agrupar commits relacionados ao mesmo tema em um único item. Máximo de 5 itens por seção.

**Justificativa:** `feat: implement sliding window lua script` não comunica nada para um gerente de produto. "Implementei o algoritmo de controle de tráfego da API" comunica. O standup é para o time, não para o git.

**Impacto na spec:** RF-6, RF-7, CA-1, CA-6, RNF de máximo 5 itens.

---

### D4 — O que fazer quando não há commits ontem?

**Decisão:** Perguntar ao usuário o que foi feito antes de gerar. Não gerar standup com seção "Ontem" vazia sem confirmação.

**Justificativa:** Ausência de commits pode significar: dia não trabalhado, trabalho em outra branch, pair programming, reuniões, ou trabalho offline. O agente não deve assumir — deve perguntar.

**Impacto na spec:** RF-8, CA-2.
