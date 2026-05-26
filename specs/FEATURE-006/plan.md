# Plano de Implementação — FEATURE-006

## Resumo Técnico

A feature entrega dois artefatos independentes: um arquivo de template para o GitHub e um comando de prompt para o agente. O template é um arquivo Markdown estático colocado na localização correta do repositório. O comando `/speckit.issue` é um prompt que instrui o agente a usar o `gh` CLI para interagir com a API do GitHub — sem código executável adicional.

## Abordagem

### Fluxo — Issue Template

```
Contribuidor abre novo Issue no GitHub
        |
        v
GitHub detecta .github/ISSUE_TEMPLATE/feature-spec.md
        |
        v
Interface do GitHub exibe template com seções preenchíveis
        |
        v
Contribuidor preenche e submete o Issue
        |
        v
Issue criado com label "spec" e prefixo "[SPEC]" no título
```

### Fluxo — Modo Publish (spec → Issue)

```
Usuário executa /speckit.issue
        |
        v
Agente pergunta: modo publish ou scaffold?
        |
        v (publish)
Agente pergunta: qual feature? (ex: FEATURE-005)
        |
        v
Agente lê specs/<ID>/spec.md
Se não existir → erro, encerra
        |
        v
Agente extrai: título, contexto, problema, solução, critérios de aceite
        |
        v
Agente apresenta preview do Issue ao usuário e pede confirmação
        |
        v
gh issue create --title "..." --body "..." --label "spec"
Se label não existir → gh issue create sem --label, avisa usuário
        |
        v
Agente retorna URL do Issue criado
```

### Fluxo — Modo Scaffold (Issue → spec)

```
Usuário executa /speckit.issue
        |
        v
Agente pergunta: modo publish ou scaffold?
        |
        v (scaffold)
Agente pergunta: número do Issue?
        |
        v
gh issue view <número> --json title,body,labels
Se erro → informa usuário, encerra
        |
        v
Agente extrai campos do body do Issue
(heurística: seções Markdown; campos ausentes → marcador TODO)
        |
        v
Agente lista IDs existentes em specs/, propõe próximo ID
Aguarda confirmação do usuário
        |
        v
Se specs/<ID>/spec.md existir → alerta, pede confirmação para sobrescrever
        |
        v
Cria specs/<ID>/spec.md com conteúdo extraído
        |
        v
gh issue comment <número> --body "Spec criada em specs/<ID>/spec.md..."
        |
        v
Agente informa: arquivo criado, próximo passo é /speckit.clarify
```

### Componentes Afetados

- `CLAUDE.md` — adicionar `/speckit.issue` na tabela de comandos
- `README.md` — mencionar Issue Template na seção de estrutura e quickstart
- `CHANGELOG.md` — entrada na seção `[Não lançado]`

### Novos Componentes

- `.github/ISSUE_TEMPLATE/feature-spec.md` — template de Issue para features
- `.claude/commands/speckit.issue.md` — prompt do comando bidirecional

### Contratos de Interface

**Body do Issue gerado pelo modo `publish`:**

```markdown
## Contexto
<extraído da spec>

## Problema
<extraído da spec>

## Solução Proposta
<extraído da spec>

## Critérios de Aceite
<lista numerada extraída da spec>

## Não-Escopo
<extraído da spec>

---
Spec completa: [specs/<ID>/spec.md](<link para o arquivo no repositório>)
```

**Comentário adicionado no Issue pelo modo `scaffold`:**

```
Spec criada em `specs/<ID>/spec.md` a partir deste Issue.

Próximo passo: execute `/speckit.clarify` para responder perguntas de clarificação antes de avançar para o plano.
```

## Dependências

### Dependências Técnicas

- `gh` CLI instalado e autenticado com escopo `repo` (necessário para criar Issues e comentários)
- Repositório hospedado no GitHub

### Dependências de Features

Nenhuma — os dois artefatos são independentes entre si e de outras features.

## Riscos

### Riscos de Segurança

Nenhum risco identificado. O comando usa `gh` CLI autenticado; não há manipulação de credenciais ou dados sensíveis.

### Riscos de Performance

- **BAIXO:** `gh issue view` e `gh issue create` fazem chamadas à API do GitHub com latência de 200-500ms. Aceitável para uso interativo.

### Riscos de Disponibilidade

- **BAIXO:** Se o GitHub estiver fora do ar ou o `gh` não estiver autenticado, o comando falha com mensagem de erro do próprio `gh`. O agente deve detectar erro e orientar o usuário a verificar a autenticação.

### Riscos de Manutenibilidade

- **BAIXO:** A extração de campos do Issue no modo `scaffold` usa heurística baseada em títulos de seção Markdown. Issues com formatação não-padrão produzirão scaffolds incompletos — comportamento esperado e documentado (marcadores TODO).

## Estratégia de Testes

### Testes Manuais

1. Abrir novo Issue no GitHub e confirmar que o template aparece automaticamente
2. Executar `/speckit.issue` modo `publish` com `FEATURE-002` → verificar Issue criado com campos corretos e link para spec
3. Executar `/speckit.issue` modo `publish` com ID inexistente → verificar mensagem de erro sem criar Issue
4. Executar `/speckit.issue` modo `scaffold` com Issue que usou o template → verificar spec gerada com campos mapeados
5. Executar `/speckit.issue` modo `scaffold` com Issue de texto livre → verificar spec gerada com marcadores TODO
6. Executar `/speckit.issue` modo `scaffold` com número inexistente → verificar erro sem criar arquivo

### Critério de Conclusão dos Testes

Todos os 6 cenários acima produzem o resultado esperado. Resultados documentados em `specs/FEATURE-006/test-results.md`.

## Decisões Técnicas

- **Decisão:** Formato Markdown legado para o Issue Template (não formulário YAML)
  - Alternativas: `issue_form.yml` com campos estruturados
  - Justificativa: Markdown puro é o princípio do projeto; suporte universal; sem dependência de features do GitHub que podem variar por plano
  - Consequências: sem validação obrigatória de campos; usuário pode submeter Issue com seções vazias

- **Decisão:** Preview + confirmação antes de criar o Issue no modo `publish`
  - Alternativas: criar diretamente sem preview
  - Justificativa: criação de Issue é ação pública e irreversível (Issues não podem ser deletados, apenas fechados); confirmação evita Issues com informação incorreta
  - Consequências: um turno extra de interação

- **Decisão:** Agente propõe próximo ID e aguarda confirmação no modo `scaffold`
  - Alternativas: criar automaticamente sem perguntar
  - Justificativa: o usuário pode querer usar um ID específico (ex: já planejou que é FEATURE-007); confirmar é mais seguro
  - Consequências: um turno extra de interação

## Estimativa de Complexidade

| Area | Complexidade | Justificativa |
|------|-------------|---------------|
| Issue Template | P | Arquivo Markdown estático com instruções |
| Comando /speckit.issue | M | Dois modos com fluxos distintos; heurística de extração do modo scaffold |
| Atualizações de docs | P | CLAUDE.md, README.md, CHANGELOG.md |
| Testes manuais | P | 6 cenários diretos |
