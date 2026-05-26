# EXAMPLE-002 — Autenticação OAuth2 com GitHub (Authorization Code Flow + PKCE)

## Contexto

A aplicação web atualmente exige que usuários criem e gerenciem uma senha própria. Isso gera fricção no onboarding, aumenta a taxa de abandono no cadastro e coloca a responsabilidade de armazenar credenciais no produto. A maioria dos usuários-alvo já tem uma conta no GitHub e prefere autenticar com ela.

## Problema

O sistema de autenticação atual tem três problemas estruturais:
- **Fricção de cadastro:** Formulário de registro com e-mail, senha e confirmação de senha afasta 40% dos visitantes antes de completar o cadastro (dado do funil de analytics)
- **Responsabilidade de credenciais:** O produto armazena hashes de senha, o que implica risco de vazamento e responsabilidade de compliance
- **Recuperação de senha:** Fluxo de "esqueci minha senha" é a segunda fonte de tickets de suporte, atrás apenas de dúvidas de uso

## Solução Proposta

Implementar autenticação via GitHub OAuth2 usando o **Authorization Code Flow com PKCE** (Proof Key for Code Exchange). O PKCE protege o fluxo contra interceptação do authorization code, sendo obrigatório para aplicações públicas mesmo com client_secret. O usuário clica em "Entrar com GitHub", é redirecionado ao GitHub para autorizar, volta à aplicação autenticado. Nenhuma senha é armazenada no produto.

## Atores

- **Usuário** — clica em "Entrar com GitHub" e autoriza a aplicação no GitHub
- **Browser** — armazena o code_verifier do PKCE e o state anti-CSRF em sessionStorage
- **Frontend (SPA)** — inicia o fluxo, gera PKCE, redireciona ao GitHub, processa o callback
- **Backend API** — troca o authorization code por access token, busca perfil no GitHub, cria/atualiza registro de usuário, emite JWT de sessão
- **GitHub OAuth App** — provedor de identidade externo (github.com/settings/developers)
- **Banco de dados** — armazena registro de usuário vinculado ao GitHub ID

## Requisitos Funcionais

1. O frontend deve gerar um `code_verifier` aleatório de 43-128 caracteres e calcular o `code_challenge` como SHA-256 do verifier codificado em base64url
2. O frontend deve gerar um `state` aleatório de 32 bytes e armazená-lo em sessionStorage antes do redirecionamento
3. O redirecionamento ao GitHub deve incluir os parâmetros: `client_id`, `redirect_uri`, `scope=read:user,user:email`, `state`, `code_challenge`, `code_challenge_method=S256`
4. Ao receber o callback, o frontend deve verificar que o `state` retornado corresponde ao armazenado em sessionStorage antes de prosseguir
5. O frontend deve enviar ao backend: `code`, `code_verifier` e `redirect_uri`
6. O backend deve trocar o `code` por `access_token` junto à API do GitHub, enviando `client_id`, `client_secret`, `code`, `redirect_uri` e `code_verifier`
7. O backend deve buscar o perfil do usuário na API do GitHub (`GET /user`) usando o `access_token` obtido
8. O backend deve buscar o e-mail primário verificado do usuário na API do GitHub (`GET /user/emails`) caso o e-mail não esteja no perfil público
9. O backend deve criar um registro de usuário se o `github_id` não existir no banco, ou atualizar nome e avatar se já existir
10. O backend deve emitir um JWT de sessão assinado com a identidade do usuário e retorná-lo ao frontend
11. O backend deve recusar o callback se o `state` não corresponder ao esperado, retornando `400 Bad Request`
12. O backend deve recusar o callback se o GitHub retornar `error` no parâmetro de callback, retornando `400 Bad Request` com a mensagem do erro

## Requisitos Não-Funcionais

- **Segurança:** O `client_secret` do GitHub OAuth App nunca deve aparecer em código frontend, logs ou variáveis de ambiente expostas ao browser
- **Segurança:** O `code_verifier` nunca deve ser enviado ao GitHub diretamente — apenas o `code_challenge`
- **Segurança:** O `state` deve ser invalidado após o primeiro uso (não reutilizável)
- **Segurança:** O `access_token` do GitHub não deve ser armazenado no banco após a troca — apenas o `github_id` e dados de perfil
- **Performance:** O fluxo completo (redirect → callback → JWT emitido) deve ser concluído em menos de 3 segundos em condições normais de rede
- **UX:** Em caso de erro no callback, o usuário deve ver mensagem legível e link para tentar novamente — não stack trace
- **Privacidade:** Apenas `github_id`, nome de exibição, e-mail primário e URL do avatar são armazenados — nenhum outro dado do perfil GitHub

## Critérios de Aceite

1. Dado um usuário que nunca usou a aplicação, ao concluir o fluxo OAuth2 pela primeira vez, um novo registro é criado na tabela `users` com `github_id`, `name`, `email` e `avatar_url` corretos
2. Dado um usuário que já tem registro, ao concluir o fluxo OAuth2 novamente, o registro existente é atualizado (nome e avatar) mas nenhum registro duplicado é criado
3. Dado que o `state` no callback não corresponde ao armazenado em sessionStorage, o frontend aborta o fluxo e exibe mensagem de erro sem chamar o backend
4. Dado que o backend recebe um `code` já utilizado (replay), o GitHub retorna erro e o backend retorna `400 Bad Request`
5. Dado que o `code_verifier` enviado ao backend não corresponde ao `code_challenge` enviado ao GitHub, a troca de token falha e o backend retorna `400 Bad Request`
6. Após autenticação bem-sucedida, o frontend recebe um JWT válido e o usuário é redirecionado para a página autenticada
7. Dado que o usuário nega a autorização no GitHub, o callback retorna `error=access_denied` e o usuário vê mensagem "Autorização negada. Tente novamente." com botão de retry
8. O `access_token` do GitHub não está presente em nenhuma tabela do banco após a conclusão do fluxo
9. Dado um `state` reutilizado (segunda chamada com o mesmo state), o backend retorna `400 Bad Request`

## Não-Escopo

- Login com outros provedores OAuth2 (Google, Facebook, etc.) — esta spec é exclusiva do GitHub
- Autenticação por senha (o sistema legado de senha é mantido em paralelo, não removido nesta feature)
- Refresh automático do access token do GitHub — o token é usado apenas durante o fluxo de login
- Revogação de autorização via interface da aplicação (o usuário revoga diretamente no GitHub)
- Criação de conta sem OAuth2 (cadastro por e-mail)
- Single Sign-On (SSO) corporativo
- Vinculação de múltiplos provedores ao mesmo usuário

## Dependências

- GitHub OAuth App criada em github.com/settings/developers com `client_id` e `client_secret` configurados
- Variáveis de ambiente no backend: `GITHUB_CLIENT_ID`, `GITHUB_CLIENT_SECRET`, `GITHUB_REDIRECT_URI`
- Variável de ambiente no frontend (build-time): `VITE_GITHUB_CLIENT_ID`
- Tabela `users` com colunas: `id`, `github_id`, `name`, `email`, `avatar_url`, `created_at`, `updated_at`
- Tabela `oauth_states` para armazenar states válidos com TTL (ou alternativa via Redis)
- Biblioteca de criptografia disponível no frontend (Web Crypto API — nativa nos browsers modernos)
- Biblioteca JWT no backend para emitir tokens de sessão (já existente no projeto)

## Riscos Conhecidos

- **ALTO — Vazamento do client_secret:** Se o `client_secret` aparecer no bundle frontend, qualquer pessoa pode impersonar a aplicação. Mitigação: client_secret fica apenas no backend; frontend usa apenas `client_id`
- **MÉDIO — Fixação de estado (state fixation):** Se o `state` for previsível ou fixo, um atacante pode forjar o callback. Mitigação: geração criptograficamente segura com `crypto.getRandomValues`
- **MÉDIO — Open redirect no callback:** Se o `redirect_uri` não for validado, um atacante pode redirecionar o code para seu servidor. Mitigação: `redirect_uri` é fixo no GitHub OAuth App e validado pelo backend
- **BAIXO — Rate limit da API do GitHub:** O GitHub limita chamadas à API. Em picos de login, múltiplas chamadas `/user` e `/user/emails` podem ser limitadas. Mitigação: fora do escopo desta versão; adicionar cache se necessário
- **BAIXO — Conta GitHub sem e-mail público:** Usuários com e-mail privado no GitHub exigem chamada extra a `/user/emails`. Já previsto no RF-8

## Perguntas em Aberto

Nenhuma — todas as questões foram respondidas em `perguntas-respondidas.md`.
