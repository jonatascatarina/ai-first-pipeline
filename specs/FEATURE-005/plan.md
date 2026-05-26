# Plano de Implementação — FEATURE-005

## Resumo Técnico

O `/speckit.review` é um comando de prompt puro em `.claude/commands/speckit.review.md`. Não há código executável — o comportamento é inteiramente definido por instruções ao agente. A implementação consiste em escrever um prompt que guia o agente através de um fluxo interativo de 4 fases: coleta de input, extração de critérios, revisão guiada por critério e geração do output final.

## Abordagem

### Fluxo Principal

```
Usuário executa /speckit.review
        |
        v
[Fase 1] Agente pergunta o ID da feature
        |
        v
[Fase 2] Agente lê specs/<ID>/spec.md
         Extrai critérios de aceite
         Apresenta lista para confirmação
        |
        v
[Fase 3] Para cada critério:
         - Apresenta o critério
         - Formula 1-2 perguntas específicas
         - Aguarda resposta: OK / PARCIAL / AUSENTE + observação
        |
        v
[Fase 4] Agente pergunta observações gerais
         Calcula veredicto
         Gera bloco Markdown copiável
        |
        v
Output entregue ao revisor
```

### Componentes Afetados

- `CLAUDE.md` — adicionar `/speckit.review` na tabela de comandos disponíveis
- `README.md` — adicionar `/speckit.review` na listagem de comandos

### Novos Componentes

- `.claude/commands/speckit.review.md` — o prompt completo do comando

### Modelo de Dados

Nenhuma mudança em schema ou persistência. Os dados fluem apenas na memória da sessão do agente.

### Contratos de Interface

**Input do usuário:**
- Feature ID (ex: `FEATURE-002`)
- Por critério: `OK`, `PARCIAL` ou `AUSENTE` + observação opcional em texto livre
- Observações gerais ao fim (texto livre, opcional)

**Output do agente:**
```markdown
## Revisao de Codigo — FEATURE-NNN
**Revisado por:** <nome do revisor ou "Revisor">
**Data:** <data>
**Modelo:** ReviewAgent (claude-sonnet-4-6)

### Conformidade com Spec

| # | Criterio de Aceite | Status | Observacao |
|---|-------------------|--------|------------|
| 1 | <resumo do criterio> | OK / PARCIAL / AUSENTE | <texto> |

### Observacoes Gerais
<texto do revisor ou "Nenhuma">

### Veredito
**APROVADO / BLOQUEADO / CONDICIONALMENTE APROVADO**

**Condicoes de aprovacao:** (se condicionalmente aprovado)
- <acao corretiva por critério PARCIAL>

**Criterios bloqueantes:** (se bloqueado)
- CA-N: <descricao do criterio ausente>
```

## Dependencias

### Dependencias Tecnicas

Nenhuma. Feature é inteiramente um prompt Markdown.

### Dependencias de Features

Nenhuma. O comando lê specs existentes, mas não depende de outra feature estar implementada.

### Dependencias de Dados

- `specs/<ID>/spec.md` deve existir com seção de critérios de aceite

## Riscos

### Riscos de Segurança

Nenhum risco identificado. O comando não executa código, não acessa sistemas externos e não persiste dados.

### Riscos de Performance

- **BAIXO:** Specs com mais de 10 critérios tornam a sessão longa. O prompt deve orientar o agente a manter perguntas concisas para não cansar o revisor.

### Riscos de Disponibilidade

Nenhum. Sem dependências externas.

### Riscos de Manutenibilidade

- **BAIXO:** O prompt codifica o formato de output. Se o formato mudar (ex: novas colunas na tabela), o prompt precisa ser atualizado manualmente. Mitigação: o formato está documentado neste plano como contrato de interface.

## Estrategia de Testes

### Testes Manuais

1. Executar `/speckit.review` com `FEATURE-002` como input
2. Responder OK a todos os critérios → verificar veredicto APROVADO
3. Executar novamente, responder AUSENTE a um critério → verificar veredicto BLOQUEADO
4. Executar novamente, responder PARCIAL a um critério → verificar veredicto CONDICIONALMENTE APROVADO com condição listada
5. Informar ID inexistente → verificar mensagem de erro e encerramento sem output

### Criterio de Conclusao dos Testes

Todos os 5 cenários acima produzem output correto e veredicto correspondente.

## Decisoes Tecnicas

- **Decisão:** Output apenas na tela, sem salvar arquivo
  - Alternativas: salvar em `specs/<ID>/review-<data>.md`
  - Justificativa: o artefato canônico é o comentário no PR, não um arquivo local. Salvar localmente criaria artefatos duplicados fora de sincronia com o GitHub
  - Consequências: sem histórico local de revisões — aceitável para o escopo atual

- **Decisão:** Agente não lê código diretamente — depende do revisor
  - Alternativas: agente lê diff via `git diff` ou arquivos do projeto
  - Justificativa: o comando é agnóstico ao repositório de código do produto; o template serve projetos onde o agente pode não ter acesso ao código
  - Consequências: qualidade da revisão depende do comprometimento do revisor — risco documentado

- **Decisão:** Confirmar critérios extraídos antes de iniciar a revisão
  - Alternativas: iniciar diretamente sem confirmação
  - Justificativa: specs podem estar desatualizadas; a confirmação é uma salvaguarda barata que evita revisar critérios errados
  - Consequências: adiciona um turno de interação — aceitável dado o ganho de confiabilidade

## Estimativa de Complexidade

| Area | Complexidade | Justificativa |
|------|-------------|---------------|
| Prompt (command file) | P | Fluxo linear com 4 fases bem definidas |
| Atualizacao de CLAUDE.md | P | Adicionar uma linha na tabela de comandos |
| Atualizacao de README.md | P | Adicionar uma linha na listagem |
| Testes manuais | P | 5 cenarios diretos, sem setup tecnico |
