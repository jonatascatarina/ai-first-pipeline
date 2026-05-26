# Perguntas Respondidas — FEATURE-003

## Rodada 1 — 2026-05-26

Todas as decisões de design foram tomadas pelo SpecAgent com autonomia total concedida pelo usuário. As questões abaixo documentam as decisões e sua justificativa.

---

### D1 — O que é "automático"? Gerado a partir de commits, specs ou os dois?

**Decisão:** Os dois. Git log fornece o range e a classificação por prefixo. As specs fornecem descrições ricas que substituem as mensagens de commit no output final. Commits sem spec usam a mensagem como fallback.

**Justificativa:** Usar só commits produz changelogs técnicos ("feat: implement sliding window"). Usar só specs desconecta do que foi realmente entregue. Cruzar os dois é o melhor dos dois mundos.

**Impacto na spec:** RF-3 (extrair IDs de spec dos commits) + RF-4 (ler spec.md para descrição) trabalham juntos.

---

### D2 — Quem aciona e quando?

**Decisão:** Comando manual `/speckit.changelog` executado pelo responsável pelo release antes de criar uma nova versão. Não é hook automático nem CI.

**Justificativa:** Changelogs exigem julgamento humano sobre o que é visível ao usuário. Automatizar o disparo sem revisão humana produziria entradas incorretas. O modelo do template é sempre human-in-the-loop.

**Impacto na spec:** RF-7 (apresentar rascunho antes de escrever) e CA-7 (sem modificação sem confirmação) derivam desta decisão.

---

### D3 — O que fazer com `chore:`, `docs:` e `test:` commits?

**Decisão:** Agrupar sob `### Interno` em vez de omitir. O usuário pode revisar e deletar manualmente antes de confirmar.

**Justificativa:** Omitir silenciosamente pode esconder contexto útil (ex: `chore: upgrade Redis 6→7` é relevante para operações). Deixar visível com label "Interno" dá controle ao usuário sem impor a decisão.

**Impacto na spec:** RF-5 atualizado para "agrupar em Interno" em vez de "ignorar".

---

### D4 — Se o usuário recusar o rascunho, o que acontece?

**Decisão:** O comando encerra sem modificar `CHANGELOG.md`. Para ajustar, o usuário re-executa o comando com contexto adicional.

**Justificativa:** Edição inline de rascunho no terminal é frágil. É mais seguro encerrar, o usuário edita a spec ou os commits, e re-executa.

**Impacto na spec:** CA-7 especifica apenas "não modificado" — sem loop de edição.
