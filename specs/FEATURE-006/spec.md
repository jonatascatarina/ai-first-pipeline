# FEATURE-006 — Integração Bidirecional com GitHub Issues

## Contexto

O pipeline SDD do projeto opera inteiramente em arquivos Markdown locais. Quando um time usa GitHub Issues como rastreador de trabalho, há uma lacuna: specs em `specs/` e Issues no GitHub são artefatos desconexos. Novos requisitos chegam como Issues sem estrutura de spec; specs aprovadas ficam invisíveis para quem acompanha o projeto pelo GitHub.

## Problema

A desconexão entre specs locais e GitHub Issues gera dois atritos concretos:

- **Entrada:** Quem abre um Issue para solicitar uma feature não sabe que precisa preencher contexto, problema, critérios de aceite e não-escopo. O Issue chega vago e exige rodadas de clarificação antes de virar spec.
- **Saída:** Uma spec aprovada e implementada não aparece no issue tracker do GitHub. Stakeholders que acompanham o projeto via Issues não têm visibilidade do que foi especificado e planejado.

## Solução Proposta

Duas entregas complementares:

1. **Issue Template** — arquivo `.github/ISSUE_TEMPLATE/feature-spec.md` que estrutura a abertura de Issues no GitHub com os campos do pipeline SDD (contexto, problema, solução proposta, critérios de aceite, não-escopo). Quem abre um Issue via GitHub UI recebe o formulário certo.

2. **Comando `/speckit.issue`** — comando bidirecional que opera em dois modos:
   - **Modo `publish`:** lê uma spec local (`specs/<ID>/spec.md`) e cria um GitHub Issue com as informações essenciais da spec, usando `gh issue create`
   - **Modo `scaffold`:** lê um GitHub Issue existente (por número) via `gh issue view` e gera o scaffold de `specs/<ID>/spec.md` com os campos extraídos do Issue, pronto para o usuário completar

## Atores

- **Contribuidor externo** — abre Issues via GitHub UI usando o template, sem precisar conhecer o formato de spec
- **Desenvolvedor interno** — usa `/speckit.issue scaffold` para converter Issues em specs ou `/speckit.issue publish` para publicar specs como Issues
- **Agente** — executa as operações de leitura/criação de Issues e geração de arquivos (claude-sonnet-4-6)
- **GitHub CLI (`gh`)** — interface com a API do GitHub para criar e ler Issues

## Requisitos Funcionais

### Issue Template

1. O arquivo `.github/ISSUE_TEMPLATE/feature-spec.md` deve ser criado com as seções: Contexto, Problema, Solução Proposta, Critérios de Aceite e Não-Escopo
2. O template deve conter instruções em comentário HTML explicando o que preencher em cada seção
3. O template deve ter `labels: ["spec"]` no frontmatter YAML para categorização automática
4. O template deve ter `title: "[SPEC] "` como prefixo de título para identificação imediata

### Comando `/speckit.issue` — Modo Publish

5. O agente deve perguntar o ID da feature a publicar (ex: `FEATURE-005`)
6. O agente deve ler `specs/<ID>/spec.md` e extrair: título, contexto, problema, solução proposta e critérios de aceite
7. O agente deve criar o Issue via `gh issue create` com título, body formatado e label `spec`
8. O body do Issue deve conter link direto para `specs/<ID>/spec.md` no repositório
9. O agente deve retornar a URL do Issue criado ao usuário
10. Se `specs/<ID>/spec.md` não existir, o agente deve informar o erro e encerrar

### Comando `/speckit.issue` — Modo Scaffold

11. O agente deve perguntar o número do Issue a converter (ex: `42`)
12. O agente deve ler o Issue via `gh issue view <número> --json title,body,labels`
13. O agente deve extrair do corpo do Issue os campos correspondentes às seções do template
14. O agente deve determinar o próximo ID disponível consultando os diretórios existentes em `specs/`
15. O agente deve criar `specs/<ID>/spec.md` com os campos extraídos e marcadores `<!-- TODO: completar -->` onde a informação estiver ausente ou vaga
16. O agente deve adicionar comentário no Issue via `gh issue comment` informando que a spec foi criada com link para o arquivo
17. Se o Issue não existir ou `gh` retornar erro, o agente deve informar e encerrar sem criar arquivos

## Requisitos Não-Funcionais

- O comando `/speckit.issue` requer `gh` CLI autenticado no repositório correto
- O Issue Template deve funcionar sem nenhuma configuração adicional após o merge — apenas criando o arquivo na localização correta
- O modo `scaffold` não deve sobrescrever uma spec existente — deve alertar e pedir confirmação se o ID já tiver `spec.md`
- O body do Issue gerado pelo modo `publish` deve ser Markdown válido e legível sem contexto adicional
- O agente deve perguntar o modo (publish ou scaffold) antes de executar qualquer operação

## Critérios de Aceite

1. Dado que o repositório tem `.github/ISSUE_TEMPLATE/feature-spec.md`, ao abrir um novo Issue no GitHub a interface exibe o template com as seções preenchíveis
2. Dado `/speckit.issue` em modo `publish` com `FEATURE-002`, o Issue criado contém título correto, seções de Contexto, Problema e Critérios de Aceite, e link para `specs/FEATURE-002/spec.md`
3. Dado `/speckit.issue` em modo `publish` com ID inexistente, o agente exibe mensagem de erro sem criar Issue
4. Dado `/speckit.issue` em modo `scaffold` com número de Issue que usou o template, o arquivo `specs/<ID>/spec.md` gerado contém os campos do Issue mapeados para as seções corretas da spec
5. Dado `/speckit.issue` em modo `scaffold` com Issue que não usou o template (corpo livre), o arquivo gerado contém o que foi possível extrair com marcadores `<!-- TODO: completar -->` nas seções ausentes
6. Dado modo `scaffold`, um comentário é adicionado no Issue com link para a spec criada
7. Dado modo `scaffold` com número de Issue inexistente, o agente exibe erro e não cria nenhum arquivo
8. Dado modo `scaffold` com ID que já tem `spec.md`, o agente alerta e pede confirmação antes de sobrescrever

## Não-Escopo

- Sincronização automática bidirecional (webhook, CI)
- Fechar ou modificar Issues existentes além de adicionar comentário
- Leitura de comentários do Issue (apenas o body original)
- Suporte a GitHub Projects ou Milestones
- Criação de Issues em repositórios diferentes do atual
- Conversão de Issues de bug ou suporte — apenas Issues de feature/spec

## Dependências

- `gh` CLI instalado e autenticado com permissão de escrita no repositório
- Repositório hospedado no GitHub (não GitLab, Bitbucket, etc.)
- Para modo `publish`: spec existente em `specs/<ID>/spec.md`
- Para modo `scaffold`: Issue existente no repositório com número válido

## Riscos Conhecidos

- **MÉDIO — Issues sem template:** O modo `scaffold` deve funcionar mesmo com Issues que não usaram o template. A extração de campos por heurística (seções Markdown) pode falhar em Issues com corpo livre. Mitigação: marcadores `TODO` e apresentação do resultado para revisão antes de salvar.
- **BAIXO — Rate limit da API do GitHub:** Criação de muitos Issues em sequência pode atingir rate limit. Fora do escopo desta versão — o comando opera em um Issue por execução.
- **BAIXO — Spec desatualizada publicada:** O modo `publish` publica o estado atual da spec; se a spec mudar depois, o Issue fica desatualizado. Mitigação: o body do Issue inclui link para o arquivo no repositório — a versão canônica está sempre na spec.

## Perguntas em Aberto

Nenhuma — todas as decisões foram tomadas pelo SpecAgent com autonomia concedida pelo usuário.
