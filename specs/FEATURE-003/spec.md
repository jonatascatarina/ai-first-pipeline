# FEATURE-003 — Gerador Automático de Changelog

## Contexto

O `CHANGELOG.md` deste template é escrito à mão. O projeto já tem duas fontes de verdade ricas: commits git (com mensagens estruturadas) e specs em `specs/` (com descrições detalhadas de cada feature). Um comando que cruza essas duas fontes gera changelogs consistentes, rastreáveis e sem esforço manual.

## Problema

Changelogs escritos manualmente sofrem de três problemas recorrentes:
- São esquecidos antes de releases — o time lembra de fazer, mas não antes do push
- São inconsistentes — cada pessoa escreve em nível de detalhe diferente
- São desconectados das specs — não referenciam o que foi prometido, apenas o que foi feito

## Solução Proposta

Um comando `/speckit.changelog` que lê o `git log` desde a última tag (ou range especificado pelo usuário), cruza os IDs de spec referenciados nos commits com os arquivos `specs/<ID>/spec.md`, classifica entradas por tipo (Adicionado, Corrigido, Removido), apresenta o rascunho para aprovação e insere a nova seção versionada no `CHANGELOG.md`.

## Atores

- **Responsável pelo release** — executa o comando antes de criar uma nova versão
- **Agente** — lê git log e specs, produz o rascunho (claude-sonnet-4-6)
- **`CHANGELOG.md`** — arquivo de destino, modificado somente após aprovação

## Requisitos Funcionais

1. O comando deve perguntar ao usuário a versão a ser registrada (ex: `1.1.0`) e a data (padrão: hoje)
2. O agente deve ler o `git log` desde a última tag git, ou todos os commits se nenhuma tag existir
3. Para cada commit, extrair referências a spec no padrão `FEATURE-NNN` ou `EXAMPLE-NNN`
4. Para cada spec referenciada, ler `specs/<ID>/spec.md` e usar o título e solução proposta como base da entrada
5. Classificar commits por prefixo convencional: `feat:` → Adicionado, `fix:` → Corrigido, `chore:/docs:/test:` → agrupar em "Interno"
6. Agrupar múltiplos commits da mesma spec em uma única entrada no changelog
7. Apresentar o rascunho completo ao usuário antes de escrever em disco
8. Inserir a nova seção abaixo de `## [Não lançado]` no `CHANGELOG.md`, no formato `## [X.Y.Z] — YYYY-MM-DD`
9. Commits sem referência a spec devem aparecer como entradas baseadas na mensagem de commit, classificadas pelo prefixo

## Requisitos Não-Funcionais

- Output sempre em português, independente do idioma dos commits
- Formato obrigatório: Keep a Changelog (`### Adicionado`, `### Corrigido`, `### Removido`)
- Máximo de uma entrada por spec no changelog — nunca listar cada commit separadamente
- O arquivo `CHANGELOG.md` só é modificado após confirmação explícita do usuário

## Critérios de Aceite

1. Dado commits `feat: SDD spec FEATURE-002` e `feat: implement /pr-checklist command (FEATURE-002)`, o output agrupa em uma única entrada referenciando FEATURE-002 com descrição da spec
2. Dado commit com prefixo `feat:`, a entrada aparece sob `### Adicionado`
3. Dado commit com prefixo `fix:`, a entrada aparece sob `### Corrigido`
4. Dado spec ID no commit e `specs/<ID>/spec.md` existente, a descrição usa o título da spec, não o texto do commit
5. Dado repositório sem tags git, usa todos os commits como range
6. A nova seção segue o formato `## [X.Y.Z] — YYYY-MM-DD`
7. Dado usuário responder "não" na confirmação, o `CHANGELOG.md` não é modificado
8. Dado commit sem referência a spec (`chore: fix typo`), a entrada aparece sob `### Interno` baseada na mensagem do commit

## Não-Escopo

- Bump automático de versão semântica — usuário informa a versão manualmente
- Push para GitHub Releases ou npm após geração
- Suporte a múltiplos CHANGELOGs (monorepos)
- Geração retroativa para versões já lançadas
- Validação de se os commits seguem Conventional Commits

## Dependências

- Git disponível e repositório inicializado
- `CHANGELOG.md` existente na raiz com o formato `## [Não lançado]`
- Specs em `specs/<ID>/spec.md` para cada feature referenciada nos commits

## Riscos Conhecidos

- **BAIXO — Spec ausente para commit referenciado:** Se o commit menciona `FEATURE-005` mas o arquivo não existe, o agente usa a mensagem do commit como fallback
- **BAIXO — Histórico de commits sem padrão:** Repositórios com mensagens livres produzem entradas de baixa qualidade; o template incentiva Conventional Commits mas não força

## Perguntas em Aberto

Nenhuma — todas as decisões foram tomadas pelo SpecAgent.
