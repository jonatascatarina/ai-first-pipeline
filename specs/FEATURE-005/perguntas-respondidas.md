# Perguntas Respondidas — FEATURE-005

## P1 — Qual a diferença real entre `/speckit.review` e `/speckit.analyze`?

**Resposta:** O `/speckit.analyze` é autônomo — o agente analisa o código diretamente e produz um relatório sem envolver o revisor humano. O `/speckit.review` é colaborativo — o agente não acessa o código, mas guia o revisor humano através de cada critério da spec com perguntas específicas. Os dois são complementares: analyze é para CI/CD e revisão rápida pelo autor, review é para revisão humana estruturada antes do merge.

## P2 — O output deve ser salvo em arquivo ou apenas mostrado na tela?

**Resposta:** Apenas mostrado na tela, dentro de um bloco copiável. O revisor copia e cola no comentário do PR manualmente. Não há persistência em arquivo — diferente do `/speckit.analyze` que salva `analysis-report.md`. Razão: o comentário no PR é o artefato canônico, não um arquivo local.

## P3 — O agente deve perguntar sobre itens do plano (plan.md) ou apenas da spec?

**Resposta:** Apenas da spec, especificamente os critérios de aceite. O plano é um artefato interno de planejamento; o revisor verifica o resultado prometido (spec), não como foi planejado internamente. Ler o `plan.md` seria escopo adicional que aumenta o tempo de revisão sem proporcional ganho de qualidade.

## P4 — Como o agente deve lidar com specs que têm muitos critérios (ex: 10+)?

**Resposta:** Apresentar um critério por vez sem mudança de comportamento. Se o revisor quiser agrupar, ele pode pedir ao agente. O limite prático é o tempo do revisor — specs bem escritas têm entre 4 e 8 critérios de aceite. Critérios em excesso indicam que a spec deveria ser dividida.

## P5 — O veredicto CONDICIONALMENTE APROVADO deve listar ações corretivas obrigatórias?

**Resposta:** Sim. Para cada critério PARCIAL, o agente deve sugerir a ação corretiva mínima para que o critério seja considerado OK. Essas ações formam as "condições de aprovação" listadas no veredicto. O autor do PR deve endereçar todas as condições antes do merge.

## P6 — O comando deve confirmar os critérios extraídos com o revisor antes de iniciar?

**Resposta:** Sim, conforme o risco identificado de specs sem critérios bem estruturados. O agente apresenta a lista de critérios extraídos e pergunta se está correto antes de iniciar as perguntas de revisão. Isso evita revisar critérios errados e dá ao revisor a chance de corrigir se a spec foi atualizada recentemente.
