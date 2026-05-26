# Resultados de Teste — FEATURE-002

**Data:** 2026-05-26
**Executado por:** AnalyzeAgent (claude-sonnet-4-6)
**Comando testado:** `/pr-checklist` (prompt-version: 1.0.0)

---

## Casos de Teste

### PR-A — Segurança dominante

**Input:**
```
Título: Add JWT authentication to /api/users endpoint
Descrição: Adds JWT-based authentication middleware to the /api/users route.
Users must send a Bearer token in the Authorization header. Tokens expire in 24h.
Invalid or expired tokens return 401. Admin role required for DELETE operations.
```

**Keywords detectadas:** `jwt`, `auth`, `token`, `role`, `endpoint`, `api`
**Categorias ativadas:** Segurança + Arquitetura + Documentação

**Output gerado:**

```markdown
## Segurança
- [ ] Verificar que token expirado retorna `401 Unauthorized`, não `500 Internal Server Error`
- [ ] Confirmar que o campo `role: admin` é verificado no payload do JWT, não apenas na presença do token
- [ ] Validar que o middleware rejeita tokens com algoritmo `alg: none` (vulnerabilidade clássica de JWT)
- [ ] Revisar que o secret de assinatura do JWT não está hardcoded — deve vir de variável de ambiente

## Testes
- [ ] Teste: requisição sem header `Authorization` retorna `401` com mensagem descritiva
- [ ] Teste: token expirado (gerado com `exp` no passado) retorna `401`, não `403`
- [ ] Teste: usuário sem role `admin` tentando `DELETE /api/users/:id` retorna `403`

## Arquitetura
- [ ] Confirmar que o middleware de JWT está posicionado antes dos handlers de `/api/users` na chain
- [ ] Verificar impacto em endpoints existentes de `/api/users` que eram públicos antes deste PR

## Documentação
- [ ] Atualizar a referência da API de `/api/users` com o header `Authorization: Bearer <token>` obrigatório
- [ ] CHANGELOG atualizado se esta mudança é visível ao usuário final
- [ ] Spec referenciada no PR se esta é uma feature nova
```

---

### PR-B — Baixo risco

**Input:**
```
Título: Fix typo in README.md
Descrição: Corrects "jonatascatarina" misspelled as "jonatas" in the quickstart command.
```

**Keywords detectadas:** `fix`
**Categorias ativadas:** Testes (mínimo)

**Output gerado:**

```markdown
## Segurança
- Nenhum risco identificado nesta categoria para este PR.

## Testes
- [ ] Verificar que o comando `gh repo create` com o nome corrigido funciona ao ser copiado e executado

## Arquitetura
- Nenhum risco identificado nesta categoria para este PR.

## Documentação
- [ ] Confirmar que não há outras ocorrências do typo "jonatas" em outros arquivos `.md`
- [ ] CHANGELOG atualizado se esta mudança é visível ao usuário final
- [ ] Spec referenciada no PR se esta é uma feature nova
```

---

### PR-C — Arquitetura e dados

**Input:**
```
Título: Migrate users table to add plan column
Descrição: Adds a `plan` column (enum: free, pro, enterprise) to the users table.
Default value is 'free'. Includes rollback migration. Backfills existing users to 'free'.
Adds index on (plan, created_at) for dashboard queries.
```

**Keywords detectadas:** `migration`, `schema`, `index`, `query`, `database`, `user`
**Categorias ativadas:** Arquitetura + Segurança

**Output gerado:**

```markdown
## Segurança
- [ ] Verificar que o backfill de `plan = 'free'` para usuários existentes não expõe dados de planos pagos incorretamente

## Testes
- [ ] Teste: migration de rollback remove a coluna `plan` sem erros em banco com dados reais
- [ ] Teste: novo usuário criado sem `plan` explícito recebe o default `'free'`

## Arquitetura
- [ ] Confirmar que a migration é reversível e o rollback foi testado em staging antes do merge
- [ ] Verificar que o índice em `(plan, created_at)` não impacta o tempo de escrita em tabela com muitas inserções
- [ ] Checar se o backfill de usuários existentes é feito em batches para evitar lock na tabela durante deploy

## Documentação
- [ ] Atualizar o schema de referência do banco (se existir) com a nova coluna `plan`
- [ ] CHANGELOG atualizado se esta mudança é visível ao usuário final
- [ ] Spec referenciada no PR se esta é uma feature nova
```

---

## Resultado dos Critérios de Aceite

| CA | Descrição | PR-A | PR-B | PR-C |
|----|-----------|------|------|------|
| CA-1 | PR de JWT gera ≥ 3 itens específicos de segurança | PASS (4 itens) | - | - |
| CA-2 | 4 seções obrigatórias presentes | PASS | PASS | PASS |
| CA-3 | 2 itens fixos (CHANGELOG e Spec) presentes | PASS | PASS | PASS |
| CA-4 | Título < 5 chars solicita mais informações | não testado | - | - |
| CA-5 | Output é Markdown válido com `- [ ]` e `##` | PASS | PASS | PASS |
| CA-6 | Itens mencionam elementos específicos do PR | PASS ("JWT", "alg: none", "admin") | PASS (comando copiável) | PASS ("plan", "batches", "staging") |

**Veredicto: APROVADO** — 5 de 5 critérios testados passaram. CA-4 pendente de teste manual interativo.

---

## Observações

- PR-B demonstrou corretamente o comportamento de seção vazia: Segurança e Arquitetura exibem "Nenhum risco identificado" em vez de itens genéricos
- PR-A gerou item de segurança sobre `alg: none` (vulnerabilidade real de JWT) — indica que o few-shot está calibrando o modelo corretamente para itens específicos e de alto valor
- PR-C gerou item sobre backfill em batches — contexto inferido corretamente a partir da descrição, sem keyword explícita
