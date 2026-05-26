# EXAMPLE-001 — Rate Limiting por Usuário (API REST)

## Contexto

A API REST do produto está exposta publicamente e não possui nenhum mecanismo de controle de taxa. Clientes abusivos ou comprometidos podem fazer milhares de requisições por segundo, degradando a experiência de todos os usuários e aumentando custos de infraestrutura.

## Problema

Sem rate limiting, qualquer cliente autenticado pode fazer requisições ilimitadas. Em incidentes recentes:
- Um cliente com bug de retry saturou o endpoint `/search` com 50.000 req/min durante 8 minutos
- O P99 de latência subiu de 120ms para 4.200ms para todos os usuários durante o incidente
- O custo de infraestrutura daquele dia foi 340% acima da média

## Solução Proposta

Implementar rate limiting por usuário autenticado usando o algoritmo **sliding window** com armazenamento no Redis. Cada usuário terá um bucket de requisições com limite configurável por plano de assinatura. Requisições além do limite recebem resposta `429 Too Many Requests` com header informando quando o limite será resetado.

## Atores

- **Usuário da API** — consome endpoints REST com token JWT
- **Sistema de Rate Limiting** — intercepta requisições antes de chegarem ao handler
- **Redis** — armazena contadores de requisições com TTL
- **Operações** — configura limites por plano via variável de ambiente ou tabela de configuração

## Requisitos Funcionais

1. O sistema deve contar requisições por usuário autenticado (identificado pelo `user_id` no JWT)
2. O limite padrão é de 1.000 requisições por hora por usuário
3. Usuários com plano `pro` têm limite de 5.000 requisições por hora
4. Usuários com plano `enterprise` têm limite de 50.000 requisições por hora
5. Quando o limite é atingido, a resposta deve ser `429 Too Many Requests`
6. Toda resposta da API deve incluir os headers:
   - `X-RateLimit-Limit` — limite total do usuário
   - `X-RateLimit-Remaining` — requisições restantes na janela atual
   - `X-RateLimit-Reset` — Unix timestamp de quando a janela reseta
7. O header `Retry-After` deve estar presente em respostas `429`, com o número de segundos até o reset
8. Requisições não autenticadas têm limite separado de 100 requisições por hora por IP
9. O rate limiting deve ser aplicado antes da lógica de negócio — não deve chegar ao banco de dados

## Requisitos Não-Funcionais

- **Latência:** O middleware de rate limiting deve adicionar no máximo 5ms de overhead ao P99
- **Disponibilidade:** Se o Redis estiver indisponível, o sistema deve operar em modo degradado (fail-open) — requisições passam sem verificação de limite, com log de alerta
- **Precisão:** A janela sliding window deve ter precisão de ±1% na contagem
- **Escalabilidade:** Deve suportar 10.000 usuários simultâneos sem degradação
- **Atomicidade:** A operação de verificar e incrementar o contador deve ser atômica para evitar race conditions

## Critérios de Aceite

1. Dado um usuário do plano `free` fazendo 1.001 requisições em uma hora, a requisição 1.001 retorna `429` e as anteriores retornam sucesso
2. Dado um usuário do plano `pro` fazendo 5.001 requisições em uma hora, a requisição 5.001 retorna `429`
3. Toda resposta bem-sucedida contém os três headers `X-RateLimit-*` com valores corretos
4. Uma resposta `429` contém `X-RateLimit-*` e `Retry-After` com valor entre 1 e 3600
5. Dado que o Redis está offline, todas as requisições retornam normalmente (sem erro 429) e um log de nível `WARN` é emitido para cada requisição
6. Dado dois requests simultâneos do mesmo usuário com 1 requisição restante no limite, no máximo um deles passa e o outro recebe `429` (sem race condition)
7. Um usuário que atingiu o limite às 14h30 pode fazer novas requisições a partir de 15h30 (janela de 1 hora)
8. Requisições não autenticadas de um mesmo IP recebem `429` após 100 requisições em uma hora

## Não-Escopo

- Limites por endpoint específico (todos os endpoints compartilham o mesmo bucket)
- Rate limiting por IP para usuários autenticados
- Interface administrativa para visualizar uso de rate limit
- Limite diferente por método HTTP (GET vs POST)
- Whitelist de usuários ou IPs via API — apenas via configuração de ambiente
- Notificação proativa ao usuário quando está se aproximando do limite

## Dependências

- Redis 6.0 ou superior (para suporte a comandos Lua atômicos)
- Middleware de autenticação JWT deve estar implementado e o `user_id` disponível no contexto da requisição
- Tabela `users` com coluna `plan` (valores: `free`, `pro`, `enterprise`)

## Riscos Conhecidos

- **Race condition na sliding window:** Implementações ingênuas com INCR/EXPIRE separados são não-atômicas. Mitigação: usar script Lua ou RedisTransaction
- **Memória Redis:** 10.000 usuários ativos × sliding window = estimativa de 10MB de memória no Redis
- **Fail-open:** Modo degradado sem Redis permite abuso temporário. Aceito pelo negócio — a alternativa (fail-closed) causaria downtime

## Perguntas em Aberto

Nenhuma — todas as questões foram respondidas em `perguntas-respondidas.md`.
