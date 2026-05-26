# Plano de Implementação — FEATURE-002

## Resumo Técnico

Esta feature é implementada inteiramente como um arquivo Markdown em `.claude/commands/pr-checklist.md`. Não há código, dependências ou infraestrutura — apenas o prompt que instrui o agente a gerar o checklist. A qualidade do output depende diretamente da qualidade do prompt.

## Abordagem

### Componente único

**Arquivo:** `.claude/commands/pr-checklist.md`

O arquivo define o comando `/pr-checklist` para Claude Code. Quando executado, o agente lê o prompt, solicita título e descrição do PR ao usuário, detecta categorias de risco por keywords, e gera o checklist categorizado em Markdown dentro de um bloco copiável.

### Estratégia de detecção de contexto

O prompt usa few-shot: apresenta exemplos de bom e mau item para calibrar o modelo antes da geração. A detecção de keywords é feita pelo próprio LLM — sem regex ou lógica explícita.

Tabela de keywords → categorias:

| Keyword detectada | Categoria ativada | Exemplos de itens gerados |
|------------------|------------------|--------------------------|
| auth, jwt, token, senha, permissão, role | Segurança | Verificar expiração de token, escopo de permissões |
| banco, migration, schema, query, índice | Arquitetura | Verificar rollback da migration, impacto em queries existentes |
| fix, bug, regressão, comportamento | Testes | Teste de regressão para o caso corrigido |
| api, endpoint, contrato, breaking | Arquitetura + Docs | Versioning, atualização de OpenAPI |
| dependência, package, library, upgrade | Segurança + Arquitetura | CVE check, licença, breaking changes |

### Exemplo de input/output

**Input:**
```
Título: Add rate limiting middleware to all API endpoints
Descrição: Implements sliding window rate limiting using Redis. Each user gets 1000 req/hour.
Free plan users are limited to 100 req/hour. Returns 429 with Retry-After header.
```

**Output gerado pelo agente:**

```markdown
## Segurança
- [ ] Verificar que o rate limit por IP também cobre requisições não autenticadas
- [ ] Confirmar que o header `Retry-After` não vaza informações sobre a estrutura interna
- [ ] Validar comportamento quando Redis está indisponível (fail-open documentado?)
- [ ] Revisar se tokens de serviço estão excluídos do rate limit conforme spec

## Testes
- [ ] Teste de concorrência: duas requisições simultâneas com 1 slot restante — exatamente 1 passa
- [ ] Teste de integração: usuário free atinge 100/h e recebe 429 com Retry-After correto
- [ ] Teste de regressão: endpoints existentes continuam respondendo com os novos headers

## Arquitetura
- [ ] Verificar que o middleware está posicionado antes dos handlers de negócio
- [ ] Confirmar que Redis como dependência de runtime está documentado no README
- [ ] Revisar impacto de latência: middleware deve adicionar < 5ms ao P99

## Documentação
- [ ] Headers `X-RateLimit-*` documentados na referência da API
- [ ] Comportamento do 429 e campo `Retry-After` descritos para consumidores
- [ ] CHANGELOG atualizado com a mudança visível ao usuário
- [ ] Spec FEATURE-001 referenciada no PR
```

## Dependências

- Nenhuma dependência técnica
- Claude Sonnet 4.6 como modelo executor (CLAUDE.md)

## Riscos

### Riscos de qualidade do prompt
- **MÉDIO — Itens genéricos:** Se o prompt não tiver exemplos few-shot suficientes, o modelo gera itens óbvios. Mitigação: incluir seção de exemplos ruins explícitos com instrução para evitá-los.
- **BAIXO — Falso positivo de keyword:** O modelo detecta "token" em contexto irrelevante e gera itens de segurança desnecessários. Aceitável — item extra irrelevante é melhor que item crítico faltando.

### Riscos de manutenibilidade
- **BAIXO — Prompt coupling:** Se as categorias mudarem, o prompt precisa ser atualizado manualmente. Sem mecanismo de teste automatizado para qualidade do output.

## Estimativa de Complexidade

| Área | Complexidade | Justificativa |
|------|-------------|---------------|
| Prompt engineering | M | Few-shot calibration + keyword detection requer iteração |
| Estrutura do arquivo | P | Formato já definido pelos outros comandos /speckit.* |
| Testes | P | Testes manuais com 3-5 exemplos representativos |
