# FEATURE-005 — Spec-Guided Code Review

## Contexto

Revisões de código são mais eficazes quando o revisor sabe exatamente o que a feature deveria fazer. Hoje, o projeto tem specs detalhadas em `specs/<ID>/spec.md` com critérios de aceite verificáveis — mas o revisor precisa ler a spec por conta própria, mapear cada critério para o código e formular perguntas manualmente. Esse processo é lento e inconsistente.

O `/speckit.analyze` já existe para análise autônoma pelo agente. Mas há casos em que o revisor humano precisa conduzir a revisão com suporte do agente — especialmente quando o código está no GitHub e o agente não tem acesso direto ao diff.

## Problema

Sem uma ferramenta de revisão guiada por spec:
- O revisor revisa com base em boas práticas genéricas, não nos critérios acordados com o autor
- Critérios de aceite da spec ficam de fora da revisão porque o revisor não os memorizou
- O feedback fica disperso em comentários de linha sem estrutura de conformidade com spec
- A conclusão da revisão ("aprovado" ou "bloqueado") não está ligada a evidências específicas da spec

## Solução Proposta

Um comando `/speckit.review` que, dado um ID de feature e o contexto de código fornecido pelo revisor, conduz uma revisão estruturada perguntando sobre cada critério de aceite da spec. O agente formula perguntas específicas ao revisor, coleta observações e produz um comentário de revisão formatado para colar no PR do GitHub.

## Atores

- **Revisor** — humano que conhece o código, responde às perguntas do agente sobre cada critério
- **Agente** — lê a spec, formula perguntas por critério, coleta respostas, produz o comentário de revisão (claude-sonnet-4-6)
- **Autor do PR** — recebe o comentário de revisão no GitHub

## Requisitos Funcionais

1. O agente deve perguntar qual feature revisar (ex: `FEATURE-001`) e confirmar que o revisor tem o código disponível para consulta
2. O agente deve ler `specs/<ID>/spec.md` e extrair os critérios de aceite
3. Para cada critério de aceite, o agente deve:
   a. Apresentar o critério em linguagem clara
   b. Formular uma ou duas perguntas específicas para o revisor verificar no código
   c. Coletar a resposta do revisor (OK / PARCIAL / AUSENTE + observação opcional)
4. Ao fim de todos os critérios, o agente deve perguntar se há observações gerais adicionais
5. O agente deve produzir um comentário de revisão em Markdown pronto para colar no PR, contendo:
   - Tabela de conformidade por critério (OK / PARCIAL / AUSENTE)
   - Seção de observações por critério que tiver PARCIAL ou AUSENTE
   - Veredicto final: APROVADO / BLOQUEADO / CONDICIONALMENTE APROVADO
   - Condições de aprovação se veredicto for condicional
6. O veredicto deve ser BLOQUEADO se qualquer critério estiver AUSENTE
7. O veredicto deve ser CONDICIONALMENTE APROVADO se qualquer critério estiver PARCIAL e nenhum AUSENTE
8. O veredicto deve ser APROVADO somente se todos os critérios estiverem OK

## Requisitos Não-Funcionais

- O agente apresenta um critério por vez — não despeja todos de uma vez
- O output final é um bloco Markdown copiável, válido como comentário de PR no GitHub
- O comando não lê arquivos de código diretamente — depende do revisor como fonte de verdade
- Tempo médio de revisão guiada: até 2 minutos por critério de aceite
- Output sempre em português, independente do idioma da spec

## Critérios de Aceite

1. Dado `FEATURE-002` como input, o agente apresenta cada um dos 6 critérios de aceite da spec um por vez com perguntas específicas derivadas do critério
2. Dado todos os critérios respondidos como OK, o veredicto no output é APROVADO
3. Dado ao menos um critério respondido como AUSENTE, o veredicto no output é BLOQUEADO
4. Dado ao menos um critério respondido como PARCIAL e nenhum AUSENTE, o veredicto é CONDICIONALMENTE APROVADO com as condições listadas
5. O output contém uma tabela Markdown com colunas: Critério, Status, Observação
6. O output é entregue dentro de um bloco copiável
7. Dado feature ID inexistente (sem `spec.md`), o agente informa o erro e encerra sem gerar output

## Não-Escopo

- Leitura direta de diffs ou arquivos de código pelo agente
- Integração com GitHub API para postar o comentário automaticamente
- Revisão de múltiplas features em uma única execução
- Histórico ou persistência de revisões anteriores
- Suporte a critérios de aceite fora do padrão do projeto (specs externas)
- Análise autônoma pelo agente (isso é responsabilidade do `/speckit.analyze`)

## Dependências

- Spec completa em `specs/<ID>/spec.md` com seção de critérios de aceite
- Revisor com acesso ao código ou diff do PR
- Nenhuma dependência técnica — feature é inteiramente um prompt em `.claude/commands/`

## Riscos Conhecidos

- **MÉDIO — Specs sem critérios de aceite estruturados:** Se a spec não tiver critérios numerados ou bem delimitados, o agente pode extrair critérios incorretos. Mitigação: o agente apresenta os critérios extraídos para confirmação do revisor antes de iniciar
- **BAIXO — Respostas vagas do revisor:** O revisor pode responder "OK" sem ter verificado. O agente não tem como validar — qualidade depende do comprometimento do revisor
- **BAIXO — Spec desatualizada:** Se o código evoluiu além da spec, os critérios extraídos não cobrem a implementação real. Mitigação: o agente alerta sobre esse risco no início da revisão

## Perguntas em Aberto

Nenhuma — todas as decisões foram tomadas pelo SpecAgent com autonomia concedida pelo usuário.
