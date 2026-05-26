# Perguntas Respondidas — EXAMPLE-002

## Rodada 1 — 2026-05-26

### P1 — PKCE é obrigatório mesmo com client_secret?

**Contexto:** PKCE foi criado para aplicações públicas (SPAs, apps mobile) que não conseguem guardar um client_secret com segurança. No caso de uma aplicação web com backend, o client_secret já provê proteção. O RFC 9700 recomenda PKCE em todos os casos, mas nem todos os provedores o suportam com client_secret simultaneamente.

**Resposta:** Sim, PKCE é obrigatório mesmo com client_secret. O GitHub suporta ambos simultaneamente. PKCE adiciona uma camada extra de proteção: mesmo que um atacante capture o `authorization_code` no redirect, ele não consegue trocá-lo por token sem o `code_verifier` correspondente. Esta é a melhor prática atual e o GitHub a documenta.

**Impacto na spec:** RF-1 ao RF-5 especificam PKCE como obrigatório. Sem negociação.

---

### P2 — O state deve ser validado apenas no frontend ou também no backend?

**Contexto:** Validar o `state` apenas no frontend é suficiente se o frontend for confiável. Mas um CSRF mais sofisticado poderia contornar validações client-side. Validar no backend duplica esforço mas adiciona defesa em profundidade.

**Resposta:** Validação em ambos. O frontend valida antes de chamar o backend (CA-3). O backend também valida consultando a tabela `oauth_states` (CA-9). A tabela registra states emitidos e os invalida após o primeiro uso. TTL de 10 minutos — states mais antigos são rejeitados automaticamente.

**Impacto na spec:** RF-11 e CA-9 especificam validação no backend. Dependência em tabela `oauth_states` adicionada.

---

### P3 — O access token do GitHub deve ser armazenado para uso futuro (ex: criar issues, ler repos)?

**Contexto:** Se o produto precisar fazer ações no GitHub em nome do usuário após o login (ex: criar PRs, comentar em issues), o access token precisaria ser armazenado. Isso muda significativamente a arquitetura de segurança.

**Resposta:** Não. O access token do GitHub é usado apenas para buscar o perfil durante o login e descartado em seguida. Se no futuro o produto precisar de integração mais profunda com o GitHub, isso será uma nova feature com nova spec. CA-8 documenta explicitamente que o token não é persistido.

**Impacto na spec:** RNF de Segurança e CA-8 explicitam descarte do token. Não-escopo atualizado para mencionar isso.

---

### P4 — O que acontece se o usuário autoriza no GitHub mas volta com e-mail já cadastrado por outro método?

**Contexto:** O sistema tem autenticação por senha legada. Um usuário que se cadastrou com e-mail + senha pode tentar fazer login com GitHub usando o mesmo e-mail. Sem tratamento, isso cria dois registros com o mesmo e-mail.

**Resposta:** Vincular automaticamente se o e-mail for o mesmo. Se o backend encontrar um usuário existente com o mesmo e-mail mas sem `github_id`, deve preencher o `github_id` no registro existente e retornar JWT normalmente. O usuário passa a poder usar ambos os métodos. Isso deve ser logado para auditoria.

**Impacto na spec:** RF-9 atualizado: "cria um registro se `github_id` não existir e nenhum usuário tiver o mesmo e-mail; vincula se o e-mail já existir; atualiza nome e avatar se `github_id` já existir".

---

### P5 — Como armazenar o state? SessionStorage, cookie ou banco de dados?

**Contexto:** SessionStorage no frontend é simples mas perdido ao fechar a aba. Cookie httpOnly é mais seguro mas exige configuração de domínio. Banco de dados é o mais robusto mas adiciona latência.

**Resposta:** Duplo armazenamento: sessionStorage no frontend (para validação imediata no callback) e tabela `oauth_states` no backend (para defesa em profundidade e invalidação após uso). O TTL da tabela é 10 minutos — suficiente para completar o fluxo OAuth2 em condições normais e anormais.

**Impacto na spec:** Dependência em tabela `oauth_states` adicionada. P2 desta lista detalha o comportamento.

---

### P6 — O JWT de sessão emitido deve ter qual duração?

**Contexto:** JWTs de longa duração são convenientes mas aumentam a janela de abuso em caso de vazamento. JWTs de curta duração com refresh token são mais seguros mas adicionam complexidade.

**Resposta:** JWT de 7 dias, sem refresh token por ora. O sistema legado de senha já usa JWT de 7 dias — manter consistência. Se o usuário precisar de mais segurança, pode revogar manualmente no GitHub. Adicionar refresh token é escopo de uma feature futura de segurança de sessão.

**Impacto na spec:** RF-10 especifica emissão do JWT. Duração de 7 dias documentada aqui, não na spec (decisão de implementação, não de produto).

---

### P7 — Qual scope solicitar ao GitHub?

**Contexto:** `read:user` dá acesso ao perfil público. `user:email` é necessário para ler e-mails verificados (incluindo e-mails privados). Solicitar mais permissões que o necessário viola o princípio do mínimo privilégio e pode assustar usuários na tela de autorização do GitHub.

**Resposta:** Apenas `read:user` e `user:email`. Estes dois scopes são suficientes para obter nome, avatar e e-mail verificado. Nenhum outro scope deve ser solicitado nesta feature. Se uma integração mais profunda for necessária no futuro, nova spec deve tratar do re-authorization com scopes adicionais.

**Impacto na spec:** RF-3 especifica `scope=read:user,user:email` explicitamente.

---

### P8 — Como tratar o caso de usuário que revogou o acesso no GitHub e tenta fazer login novamente?

**Contexto:** Se o usuário revogou a autorização no github.com/settings/applications, o próximo login OAuth2 solicitará autorização novamente (comportamento padrão do GitHub). Não há ação especial necessária no produto, pois o fluxo recomeça do zero.

**Resposta:** Nenhum tratamento especial. O GitHub apresenta a tela de autorização novamente. O produto trata como um login normal. Nenhum estado do lado do produto precisa ser limpo — o `access_token` não é persistido. Comportamento correto por padrão.

**Impacto na spec:** Nenhum. Confirmação de que não há caso especial a tratar.
