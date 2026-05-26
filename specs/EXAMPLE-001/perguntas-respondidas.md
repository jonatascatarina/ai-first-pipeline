# Perguntas Respondidas — EXAMPLE-001

## Rodada 1 — 2026-05-26

### P1 — O algoritmo de rate limiting deve ser fixed window ou sliding window?

**Contexto:** Fixed window é mais simples mas tem o problema do "burst no boundary" — um usuário pode fazer 1.000 req nos últimos segundos de uma janela e 1.000 req nos primeiros da próxima, efetivamente fazendo 2.000 req em poucos segundos. Sliding window evita isso mas é mais complexa.

**Resposta:** Sliding window. O caso de burst no boundary já aconteceu em produção com o sistema atual de logging (que usa fixed window) e causou problemas. Vamos direto para sliding window.

**Impacto na spec:** Seção "Solução Proposta" atualizada para especificar sliding window explicitamente. RF-1 mantido.

---

### P2 — O que acontece com o limite quando um usuário muda de plano no meio da janela?

**Contexto:** Um usuário `free` que atinge 900/1.000 requisições e faz upgrade para `pro` — ele passa a ter 5.000 ou o contador reseta?

**Resposta:** O novo limite se aplica imediatamente, o contador não reseta. O usuário `free` com 900 req passa a ter `X-RateLimit-Remaining: 4100` (5000 - 900).

**Impacto na spec:** Adicionado em RF implícito — o middleware deve consultar o plano atual do usuário a cada requisição (ou com cache de 60s).

---

### P3 — Requisições de serviços internos (machine-to-machine) também sofrem rate limiting?

**Contexto:** Se um serviço interno usar um token de serviço para chamar a API, ele entra no mesmo bucket?

**Resposta:** Não. Tokens de serviço têm o campo `type: service` no JWT payload. O middleware deve verificar esse campo e skip o rate limiting para service tokens.

**Impacto na spec:** Adicionado como exceção no RF-1: "exceto tokens com `type: service` no payload JWT".

---

### P4 — Qual deve ser o comportamento quando o Redis demora (timeout), não quando está completamente offline?

**Contexto:** A spec menciona Redis offline (fail-open), mas um Redis lento que demora 500ms é diferente de um Redis offline.

**Resposta:** Timeout de 50ms. Se a operação Redis demorar mais que 50ms, tratar como falha e aplicar fail-open. O timeout deve ser configurável via variável de ambiente `REDIS_RATE_LIMIT_TIMEOUT_MS`.

**Impacto na spec:** RNF de Disponibilidade atualizado para incluir timeout de 50ms.

---

### P5 — A janela de 1 hora é deslizante contínua ou reinicia em horários fixos (ex: :00 de cada hora)?

**Contexto:** "1 hora deslizante" significa que se você fez a primeira req às 14h27, o limite reseta às 15h27. "Hora fixa" significa que reseta às 15h00 para todos.

**Resposta:** Janela deslizante de 60 minutos a partir da primeira requisição. O campo `X-RateLimit-Reset` deve refletir quando a requisição mais antiga na janela atual vai expirar.

**Impacto na spec:** CA-7 atualizado para refletir janela deslizante (não horário fixo).

---

### P6 — Como identificar o IP para requisições não autenticadas? O serviço fica atrás de load balancer?

**Contexto:** Se há load balancer ou proxy reverso, o IP direto é o do proxy, não do cliente real.

**Resposta:** Sim, há Nginx como proxy reverso. Usar o header `X-Forwarded-For` (primeiro IP da lista). O header é confiável porque o Nginx é configurado para sobrescrevê-lo.

**Impacto na spec:** RF-8 atualizado para especificar uso de `X-Forwarded-For`.

---

### P7 — O limite de 100 req/hora para IPs não autenticados é por IP ou compartilhado?

**Contexto:** IPs em redes corporativas ou behind NAT podem ter centenas de usuários no mesmo IP público.

**Resposta:** É por IP, ciente do risco de NAT. Se for problema no futuro, resolvemos com autenticação obrigatória. Por ora, 100 req/hora por IP é aceitável — usuários que precisam de mais devem se autenticar.

**Impacto na spec:** RF-8 mantido como está.

---

### P8 — Onde ficam armazenados os limites por plano? Hardcoded, banco de dados ou configuração?

**Contexto:** Se os limites mudarem com frequência, hardcoded é um problema. Se raramente, é mais simples.

**Resposta:** Variável de ambiente por enquanto. Limites mudam raramente (última mudança foi há 2 anos). Se precisarmos de dinamismo, faremos ADR para adicionar tabela de configuração. Variáveis: `RATE_LIMIT_FREE`, `RATE_LIMIT_PRO`, `RATE_LIMIT_ENTERPRISE`.

**Impacto na spec:** Dependência na tabela `users.plan` mantida, mas os limites em si vêm de env vars, não de uma tabela de limites.
