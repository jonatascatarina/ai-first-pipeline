# FEATURE-002 — PR Review Checklist Generator

## Contexto

Times de engenharia perdem tempo em revisões de PR porque cada revisor aplica critérios diferentes e inconsistentes. Perguntas importantes de segurança, testes e arquitetura ficam de fora quando dependem da memória do revisor. Um gerador de checklist a partir do título e descrição do PR padroniza o que deve ser verificado antes do merge.

## Problema

Sem uma checklist estruturada:
- Revisores ignoram categorias inteiras (ex: impacto em segurança) por falta de lembrança
- A qualidade da revisão varia por revisor, por dia e por nível de cansaço
- PRs com descrição vaga passam sem as perguntas críticas serem feitas
- Onboarding de novos revisores é lento porque os critérios estão na cabeça das pessoas

## Solução Proposta

Um comando `/pr-checklist` que recebe título e descrição de PR como input e gera um checklist Markdown categorizado com itens de verificação específicos ao contexto do PR. O output pode ser colado diretamente no comentário de revisão do PR.

## Atores

- **Revisor** — executa o comando antes de começar a revisão, usa o checklist gerado como guia
- **Autor do PR** — pode usar o comando para auto-revisar antes de submeter
- **Agente** — processa o input e gera o checklist (claude-sonnet-4-6)

## OUTCOMES (O que o sistema entrega)

- O sistema deve gerar um checklist Markdown a partir de título e descrição de PR
- O checklist deve conter categorias fixas: Segurança, Testes, Arquitetura, Documentação
- Cada categoria deve ter itens específicos derivados do contexto do PR (não genéricos)
- O output deve ser diretamente copiável para um comentário de PR

## SCOPE (Limite do sistema)

### Dentro do escopo
- Processamento de título e descrição de PR em texto livre
- Geração de checklist categorizado em Markdown
- Itens contextuais baseados em palavras-chave identificadas no input
- Seção de itens obrigatórios independentes de contexto

### Fora do escopo
- Integração com API do GitHub (leitura de PR real)
- Leitura ou análise de diff/código
- Interface de usuário (web, CLI própria)
- Persistência ou histórico de checklists gerados
- Integração com ferramentas de CI/CD

## BEHAVIOR (Comportamento em EARS notation)

**EARS-1 — Geração básica**
WHEN the agent receives a PR title and description,
THE SYSTEM SHALL generate a checklist with at minimum four sections: Segurança, Testes, Arquitetura, Documentação.

**EARS-2 — Itens contextuais de segurança**
WHEN the PR title or description contains keywords related to authentication, authorization, passwords, tokens, secrets, permissions, or user data,
THE SYSTEM SHALL include at minimum three security-specific checklist items derived from those keywords.

**EARS-3 — Itens contextuais de testes**
WHEN the PR title or description mentions new functionality, bug fix, or behavioral change,
THE SYSTEM SHALL include checklist items for unit tests, integration tests, and regression coverage specific to the described change.

**EARS-4 — Itens contextuais de arquitetura**
WHEN the PR title or description references database changes, new services, APIs, dependencies, or infrastructure,
THE SYSTEM SHALL include checklist items for migration safety, backward compatibility, and dependency impact.

**EARS-5 — Itens obrigatórios independentes de contexto**
REGARDLESS of the PR content,
THE SYSTEM SHALL always include: "[ ] CHANGELOG atualizado se mudança visível ao usuário" and "[ ] Spec referenciada no PR se feature nova".

**EARS-6 — Input insuficiente**
WHEN the PR title is empty or contains fewer than 5 characters,
THE SYSTEM SHALL request a more descriptive title before generating the checklist.

**EARS-7 — Formato de output**
THE SYSTEM SHALL always output valid GitHub-flavored Markdown with `- [ ]` checkboxes, section headers with `##`, and no YAML frontmatter.

## Critérios de Aceite

1. Dado título "Add JWT authentication to /api/users endpoint" e descrição adequada, o output contém seção `## Segurança` com ao menos 3 itens específicos sobre JWT, tokens e autenticação
2. Dado qualquer input válido, o output contém as quatro seções obrigatórias
3. Dado qualquer input válido, os dois itens obrigatórios (CHANGELOG e Spec) estão presentes
4. Dado título com menos de 5 caracteres, o agente solicita mais informações antes de gerar
5. O output é Markdown válido com checkboxes `- [ ]` e headers `##`
6. Os itens gerados são específicos ao contexto do PR, não genéricos como "verificar se os testes passam"

## Não-Escopo

- Integração com API do GitHub
- Leitura de diff ou arquivos modificados
- UI de qualquer tipo
- Configuração de categorias customizadas por projeto
- Suporte a múltiplos idiomas (output sempre em português)

## Dependências

- Nenhuma dependência técnica — feature é inteiramente um prompt Markdown em `.claude/commands/`
- Requer Claude Sonnet 4.6 como modelo executor (conforme CLAUDE.md)

## Riscos Conhecidos

- Itens genéricos demais: se o prompt não for específico o suficiente, o agente gera checklists genéricas que não agregam valor
- Falsos positivos de contexto: o agente pode detectar keywords e gerar itens irrelevantes (ex: "password" em uma mensagem de log que não envolve autenticação real)

## Perguntas em Aberto

Nenhuma — todas respondidas em `perguntas-respondidas.md`.
