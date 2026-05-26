# ADR-001 — Sliding Window com Redis Sorted Set para Rate Limiting

**Status:** Aprovado
**Data:** 2026-05-26
**Autores:** PlanAgent (EXAMPLE-001)
**Relacionado a:** EXAMPLE-001 — Rate Limiting por Usuário

---

## Contexto

A feature EXAMPLE-001 exige rate limiting por usuário com algoritmo sliding window. Durante o planejamento, foram avaliadas três abordagens para implementação da contagem de requisições.

A escolha do algoritmo e da estrutura de dados de armazenamento tem impacto em: precisão do rate limiting, uso de memória no Redis, complexidade do código, e comportamento sob concorrência.

---

## Decisão

Usar **Redis Sorted Set** com **script Lua para atomicidade** como mecanismo de sliding window.

Cada usuário tem uma chave `rate_limit:user:{user_id}` no Redis do tipo Sorted Set, onde cada membro é o timestamp Unix em milissegundos da requisição, e o score é o próprio timestamp.

A operação de verificar e incrementar o contador é executada como script Lua para garantir atomicidade sem necessidade de transações otimistas (WATCH/MULTI/EXEC).

---

## Alternativas Consideradas

### Alternativa A — Fixed Window com INCR/EXPIRE

Cada usuário tem um contador inteiro. A chave expira no final da janela (ex: a cada hora no :00).

**Prós:**
- Simples de implementar (2 comandos Redis)
- Uso mínimo de memória (8 bytes por usuário)
- Sem necessidade de Lua

**Contras:**
- Permite burst no boundary: um usuário pode fazer 1.000 req nos últimos segundos de uma janela e 1.000 req nos primeiros da próxima, totalizando 2.000 req em poucos segundos
- Não foi escolhido porque o burst no boundary já causou problema em produção com outro sistema (ref: perguntas-respondidas.md P1)

### Alternativa B — Sliding Window Aproximada com dois contadores

Mantém dois contadores (janela atual e anterior) e usa interpolação linear para estimar a contagem na janela deslizante.

**Prós:**
- Uso de memória O(1) por usuário
- Sem necessidade de Lua complexo

**Contras:**
- Precisão apenas aproximada — pode ter erro de ±5% dependendo da distribuição de requisições
- Complexidade de implementação maior que parece
- A spec exige precisão de ±1% (RNF), o que esta abordagem não garante

### Alternativa C — Sorted Set com script Lua (escolhida)

**Prós:**
- Sliding window precisa (±0% de erro)
- Atomicidade garantida pelo Redis via Lua — sem race conditions
- O campo `reset_at` pode ser calculado exatamente a partir do membro mais antigo
- Testável independentemente (script Lua tem semântica clara)

**Contras:**
- Uso de memória O(N) onde N é o número de requisições na janela
- Para 1.000 req/h por usuário: ~50KB por usuário ativo
- Script Lua é menos familiar para desenvolvedores que não conhecem Redis profundamente

---

## Consequências

### Positivas
- Rate limiting é preciso — CA-6 (sem race condition) é implementável corretamente
- O timestamp de reset é exato — `Retry-After` é confiável para o cliente
- Comportamento determinístico e testável

### Negativas
- Uso de memória é maior que alternativas O(1)
- Com 10.000 usuários ativos fazendo 1.000 req/h: estimativa de 500MB de memória Redis
- Se o Redis não suportar Lua (Redis Cluster com restrições), a implementação precisará de ajuste
- Scripts Lua no Redis dificultam debugging em produção

### Riscos Aceitos
- O modelo de memória O(N) é aceitável dado o volume de usuários atual (< 10.000 ativos)
- Se o volume crescer 10x, revisitar esta decisão via novo ADR

---

## Notas de Implementação

O script Lua deve ser carregado via `SCRIPT LOAD` no startup da aplicação e chamado via `EVALSHA` para evitar transmitir o script a cada requisição. Isso também permite verificar se o script está carregado com `SCRIPT EXISTS`.

Referência ao script: `specs/EXAMPLE-001/plan.md` seção "Algoritmo Sliding Window com Redis"
