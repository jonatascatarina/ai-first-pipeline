# Constituição do Projeto

Este documento define os princípios inegociáveis, contratos entre agentes e regras de governança do pipeline AI-first. Todo agente que opera neste repositório está vinculado a estas regras.

---

## Artigo I — Spec-Driven Development

1. Nenhum código de produção é escrito sem uma spec aprovada em `specs/<ID>/spec.md`.
2. A spec é a fonte de verdade. Em caso de conflito entre spec e código, a spec prevalece.
3. Mudanças de escopo exigem atualização da spec antes da implementação.
4. O ID da spec segue o padrão `FEATURE-NNN` com três dígitos.

---

## Artigo II — Qualidade de Especificação

1. Toda spec deve conter: contexto, problema, solução proposta, critérios de aceite e não-escopo.
2. Perguntas abertas impedem o avanço para a fase de plano. Todas devem ser respondidas em `perguntas-respondidas.md`.
3. Critérios de aceite devem ser verificáveis (testáveis), não subjetivos.
4. Requisitos não-funcionais (performance, segurança, disponibilidade) são obrigatórios em specs de infraestrutura e APIs.

---

## Artigo III — Planejamento

1. O plano em `plan.md` deve cobrir: abordagem técnica, dependências, riscos e estimativa de complexidade.
2. Riscos devem ter nível (baixo/médio/alto) e mitigação proposta.
3. O plano não pode contradizer a spec. Se contradizer, volte para a fase de spec.
4. Architecture Decision Records (ADRs) são obrigatórios para decisões com impacto em mais de um componente.

---

## Artigo IV — Tasks

1. Cada task em `tasks.md` deve ser autônoma: um agente deve conseguir executá-la sem contexto externo.
2. Tasks têm granularidade máxima de 4 horas de trabalho humano equivalente.
3. Toda task de código inclui: o que fazer, critério de conclusão e testes esperados.
4. Tasks de segurança (`SEC-*`) têm prioridade sobre tasks de feature.

---

## Artigo V — Agentes e Modelos

1. O modelo correto para cada fase está definido na tabela de model routing em `CLAUDE.md`.
2. Nenhum agente toma decisões de arquitetura sem registrar em ADR.
3. Agentes não modificam specs sem aprovação explícita do humano responsável.
4. Respostas de agentes que contradizem esta constituição devem ser descartadas.

---

## Artigo VI — Segurança e Compliance

1. Nenhuma credencial, secret ou dado pessoal é armazenado em Markdown ou código.
2. Specs de features que envolvem dados de usuários devem conter seção de privacidade e LGPD.
3. Vulnerabilidades identificadas em `analysis-report.md` bloqueiam merge até resolução.
4. Dependências externas são avaliadas quanto a licença e CVEs antes de inclusão.

---

## Artigo VII — Versionamento e Histórico

1. Toda alteração em spec, plano ou task é versionada via git com mensagem descritiva.
2. O `CHANGELOG.md` registra mudanças visíveis ao usuário final, não mudanças internas de processo.
3. ADRs são imutáveis após aprovação. Supersessão cria novo ADR referenciando o anterior.
4. O histórico de decisões é patrimônio do projeto — nunca reescreva commits de spec aprovada.

---

## Artigo VIII — Contribuição

1. O processo de contribuição está em `CONTRIBUTING.md`.
2. Pull Requests sem spec referenciada são rejeitados automaticamente.
3. Revisões de código verificam conformidade com spec, não apenas funcionamento.
4. A constituição pode ser alterada por consenso, registrado em ADR.
