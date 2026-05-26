# Tasks — EXAMPLE-001

## Dependências entre Tasks

```
T1 → T3 → T5 → T7 → T9 → T11
     T4 ↗
T2 → T6 → T8 → T10 → T11
               T11 → T12
```

T1 e T2 podem ser executadas em paralelo.
T3 e T4 dependem de T1 e T2 respectivamente, mas podem ser em paralelo entre si.
T11 depende de T7, T8, T9, T10 — é o teste de integração final.

---

## T1 — Criar interface e config de RateLimitStore [SETUP] [P]

**O que fazer:**
Criar a interface `RateLimitStore` que abstrai as operações de contagem de rate limit, e `RateLimitConfig` que lê os limites das variáveis de ambiente.

A interface deve ter pelo menos:
- `Check(ctx, key, limit, windowSeconds) → (count, remaining, resetAt, error)`
- `Close() → error`

`RateLimitConfig` lê as variáveis:
- `RATE_LIMIT_FREE` (padrão: 1000)
- `RATE_LIMIT_PRO` (padrão: 5000)
- `RATE_LIMIT_ENTERPRISE` (padrão: 50000)
- `REDIS_RATE_LIMIT_TIMEOUT_MS` (padrão: 50)

**Contexto necessário:**
- `specs/EXAMPLE-001/plan.md` seção "Contratos de Interface" e "Novos Componentes"
- `constitution.md` Artigo IV

**Critério de conclusão:**
- [ ] Interface `RateLimitStore` definida com tipagem completa
- [ ] `RateLimitConfig` lê todas as 4 variáveis de ambiente
- [ ] Valores padrão aplicados quando variável ausente
- [ ] Teste unitário: config com env vars ausentes usa defaults
- [ ] Teste unitário: config com env vars presentes usa os valores corretos

**Testes esperados:**
- Unitário: `NewRateLimitConfig()` com env vazia retorna limites padrão (1000/5000/50000)
- Unitário: `NewRateLimitConfig()` com `RATE_LIMIT_PRO=9999` retorna 9999 para pro

**Não inclui:**
Implementação Redis — apenas a interface e a config.

---

## T2 — Verificar exposição de user_id no JWT middleware [SETUP] [P]

**O que fazer:**
Auditar o JWT middleware existente para confirmar que `user_id` e `token_type` estão disponíveis no contexto da requisição. Se não estiverem, adicionar sem quebrar comportamento existente.

Documentar como acessar: `ctx.Value(UserIDKey)` ou equivalente.

**Contexto necessário:**
- `specs/EXAMPLE-001/spec.md` RF-1, RF-3 (sobre tokens de serviço)
- `specs/EXAMPLE-001/perguntas-respondidas.md` P3

**Critério de conclusão:**
- [ ] `user_id` acessível via contexto da request após JWT middleware
- [ ] `token_type` acessível via contexto (ou ausente = "user")
- [ ] Sem quebra de comportamento em requests existentes
- [ ] Comentário inline indicando como acessar os valores

**Testes esperados:**
- Unitário: JWT middleware com token válido popula `user_id` no contexto
- Unitário: JWT middleware com service token popula `token_type: service`

**Não inclui:**
Nenhuma mudança na lógica de autenticação — apenas auditoria e exposição de contexto.

---

## SEC-T3 — Implementar script Lua de sliding window [CORE] [M]

**O que fazer:**
Escrever e testar o script Lua que executa atomicamente no Redis a lógica de sliding window. O script deve:

1. Remover entradas mais antigas que `agora - janela_ms` (ZREMRANGEBYSCORE)
2. Contar entradas restantes (ZCARD)
3. Se count < limit: adicionar timestamp atual (ZADD), renovar TTL (EXPIRE), retornar `{count+1, remaining, 0}`
4. Se count >= limit: calcular quando a entrada mais antiga expira, retornar `{count, 0, reset_ms}`

O script deve ser testado contra um Redis de teste (não mock).

**Contexto necessário:**
- `specs/EXAMPLE-001/plan.md` seção "Algoritmo Sliding Window com Redis"

**Critério de conclusão:**
- [ ] Script Lua implementado e carregado no binário via embed ou string
- [ ] Teste: 5 requests sequenciais com limit=5 — todos passam, 6ª bloqueada
- [ ] Teste: 5 requests simultâneas com limit=1 — exatamente 1 passa (teste de atomicidade)
- [ ] Teste: após janela expirar, contador reseta corretamente
- [ ] Teste: `reset_ms` retornado corresponde ao tempo da requisição mais antiga + janela

**Testes esperados:**
- Integração (Redis real): concorrência com goroutines — sem over-count
- Integração: TTL da chave nunca excede `window_seconds + 1`

**Não inclui:**
Integração com middleware HTTP — apenas a lógica Lua e seu wrapper Go/Python/etc.

---

## T4 — Criar cache de plano de usuário [CORE] [P]

**O que fazer:**
Implementar `UserPlanCache` que consulta o plano do usuário (free/pro/enterprise) com cache em memória de 60 segundos. Interface:
- `GetPlan(ctx, user_id) → (Plan, error)`

Na ausência de registro, retornar plano `free`.

**Contexto necessário:**
- `specs/EXAMPLE-001/perguntas-respondidas.md` P2 e P8
- `specs/EXAMPLE-001/plan.md` seção "Riscos de Performance"

**Critério de conclusão:**
- [ ] Cache hit não faz query ao banco
- [ ] Cache miss faz query e armazena resultado por 60s
- [ ] Entrada de cache expirada dispara nova query
- [ ] Erro de banco retorna plano `free` + log de warning
- [ ] Teste unitário com mock do banco

**Testes esperados:**
- Unitário: 3 calls seguidas ao mesmo user_id → 1 query ao banco
- Unitário: call após 60s de TTL → nova query ao banco
- Unitário: banco offline → retorna `free`, não propaga erro

**Não inclui:**
Invalidação proativa de cache quando plano muda — isso está explicitamente no não-escopo.

---

## T5 — Implementar RedisRateLimitStore [CORE] [M]

**O que fazer:**
Implementar a interface `RateLimitStore` usando Redis. A implementação deve:
- Usar o script Lua de SEC-T3
- Respeitar timeout de `REDIS_RATE_LIMIT_TIMEOUT_MS`
- Retornar erro específico `ErrStoreUnavailable` em caso de timeout ou conexão recusada
- Log de WARN quando `ErrStoreUnavailable` é retornado

**Contexto necessário:**
- `specs/EXAMPLE-001/plan.md` seção "Modelo de Dados" (estrutura de chaves Redis)
- T1 (interface), SEC-T3 (script Lua)

**Critério de conclusão:**
- [ ] Implementa interface `RateLimitStore` completamente
- [ ] Chave Redis segue padrão `rate_limit:user:{user_id}` e `rate_limit:ip:{ip}`
- [ ] Timeout aplicado via context com deadline
- [ ] `ErrStoreUnavailable` retornado em timeout ou conexão recusada
- [ ] Log WARN emitido com `user_id` e tipo de erro ao retornar `ErrStoreUnavailable`
- [ ] Teste de integração contra Redis real

**Testes esperados:**
- Integração: operação normal retorna contagem correta
- Integração: Redis com timeout de 1ms → retorna `ErrStoreUnavailable`
- Unitário: chave gerada corretamente para user_id e para IP

**Não inclui:**
Lógica de failover ou retry — o middleware decide o que fazer com `ErrStoreUnavailable`.

---

## T6 — Implementar RateLimitMiddleware [CORE] [M]

**O que fazer:**
Implementar o middleware HTTP que:
1. Extrai identidade: `user_id` do contexto (se autenticado) ou IP do `X-Forwarded-For`
2. Verifica `token_type` — se `service`, passa sem verificação
3. Consulta plano via `UserPlanCache` para obter o limite correto
4. Chama `RateLimitStore.Check()`
5. Em sucesso: adiciona headers `X-RateLimit-*` e passa para o próximo handler
6. Em 429: retorna resposta JSON com `Retry-After`
7. Em `ErrStoreUnavailable`: fail-open, loga WARN, passa para o próximo handler

**Contexto necessário:**
- `specs/EXAMPLE-001/plan.md` seção "Fluxo Principal" e "Contratos de Interface"
- T1 (config/interface), T4 (cache de plano), T5 (store Redis)
- `specs/EXAMPLE-001/perguntas-respondidas.md` P3 (service tokens), P6 (X-Forwarded-For)

**Critério de conclusão:**
- [ ] Service tokens passam sem verificação
- [ ] Headers `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset` presentes em toda resposta (incluindo 429)
- [ ] Resposta 429 inclui `Retry-After` com valor correto em segundos
- [ ] Corpo da resposta 429 é JSON `{"error": "rate_limit_exceeded", "message": "..."}`
- [ ] Fail-open quando store retorna `ErrStoreUnavailable`
- [ ] Testes unitários com mock do store

**Testes esperados:**
- Unitário: request normal → headers corretos, próximo handler chamado
- Unitário: request com limite atingido → 429 com headers e Retry-After
- Unitário: store retorna erro → próximo handler chamado (fail-open)
- Unitário: service token → próximo handler chamado sem chamar store

**Não inclui:**
Integração com middleware chain real — isso é T7.

---

## T7 — Integrar middleware na chain HTTP [API] [P]

**O que fazer:**
Inserir o `RateLimitMiddleware` como primeiro middleware após parsing de headers, antes de autenticação de negócio e handlers de rota. Garantir que é aplicado a todas as rotas da API.

**Contexto necessário:**
- `specs/EXAMPLE-001/plan.md` seção "Componentes Afetados"
- T6 (middleware implementado)

**Critério de conclusão:**
- [ ] Middleware registrado antes dos handlers de negócio
- [ ] Todas as rotas existentes continuam funcionando (teste de regressão)
- [ ] Rate limiting ativo em ambiente de desenvolvimento
- [ ] Variáveis de ambiente documentadas no README ou `.env.example`

**Testes esperados:**
- Integração: request à qualquer rota existente inclui headers `X-RateLimit-*`
- Integração: request sem token recebe limite de IP (100/h)

**Não inclui:**
Deploy em produção — apenas integração local.

---

## SEC-T8 — Adicionar variáveis de ambiente ao template de configuração [INFRA] [P]

**O que fazer:**
Adicionar as quatro variáveis de rate limit ao `.env.example` (ou equivalente do projeto) com comentários explicativos e valores padrão.

```
# Rate limiting
RATE_LIMIT_FREE=1000          # Requisições por hora para plano free
RATE_LIMIT_PRO=5000           # Requisições por hora para plano pro
RATE_LIMIT_ENTERPRISE=50000   # Requisições por hora para plano enterprise
REDIS_RATE_LIMIT_TIMEOUT_MS=50 # Timeout para operações Redis (fail-open acima disso)
```

**Contexto necessário:**
- `specs/EXAMPLE-001/perguntas-respondidas.md` P8
- T1 (RateLimitConfig)

**Critério de conclusão:**
- [ ] Todas as 4 variáveis em `.env.example` com comentário e valor padrão
- [ ] CHANGELOG ou commit message referencia EXAMPLE-001

**Testes esperados:**
- Manual: rodar com valores padrão funciona sem Redis configurado (fail-open)

**Não inclui:**
Configuração de Redis em produção ou staging.

---

## T9 — Testes unitários completos do middleware [TEST] [M]

**O que fazer:**
Escrever suite de testes unitários cobrindo todos os cenários do middleware e store com mocks. Cobrir especificamente os cenários dos critérios de aceite da spec.

**Critérios de aceite a cobrir:**
- CA-1: usuário free, 1001 requisições → 1001ª retorna 429
- CA-2: usuário pro, 5001 requisições → 5001ª retorna 429
- CA-3: headers presentes em toda resposta
- CA-4: 429 inclui Retry-After correto
- CA-5: Redis offline → fail-open (sem 429)
- CA-8: IP não autenticado limitado a 100/h

**Contexto necessário:**
- `specs/EXAMPLE-001/spec.md` seção "Critérios de Aceite"
- T6 (middleware), T5 (store)

**Critério de conclusão:**
- [ ] Todos os 8 critérios de aceite têm pelo menos um teste unitário
- [ ] Cobertura de linha do middleware acima de 90%
- [ ] Todos os testes passam em CI

**Testes esperados:**
- Como descrito acima, um por critério de aceite

**Não inclui:**
Testes de concorrência (cobertura de T10) ou testes de carga (T11).

---

## T10 — Testes de concorrência (CA-6) [TEST] [M]

**O que fazer:**
Escrever teste de concorrência que verifica o critério de aceite CA-6: "dado dois requests simultâneos do mesmo usuário com 1 requisição restante no limite, no máximo um deles passa".

O teste deve usar goroutines ou threads para simular concorrência real contra um Redis de teste (não mock — mocks não testam atomicidade).

**Contexto necessário:**
- `specs/EXAMPLE-001/spec.md` CA-6
- SEC-T3 (script Lua atômico)

**Critério de conclusão:**
- [ ] Teste usa Redis real (não mock)
- [ ] 10 goroutines simultâneas com 1 slot restante → exatamente 1 passa
- [ ] Teste é determinístico (não flaky) em 10 execuções consecutivas
- [ ] Documentar como executar o teste localmente (requer Redis)

**Testes esperados:**
- Concorrência: 10 requests simultâneos com `remaining=1` → `passcount == 1`

**Não inclui:**
Testes de carga de alta escala (isso é T11).

---

## T11 — Testes de integração end-to-end [TEST] [M]

**O que fazer:**
Escrever testes de integração que exercitam o fluxo completo: request HTTP → middleware → Redis → resposta. Incluir:
- Fluxo completo para usuário free atingindo limite
- Fluxo completo para usuário pro com limite diferente
- Comportamento quando Redis está offline (usando Redis que não responde)
- Janela deslizante: requisições antigas expiram e novo espaço é liberado

**Contexto necessário:**
- `specs/EXAMPLE-001/spec.md` CA-1 a CA-8
- T7 (middleware integrado na chain)

**Critério de conclusão:**
- [ ] Testes usam servidor HTTP real (test server) + Redis real
- [ ] CA-1, CA-2, CA-3, CA-4, CA-5, CA-7, CA-8 cobertos
- [ ] Cleanup correto de chaves Redis entre testes
- [ ] Testes podem ser executados em CI com Redis disponível

**Testes esperados:**
- E2E: usuário free faz 1001 requests → 1001ª é 429, headers corretos em todas
- E2E: usuário muda de plano → próxima request reflete novo limite (após 60s de cache)

**Não inclui:**
Testes de carga / performance — escopo separado se necessário.

---

## T12 — Atualizar CHANGELOG e documentação [DOCS] [P]

**O que fazer:**
Registrar a feature de rate limiting no CHANGELOG e atualizar a documentação da API (se existir) com os novos headers e o comportamento do 429.

Formato CHANGELOG (seguir o existente no projeto):
```
### Added
- Rate limiting por usuário (EXAMPLE-001): sliding window por hora,
  limites configuráveis por plano (free/pro/enterprise), fail-open em
  caso de indisponibilidade do Redis
```

**Contexto necessário:**
- `specs/EXAMPLE-001/spec.md` para descrever a feature com precisão

**Critério de conclusão:**
- [ ] CHANGELOG atualizado com entrada na seção `[Unreleased]`
- [ ] Documentação da API (OpenAPI/README) inclui descrição dos headers `X-RateLimit-*`
- [ ] Documentação menciona o comportamento do `429` e o campo `Retry-After`

**Testes esperados:**
- Revisão manual: documentação está correta e completa

**Não inclui:**
Comunicado externo ou release notes para clientes.
