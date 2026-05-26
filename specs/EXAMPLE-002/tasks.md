# Tasks — EXAMPLE-002

## SEC-001 — Auditoria de variáveis de ambiente antes do deploy

**O que fazer:** Verificar que `GITHUB_CLIENT_SECRET` está configurado apenas como variável de ambiente de servidor (não exposto no bundle frontend, não em arquivos `.env` commitados, não em logs de CI). Revisar configuração de CI/CD e `.gitignore`.

**Critério de conclusão:** Auditoria documentada confirmando que o secret não aparece em nenhum artefato público. Qualquer finding corrigido antes de avançar para T1.

**Testes esperados:** `grep -r "GITHUB_CLIENT_SECRET" dist/` não retorna resultado. CI logs não contêm o valor da variável.

---

## T1 — Migration: adicionar colunas OAuth2 em `users` e criar tabela `oauth_states`

**O que fazer:** Criar migration com as duas alterações definidas em `plan.md`:
1. `ALTER TABLE users`: adicionar `github_id VARCHAR(50) UNIQUE`, `avatar_url TEXT`; tornar `password_hash` nullable
2. `CREATE TABLE oauth_states`: `id`, `state`, `used`, `created_at`, `expires_at`
3. Criar índices definidos no plano

**Critério de conclusão:** Migration executa sem erro em ambiente de desenvolvimento. Rollback também funciona. Schema resultante confere com o definido em `plan.md`.

**Testes esperados:** `migrate up` e `migrate down` executam sem erro. Consulta de schema confirma colunas e índices.

---

## T2 — Backend: implementar `OAuthStateStore`

**O que fazer:** Criar serviço que encapsula operações na tabela `oauth_states`:
- `saveState(state: string): Promise<void>` — insere state com TTL de 10 minutos
- `validateAndConsume(state: string): Promise<boolean>` — verifica existência, não-expiração e não-uso; marca como `used = true` atomicamente; retorna false se qualquer condição falhar

**Critério de conclusão:** Testes unitários passam para todos os casos: state válido, state expirado, state já usado, state inexistente.

**Testes esperados:**
- `saveState` + `validateAndConsume` imediato → retorna true
- `saveState` + `validateAndConsume` duas vezes → segunda retorna false
- State inserido com `expires_at` no passado → `validateAndConsume` retorna false

---

## T3 — Backend: implementar `GitHubOAuthService`

**O que fazer:** Criar serviço com dois métodos:
- `exchangeCode(code, codeVerifier, redirectUri): Promise<string>` — chama `POST https://github.com/login/oauth/access_token` e retorna o access_token
- `getUserProfile(accessToken): Promise<GitHubProfile>` — chama `GET /user`; se email ausente ou não verificado, chama `GET /user/emails` e seleciona o e-mail primário verificado; lança erro se nenhum e-mail verificado for encontrado

**Critério de conclusão:** Testes com mocks HTTP para: perfil com email público verificado (1 chamada), perfil sem email público (2 chamadas, retorna email primário verificado), perfil sem nenhum email verificado (lança erro), code inválido (lança erro com mensagem legível).

**Testes esperados:** Os 4 cenários acima cobertos por testes unitários com mock HTTP.

---

## T4 — Backend: implementar endpoint `POST /auth/github/init`

**O que fazer:** Endpoint que recebe `{ state, code_challenge }`, persiste o state via `OAuthStateStore`, e retorna `{ authorization_url }` com a URL completa do GitHub incluindo todos os parâmetros definidos em `plan.md` (client_id, redirect_uri, scope, state, code_challenge, code_challenge_method=S256).

**Critério de conclusão:** Chamada ao endpoint retorna 200 com URL válida contendo todos os parâmetros obrigatórios. Chamada com state duplicado retorna 400.

**Testes esperados:** URL gerada contém `code_challenge_method=S256`, `scope=read:user,user:email`, `redirect_uri` correto da variável de ambiente.

---

## T5 — Backend: implementar endpoint `POST /auth/github/callback`

**O que fazer:** Endpoint que recebe `{ code, code_verifier, state }` e executa o fluxo completo:
1. Valida state via `OAuthStateStore.validateAndConsume`
2. Troca code por access_token via `GitHubOAuthService.exchangeCode`
3. Busca perfil via `GitHubOAuthService.getUserProfile`
4. Faz upsert no banco (lógica de vinculação por email definida em P4 das perguntas)
5. Emite JWT de sessão (7 dias)
6. Retorna `{ token, user }`

Retorna 400 com `{ error, message }` em qualquer falha (state inválido, code inválido, email não verificado, erro da API GitHub).

**Critério de conclusão:** Todos os critérios de aceite CA-1 ao CA-9 da spec passam em testes de integração.

**Testes esperados:**
- Novo usuário → registro criado, JWT retornado
- Usuário existente (mesmo github_id) → atualizado, sem duplicata
- Usuário existente (mesmo email, sem github_id) → vinculado, JWT retornado
- State inválido → 400
- Code inválido → 400
- Replay de state → 400

---

## T6 — Frontend: implementar `PKCEHelper`

**O que fazer:** Módulo utilitário com duas funções usando exclusivamente Web Crypto API:
- `generateCodeVerifier(): string` — gera 64 bytes aleatórios e codifica em base64url (resultado entre 43-128 chars)
- `generateCodeChallenge(verifier: string): Promise<string>` — calcula SHA-256 do verifier e codifica em base64url

**Critério de conclusão:** Testes unitários confirmam: verifier tem comprimento correto, challenge é SHA-256 correto do verifier, nenhuma chamada a `Math.random()`.

**Testes esperados:** Par verifier/challenge gerado é aceito por biblioteca de validação PKCE de referência.

---

## T7 — Frontend: implementar botão e lógica de inicialização do fluxo

**O que fazer:** Na página de login, adicionar botão "Entrar com GitHub". Ao clicar:
1. Gerar code_verifier e code_challenge via `PKCEHelper`
2. Gerar state via `crypto.getRandomValues`
3. Armazenar code_verifier e state em sessionStorage
4. Chamar `POST /auth/github/init` com state e code_challenge
5. Redirecionar para authorization_url retornada

**Critério de conclusão:** Clicar no botão redireciona para GitHub com todos os parâmetros corretos. sessionStorage contém code_verifier e state após o clique.

**Testes esperados:** Teste de componente verifica que sessionStorage é populado e que `window.location.href` recebe a URL correta.

---

## T8 — Frontend: implementar callback route `/auth/github/callback`

**O que fazer:** Rota que processa o retorno do GitHub:
1. Ler `code` e `state` dos query params da URL
2. Verificar se há `error` nos query params → exibir mensagem de erro com botão de retry
3. Ler state armazenado em sessionStorage → se diferente do retornado, exibir erro de segurança e parar
4. Ler code_verifier do sessionStorage
5. Chamar `POST /auth/github/callback` com code, code_verifier, state
6. Ao receber JWT, armazenar e redirecionar para área autenticada
7. Em caso de erro do backend, exibir mensagem legível com botão de retry

**Critério de conclusão:** CA-3 (state mismatch exibido no frontend), CA-6 (redirecionamento após sucesso) e CA-7 (mensagem de erro para access_denied) passam em testes de componente.

**Testes esperados:**
- State mismatch → mensagem de erro, sem chamada ao backend
- `error=access_denied` → "Autorização negada. Tente novamente."
- Fluxo feliz → JWT armazenado, redirect para área autenticada

---

## T9 — Job de limpeza de states expirados

**O que fazer:** Implementar rotina que executa `DELETE FROM oauth_states WHERE expires_at < NOW()`. Pode ser um cron job diário ou uma função invocada ao inserir novos states. Documentar a escolha e configurar para o ambiente de produção.

**Critério de conclusão:** States expirados são removidos automaticamente. Tabela não cresce indefinidamente. Escolha de abordagem documentada.

**Testes esperados:** Após execução da limpeza, registros com `expires_at` no passado não existem mais na tabela.

---

## T10 — Documentar configuração da GitHub OAuth App

**O que fazer:** Adicionar seção em `docs/` (ou no README do projeto) explicando como criar e configurar a GitHub OAuth App para desenvolvimento e produção: URL de autorização, callback URL, variáveis de ambiente necessárias, e como testar localmente com `localhost`.

**Critério de conclusão:** Um desenvolvedor consegue configurar o ambiente do zero seguindo apenas a documentação, sem precisar perguntar para o time.

**Testes esperados:** Review por um desenvolvedor que não conhece o projeto confirma que as instruções são suficientes.
