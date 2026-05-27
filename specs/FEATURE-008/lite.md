# FEATURE-008 — Formalização do OnboardingAgent

**Modo:** lite
**Data:** 2026-05-26

## Outcomes

Após a feature, o `onboarding.md` deixa de ser drift não documentado e passa a ser um agente de primeira classe do pipeline: spec retroativa criada em `specs/FEATURE-008/spec.md` e entrada adicionada em `AGENTS.md`. Critério de pronto: qualquer colaborador pode entender o propósito, escopo e restrições do OnboardingAgent lendo os artefatos do pipeline, sem precisar ler o arquivo do agente diretamente.

## Tasks

- [ ] T1 — Criar `specs/FEATURE-008/spec.md` com spec retroativa do OnboardingAgent | Critério: arquivo existe com propósito, atores, requisitos funcionais, critérios de aceite e não-escopo preenchidos a partir do comportamento já implementado
- [ ] T2 — Adicionar seção `OnboardingAgent` em `AGENTS.md` | Critério: entrada consistente com as demais seções do arquivo (modelo, fase, responsabilidades, input, output, restrições)

## Risco

Sem risco: a implementação já existe e está validada — a formalização é documental.
