# Plano de Implementação — EXAMPLE-001

## Resumo Técnico

O rate limiting será implementado como middleware HTTP inserido no início da chain de middlewares, antes de autenticação de negócio e handlers. O algoritmo escolhido é sliding window com contadores no Redis, implementado de forma atômica via script Lua para evitar race conditions. Em caso de falha do Redis (timeout > 50ms ou conexão recusada), o sistema opera em fail-open: requisições passam e um alerta é logado.

## Abordagem

### Fluxo Principal

```
Request
  │
  ▼
[RateLimitMiddleware]
  ├── Extrai identidade (user_id do JWT ou IP)
  ├── Verifica tipo de token (service → skip)
  ├── Consulta limite do plano do usuário
  ├── Executa script Lua no Redis (atômico)
  │     ├── Obtém contagem na janela deslizante
  │     ├── Se count < limit → incrementa, retorna (count, ttl)
  │     └── Se count >= limit → retorna (count, ttl) sem incrementar
  ├── Define headers X-RateLimit-* em toda resposta
  ├── Se limite atingido → retorna 429 com Retry-After
  └── Se Redis falhou → fail-open + log WARN
  │
  ▼
[Próximo Middleware / Handler]
```

### Algoritmo Sliding Window com Redis

A sliding window é implementada com uma sorted set no Redis:

```
Chave: rate_limit:{user_id}
Membros: timestamps das requisições (Unix ms)
Score: o próprio timestamp

Operação (script Lua atômico):
1. ZREMRANGEBYSCORE chave 0 (agora - janela_ms)  ← remove expirados
2. count = ZCARD chave
3. Se count < limite:
   ZADD chave agora agora
   EXPIRE chave janela_segundos
   retorna (count + 1, limite - count - 1, 0)
4. Se count >= limite:
   ttl_mais_antigo = ZRANGE chave 0 0 WITHSCORES [0]
   retorna (count, 0, ttl_mais_antigo + janela_ms - agora)
```

### Componentes Afetados

- **Middleware chain** — inserção do `RateLimitMiddleware` como primeiro middleware após parsing de headers
- **JWT middleware** — precisa expor `user_id` e `token_type` no contexto da request (provavelmente já faz, verificar)
- **User service / repository** — precisa de método `GetUserPlan(user_id) → Plan` (pode ser cacheado por 60s)

### Novos Componentes

- `RateLimitMiddleware` — intercepta todas as requests, verifica e incrementa contadores
- `RateLimitStore` — interface + implementação Redis para operações de contagem
- `RateLimitConfig` — leitura de limites das variáveis de ambiente
- Script Lua `sliding_window.lua` — lógica atômica de verificação/incremento

### Modelo de Dados

Não há mudanças em banco de dados relacional.

Redis (por usuário autenticado):
```
Chave:  rate_limit:user:{user_id}
Tipo:   Sorted Set
TTL:    3600s (renovado a cada requisição)
Membros: timestamps em Unix milliseconds
Tamanho estimado: ~50 bytes × 1000 req = ~50KB por usuário ativo
```

Redis (por IP não autenticado):
```
Chave:  rate_limit:ip:{ip_address}
Tipo:   Sorted Set
TTL:    3600s
```

### Contratos de Interface

Headers em toda resposta:
```
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 743
X-RateLimit-Reset: 1748300400
```

Resposta 429:
```
HTTP/1.1 429 Too Many Requests
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1748300400
Retry-After: 1847
Content-Type: application/json

{"error": "rate_limit_exceeded", "message": "Too many requests. Try again in 1847 seconds."}
```

## Dependências

### Dependências Técnicas

- Redis 6.0+ (sorted sets + scripts Lua)
- Biblioteca Redis do projeto (verificar qual está em uso)
- JWT middleware existente expondo `user_id` e opcionalmente `token_type`

### Dependências de Features

- Autenticação JWT deve estar funcionando (pré-existente)
- Tabela `users` com coluna `plan` deve existir (pré-existente)

### Dependências de Dados

- Nenhuma migration necessária
- Redis deve estar provisionado no ambiente (verificar se já existe)

## Riscos

### Riscos de Segurança

- **MÉDIO — Bypass via token rotation:** Um atacante que gera novos tokens frequentemente poderia contornar o rate limit se o identificador fosse o token e não o `user_id`. Mitigação: identificador é sempre o `user_id`, não o token.
- **BAIXO — IP spoofing via X-Forwarded-For:** Se um atacante controlar o header `X-Forwarded-For`, pode simular diferentes IPs. Mitigação: o Nginx sobrescreve este header — o middleware confia no Nginx.
- **BAIXO — Enumeração de planos:** Os headers `X-RateLimit-Limit` revelam o plano do usuário indiretamente. Aceito — esta informação não é sensível.

### Riscos de Performance

- **ALTO — Latência do Redis:** Toda requisição faz um round-trip ao Redis. Com Redis local, P99 é ~1ms. Com Redis remoto, pode ser 5-15ms. Mitigação: configurar timeout de 50ms; Redis deve estar na mesma rede/AZ.
- **MÉDIO — GetUserPlan a cada request:** Consulta ao banco para obter o plano do usuário a cada requisição. Mitigação: cache em memória com TTL de 60s (aceitável — mudança de plano tem latência de 1 minuto).
- **BAIXO — Crescimento do sorted set:** Um usuário que faz requisições por semanas sem parar cresceria o sorted set. Mitigado pelo ZREMRANGEBYSCORE que limpa entradas antigas a cada request.

### Riscos de Disponibilidade

- **ALTO — Redis como SPF:** Se Redis ficar indisponível, o modo fail-open garante continuidade. Monitorar taxa de erros Redis para detectar degradação.
- **BAIXO — Thundering herd após Redis recovery:** Quando Redis volta após outage prolongado, os contadores estão zerados — todos os usuários têm limite completo por 1 hora.

### Riscos de Manutenibilidade

- **BAIXO — Script Lua acoplado ao Redis:** Mudança de provider de cache exigiria reescrever a lógica atômica. Mitigado pela interface `RateLimitStore` que abstrai a implementação.

## Estratégia de Testes

### Testes Unitários

- `RateLimitConfig`: parsing correto de env vars, valores padrão, validação
- `RateLimitMiddleware`: headers corretos em request normal, headers corretos em 429, skip para service tokens, fail-open quando store retorna erro
- Script Lua (testado via Redis de teste): atomicidade, contagem correta, expiração correta

### Testes de Integração

- Cenário completo: usuário `free` atinge limite, recebe 429, espera reset, volta a funcionar
- Concorrência: 10 requests simultâneos com 1 slot restante — exatamente 1 passa
- Redis offline: middleware não bloqueia requests, loga warning
- Plano `pro` tem limite correto (5.000)
- IP não autenticado tem limite de 100

### Testes de Carga

- 10.000 usuários simulando 100 req/s cada por 60 segundos
- Critério: P99 de latência do middleware < 5ms
- Ferramenta sugerida: k6

## Decisões Técnicas

- **Decisão:** Sorted Set no Redis em vez de contador simples com INCR
  - **Alternativas:** INCR com EXPIRE (não permite sliding window real), contador com timestamp em campo de hash
  - **Justificativa:** Sorted set permite calcular exatamente quantas requests ocorreram nos últimos N segundos, habilitando sliding window precisa
  - **Consequências:** Uso ligeiramente maior de memória (~50KB/usuário ativo vs ~16 bytes); complexidade da lógica Lua

- **Decisão:** Script Lua para atomicidade
  - **Alternativas:** Transação Redis (WATCH/MULTI/EXEC), lock distribuído
  - **Justificativa:** Scripts Lua são executados atomicamente pelo Redis sem necessidade de retries; mais simples que transações otimistas
  - **Consequências:** Lógica de rate limit fica no Lua, requer Redis com suporte a scripts (Redis 2.6+, temos 6.0)

- **Decisão:** Fail-open em caso de falha do Redis
  - **Alternativas:** Fail-closed (retornar 503)
  - **Justificativa:** Decisão de negócio — disponibilidade tem prioridade sobre controle de abuso durante outages
  - **Consequências:** Durante outage do Redis, rate limiting é desabilitado; risco aceito e documentado

## Estimativa de Complexidade

| Área | Complexidade | Justificativa |
|------|-------------|---------------|
| Backend | M | Script Lua + middleware + cache de plano |
| Frontend | - | Não aplicável |
| Infra/DevOps | P | Redis provavelmente já existe; adicionar vars de ambiente |
| Testes | M | Testes de concorrência e integração exigem cuidado |
