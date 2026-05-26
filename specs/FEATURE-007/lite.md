# FEATURE-007 — Badge de status no README

**Modo:** lite
**Data:** 2026-05-26

## Outcomes

Após a feature, ambos os READMEs (EN e PT-BR) exibem três badges shields.io no topo: licença MIT, último commit e GitHub stars. O critério de pronto é os três badges renderizarem corretamente ao abrir o repositório no GitHub, sem nenhuma dependência de runtime adicionada ao projeto.

## Tasks

- [ ] T1 — Adicionar os 3 badges em `README.md` | Critério: badges de licença MIT, último commit e stars visíveis na linha do topo, após o badge de release e antes do link de idioma
- [ ] T2 — Replicar os badges em `README.pt-BR.md` | Critério: badges idênticos ao `README.md` na mesma posição

## Risco

Shields.io pode ter indisponibilidade momentânea — impacto apenas visual, sem afetar funcionalidade do repositório.
