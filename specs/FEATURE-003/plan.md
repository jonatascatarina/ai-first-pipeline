# Plano de Implementação — FEATURE-003

## Resumo Técnico

Esta feature é implementada inteiramente como um arquivo Markdown em `.claude/commands/speckit.changelog.md`. O agente executa comandos git via bash para obter o log, lê os arquivos de spec referenciados, produz o rascunho e aguarda confirmação antes de modificar `CHANGELOG.md`. Zero dependências de runtime além do git.

## Abordagem

### Fluxo Principal

```
/speckit.changelog
  │
  ├── Pergunta versão (ex: 1.1.0) e data (padrão: hoje)
  │
  ├── Executa: git tag --sort=-version:refname | head -1
  │     ├── Tag encontrada → git log <tag>..HEAD --oneline
  │     └── Sem tags → git log --oneline
  │
  ├── Para cada commit:
  │     ├── Extrai prefixo (feat/fix/chore/docs/test)
  │     ├── Extrai IDs de spec (regex: FEATURE-\d{3}|EXAMPLE-\d{3})
  │     └── Se ID encontrado → lê specs/<ID>/spec.md (título + solução proposta)
  │
  ├── Agrupa por spec (múltiplos commits → 1 entrada)
  ├── Classifica: feat→Adicionado, fix→Corrigido, resto→Interno
  │
  ├── Apresenta rascunho ao usuário
  │     ├── Usuário confirma → insere seção no CHANGELOG.md
  │     └── Usuário recusa → encerra sem modificar
  │
  └── Confirma: "Seção [X.Y.Z] adicionada ao CHANGELOG.md"
```

### Estrutura do Output

```markdown
## [1.1.0] — 2026-05-26

### Adicionado
- **PR Review Checklist Generator** (FEATURE-002): comando `/pr-checklist` que gera
  checklist de revisão categorizado (Segurança, Testes, Arquitetura, Documentação)
  a partir do título e descrição de um PR

### Corrigido
- Typo no comando `gh repo create` do quickstart do README

### Interno
- Testes manuais do comando `/pr-checklist` documentados em test-results.md
```

### Comandos Git Necessários

O agente precisa executar (via bash no Claude Code):

```bash
# Última tag
git tag --sort=-version:refname | head -1

# Log desde última tag (ou todos os commits)
git log v1.0.0..HEAD --oneline --no-merges
# ou
git log --oneline --no-merges

# Conteúdo de uma spec
cat specs/FEATURE-002/spec.md
```

### Componente único

**Arquivo:** `.claude/commands/speckit.changelog.md`

O prompt instrui o agente a:
1. Coletar versão e data
2. Executar os comandos git listados acima
3. Processar o output seguindo as regras de classificação e agrupamento
4. Apresentar rascunho
5. Escrever no `CHANGELOG.md` somente após confirmação

## Dependências

- Git instalado e repositório inicializado (pré-condição do projeto)
- `CHANGELOG.md` com seção `## [Não lançado]` (parte do template)
- Specs em `specs/<ID>/spec.md` para features referenciadas

## Riscos

### Riscos de qualidade do output
- **MÉDIO — Commits mal formatados:** Mensagens sem prefixo convencional caem em "Interno" com a mensagem literal. Mitigation: o prompt instrui a limpar mensagens óbvias (ex: remover hash do commit)
- **BAIXO — Spec muito longa:** O agente lê spec.md completo; specs longas consomem contexto. Mitigação: o prompt instrui a usar apenas título (`# `) e seção `## Solução Proposta`

### Riscos de segurança
- **NENHUM:** O comando lê arquivos locais e executa git — sem acesso a rede, sem credenciais

### Riscos de manutenibilidade
- **BAIXO:** Se o formato do `CHANGELOG.md` mudar, o prompt precisa ser atualizado. Mitigado pela instrução explícita de formato no prompt.

## Estimativa de Complexidade

| Área | Complexidade | Justificativa |
|------|-------------|---------------|
| Prompt engineering | M | Lógica de agrupamento e classificação requer instrução precisa |
| Integração git | P | Comandos simples, output previsível |
| Testes | P | 2-3 casos manuais cobrem os critérios de aceite |
