# Como Contribuir

Obrigado por contribuir com o ai-first-pipeline. Este documento descreve o processo para propor mudanças no template.

---

## Pré-requisitos

- Familiaridade com `constitution.md` (leitura obrigatória antes de qualquer contribuição)
- Conta no GitHub
- Git instalado localmente

---

## Tipos de Contribuição

### 1. Melhoria de comandos `/speckit.*`

Os comandos em `.claude/commands/` são o coração do template. Para melhorar um comando:

1. Abra uma issue descrevendo o problema atual com o prompt
2. Inclua um exemplo concreto de output ruim que o prompt gera
3. Proponha o prompt revisado na issue antes de abrir PR
4. O PR deve incluir atualização em `CHANGELOG.md`

### 2. Novo exemplo em `specs/`

Para adicionar um novo exemplo de spec:

1. Crie `specs/EXAMPLE-NNN/` com o próximo número sequencial
2. Preencha todos os cinco arquivos (`spec.md`, `perguntas-respondidas.md`, `plan.md`, `tasks.md`, `analysis-report.md`)
3. O exemplo deve cobrir um domínio não coberto pelos exemplos existentes
4. Abra PR com título: `feat(examples): adiciona EXAMPLE-NNN — <descrição>`

### 3. ADR novo ou atualização

Para propor uma decisão arquitetural sobre o próprio template:

1. Crie `docs/adr/ADR-NNN-titulo.md` usando o formato do ADR-001 existente
2. ADRs são discutidos via PR antes de serem aprovados
3. ADRs aprovados são imutáveis — supersessão cria novo ADR

### 4. Correção de typo ou formatação

PRs simples de correção de texto não precisam de issue prévia. Use título: `fix(docs): <descrição>`

---

## Processo de PR

1. Fork o repositório
2. Crie branch: `feat/<descricao>` ou `fix/<descricao>`
3. Faça commits atômicos com mensagens descritivas
4. Abra PR contra `main` com:
   - Título seguindo Conventional Commits
   - Descrição explicando o porquê da mudança
   - Referência à issue se houver
5. Aguarde revisão — o foco da revisão é conformidade com `constitution.md`

---

## Padrões de Formatação

- Markdown puro, sem frontmatter YAML
- Headings até `###`
- Listas com `-`
- Sem emojis em documentos de spec
- Blocos de código com linguagem especificada
- Linhas com no máximo 120 caracteres

---

## O Que Não Aceitar

- Código executável de qualquer tipo
- Dependências de runtime (npm, pip, etc.)
- Configurações específicas de projeto (substitua por placeholders)
- Mudanças que contradizem `constitution.md` sem ADR aprovado

---

## Dúvidas

Abra uma issue com a label `question`. Respostas em até 72 horas em dias úteis.
