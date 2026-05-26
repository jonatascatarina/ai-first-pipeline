# Plano de Implementação — EXAMPLE-002

## Resumo Técnico

O fluxo OAuth2 é dividido entre frontend e backend. O frontend é responsável pela geração de PKCE e state, pelo redirecionamento ao GitHub e pela validação inicial do callback. O backend é responsável pela troca segura do código por token (usando o client_secret que nunca sai do servidor), pela busca do perfil no GitHub e pela emissão do JWT de sessão. Uma tabela `oauth_states` centraliza a validação de states no backend para defesa em profundidade.

## Abordagem

### Fluxo Principal

```
[Usuário clica "Entrar com GitHub"]
        |
        v
[Frontend] Gera code_verifier (random 64 bytes → base64url)
           Calcula code_challenge = base64url(SHA-256(code_verifier))
           Gera state = base64url(crypto.getRandomValues(32 bytes))
           Armazena code_verifier e state em sessionStorage
           Chama backend: POST /auth/github/init
        |
        v
[Backend] Persiste state em oauth_states (TTL 10 min)
          Retorna JSON: { authorization_url }
        |
        v
[Frontend] Redireciona browser para authorization_url do GitHub
        |
        v
[GitHub]  Usuário autoriza → GitHub redireciona para /auth/github/callback
          com params: code, state
        |
        v
[Frontend] Lê state do sessionStorage
           Compara com state recebido → se diferente, exibe erro e para
           Envia ao backend: POST /auth/github/callback
           body: { code, code_verifier, state }
        |
        v
[Backend] Valida state na tabela oauth_states (existe? não expirou? não foi usado?)
          Invalida state (marca como usado)
          Troca code por access_token com GitHub:
            POST https://github.com/login/oauth/access_token
            body: { client_id, client_secret, code, redirect_uri, code_verifier }
          GET https://api.github.com/user (com access_token)
          GET https://api.github.com/user/emails (se email ausente ou não verificado)
          Descarta access_token
          Upsert em users (github_id → cria ou atualiza)
          Vincula se email já existe sem github_id
          Emite JWT de sessão (7 dias)
          Retorna: { token, user: { id, name, email, avatar_url } }
        |
        v
[Frontend] Armazena JWT, redireciona para área autenticada
```

### Componentes Afetados

- **Tabela `users`** — adição de colunas `github_id`, `avatar_url`; tornar `password_hash` nullable (usuários OAuth2 não têm senha)
- **AuthController** — adição de dois endpoints: `POST /auth/github/init` e `POST /auth/github/callback`
- **Página de login** — adição do botão "Entrar com GitHub" e lógica de inicialização do fluxo
- **Callback route** — nova rota frontend `/auth/github/callback` que processa o retorno do GitHub

### Novos Componentes

- `GitHubOAuthService` — encapsula as chamadas à API do GitHub (token exchange, /user, /user/emails)
- `OAuthStateStore` — interface + implementação para persistir e validar states (tabela `oauth_states`)
- `PKCEHelper` — utilitário frontend para geração de code_verifier e code_challenge via Web Crypto API
- Migration `add_github_oauth_to_users` — adiciona colunas na tabela users e cria tabela oauth_states

### Modelo de Dados

Migration — tabela `users` (alteração):
```sql
ALTER TABLE users
  ADD COLUMN github_id VARCHAR(50) UNIQUE,
  ADD COLUMN avatar_url TEXT,
  ALTER COLUMN password_hash DROP NOT NULL;

CREATE UNIQUE INDEX idx_users_github_id ON users(github_id)
  WHERE github_id IS NOT NULL;
```

Migration — tabela `oauth_states` (nova):
```sql
CREATE TABLE oauth_states (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  state       VARCHAR(100) NOT NULL UNIQUE,
  used        BOOLEAN NOT NULL DEFAULT FALSE,
  created_at  TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  expires_at  TIMESTAMPTZ NOT NULL DEFAULT NOW() + INTERVAL '10 minutes'
);

CREATE INDEX idx_oauth_states_state ON oauth_states(state);
CREATE INDEX idx_oauth_states_expires_at ON oauth_states(expires_at);
```

Limpeza periódica de states expirados: job diário ou trigger ao inserir novos states.

### Contratos de Interface

**POST /auth/github/init**

Request:
```json
{ "state": "<base64url 32 bytes>", "code_challenge": "<base64url SHA-256>" }
```

Response 200:
```json
{
  "authorization_url": "https://github.com/login/oauth/authorize?client_id=...&state=...&code_challenge=...&code_challenge_method=S256&scope=read:user,user:email&redirect_uri=..."
}
```

**POST /auth/github/callback**

Request:
```json
{
  "code": "<authorization code do GitHub>",
  "code_verifier": "<original verifier>",
  "state": "<state retornado pelo GitHub>"
}
```

Response 200:
```json
{
  "token": "<JWT de sessão>",
  "user": {
    "id": "<uuid interno>",
    "name": "Nome do Usuário",
    "email": "usuario@example.com",
    "avatar_url": "https://avatars.githubusercontent.com/u/..."
  }
}
```

Response 400 (state inválido, code inválido, ou erro do GitHub):
```json
{ "error": "oauth_error", "message": "<mensagem legível>" }
```

## Dependências

### Dependências Técnicas

- Web Crypto API (nativa, sem biblioteca adicional no frontend)
- HTTP client no backend para chamar API do GitHub (já existente no projeto)
- Biblioteca JWT no backend para emitir tokens de sessão (já existente)
- PostgreSQL com suporte a UUID e TIMESTAMPTZ (já existente)

### Dependências de Features

- Sistema de autenticação por senha deve continuar funcionando em paralelo (não é removido)
- JWT middleware existente deve aceitar tokens gerados por este fluxo sem modificação

### Dependências de Dados

- Migration deve rodar antes do deploy do backend
- GitHub OAuth App deve estar criada com redirect_uri configurada para o ambiente de destino

## Riscos

### Riscos de Segurança

- **ALTO — client_secret exposto:** Se o `GITHUB_CLIENT_SECRET` vazar para logs ou for incluído no bundle frontend, qualquer pessoa pode impersonar a aplicação. Mitigação: revisão obrigatória de que a variável só existe em contexto de servidor; auditoria de logs antes do deploy.
- **ALTO — State previsível:** State gerado com `Math.random()` em vez de `crypto.getRandomValues()` é previsível e vulnerável a CSRF. Mitigação: `PKCEHelper` usa exclusivamente Web Crypto API; code review deve rejeitar qualquer uso de Math.random() neste contexto.
- **MÉDIO — Open redirect:** Se o frontend aceitar um `redirect_uri` dinâmico do query string, atacante pode redirecionar o code para outro servidor. Mitigação: `redirect_uri` é hardcoded no backend a partir de variável de ambiente; frontend não aceita redirect_uri de query params.
- **MÉDIO — Replay de authorization code:** Um code capturado no redirect pode ser reutilizado se o PKCE não for validado. Mitigação: PKCE obrigatório; o GitHub invalida o code após a primeira troca (comportamento padrão do provider).
- **BAIXO — Vinculação automática de conta por email:** O processo de vincular uma conta GitHub a uma conta existente por email poderia ser explorado se um atacante criasse uma conta GitHub com o email de outra pessoa. Mitigação: aceitar apenas emails verificados pelo GitHub (campo `verified: true` em `/user/emails`). Emails não verificados são rejeitados.

### Riscos de Performance

- **MÉDIO — Duas chamadas à API do GitHub em sequência:** `/user` + `/user/emails` adiciona dois round-trips à API do GitHub por login. Em condições normais, cada chamada leva 100-300ms. Mitigação: chamar `/user/emails` apenas se o email do perfil público estiver ausente ou não verificado — caso mais comum resolve com uma única chamada.
- **BAIXO — Limpeza de states expirados:** A tabela `oauth_states` pode crescer indefinidamente sem limpeza. Mitigação: job de limpeza diário com `DELETE FROM oauth_states WHERE expires_at < NOW()`.

### Riscos de Disponibilidade

- **MÉDIO — Dependência do GitHub:** Se o GitHub estiver fora do ar, login OAuth2 fica indisponível. Usuários com senha legada ainda conseguem acessar. Mitigação: exibir mensagem clara de "GitHub indisponível, use seu e-mail e senha" quando a troca de token falhar com timeout.
- **BAIXO — Rate limit da API do GitHub:** 5.000 requests/hora para tokens autenticados. Com o access_token do OAuth App (não do usuário), o limite é compartilhado por todos os logins. Em picos extremos de login, pode atingir o limite. Mitigação: fora do escopo desta versão; monitorar após lançamento.

### Riscos de Manutenibilidade

- **BAIXO — Mudanças na API do GitHub:** A API do GitHub muda com pouca frequência, mas mudanças de campos em `/user` ou `/user/emails` podem quebrar o parsing. Mitigação: `GitHubOAuthService` encapsula todo o parsing — mudanças ficam em um único lugar.

## Estratégia de Testes

### Testes Unitários

- `PKCEHelper`: code_verifier tem entre 43-128 chars, code_challenge é SHA-256 correto, geração usa crypto.getRandomValues
- `GitHubOAuthService`: parsing correto de `/user` com email público, parsing correto de `/user` sem email (fallback a `/user/emails`), rejeição de email não verificado
- `OAuthStateStore`: state é salvo, state é invalidado após uso, state expirado é rejeitado, state inexistente é rejeitado
- Endpoint `/auth/github/callback`: retorna 400 para state inválido, retorna 400 para code inválido, retorna 400 para email não verificado, retorna 200 com JWT para fluxo feliz

### Testes de Integração

- Fluxo completo com mock do GitHub: novo usuário → registro criado com campos corretos
- Fluxo completo com mock do GitHub: usuário existente (mesmo github_id) → atualiza nome e avatar, não duplica
- Fluxo completo: usuário com mesmo email mas sem github_id → vinculação automática
- Replay de state: segunda chamada com mesmo state retorna 400
- GitHub retorna erro (access_denied): backend retorna 400 com mensagem legível

### Testes de Segurança

- Confirmar que `client_secret` não aparece em nenhuma resposta HTTP (headers, body, logs)
- Confirmar que `access_token` do GitHub não é persistido no banco após o fluxo
- Confirmar que state expirado (> 10 min) é rejeitado

## Decisões Técnicas

- **Decisão:** Backend persiste o state (tabela `oauth_states`) além do sessionStorage do frontend
  - Alternativas: apenas sessionStorage no frontend
  - Justificativa: defesa em profundidade; sessionStorage pode ser manipulado em extensões maliciosas ou XSS; validação no backend é a última linha de defesa
  - Consequências: migration necessária; limpeza periódica de states expirados

- **Decisão:** Descartar access_token do GitHub após o fluxo
  - Alternativas: armazenar token cifrado para uso futuro
  - Justificativa: minimizar superfície de ataque; produto não precisa de integração GitHub além do login; se precisar no futuro, nova feature com scope adequado
  - Consequências: se o produto precisar de integração GitHub, um novo fluxo de re-authorization será necessário

- **Decisão:** Vincular automaticamente contas com mesmo email verificado
  - Alternativas: rejeitar e pedir ao usuário que faça login com senha e vincule manualmente
  - Justificativa: UX melhor; o email verificado pelo GitHub é confiável o suficiente como identificador
  - Consequências: risco residual de account takeover se o GitHub não verificar o email corretamente (mitigado exigindo `verified: true`)

- **Decisão:** `POST /auth/github/init` em vez de montar a URL no frontend
  - Alternativas: frontend monta authorization_url diretamente com VITE_GITHUB_CLIENT_ID
  - Justificativa: centralizar a lógica de construção da URL no backend facilita auditoria, permite mudar parâmetros sem rebuild do frontend e persiste o state antes do redirect
  - Consequências: um round-trip extra antes do redirect; aceitável (< 100ms)

## Estimativa de Complexidade

| Area | Complexidade | Justificativa |
|------|-------------|---------------|
| Backend (endpoints + service) | M | Dois endpoints + integração com API GitHub + upsert com vinculação |
| Frontend (PKCE + callback) | P | Web Crypto API é nativa; lógica de fluxo é linear |
| Migration | P | Duas alterações simples + nova tabela |
| Testes | M | Casos de segurança exigem mocks cuidadosos da API GitHub |
