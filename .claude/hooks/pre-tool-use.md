# Hook: pre-tool-use

Este documento define o padrão de decisão para o hook `pre-tool-use` do Claude Code. O hook é executado antes de qualquer uso de ferramenta pelo agente e pode permitir, alertar ou bloquear a operação.

Configure o hook em `.claude/settings.json` apontando para um script shell que implementa as regras abaixo.

---

## Padrão de Decisão

Cada regra tem três saídas possíveis:

- **PERMITIR** — operação segue normalmente
- **ALERTAR** — operação prossegue, mas o humano é notificado com contexto
- **BLOQUEAR** — operação é interrompida; agente recebe mensagem de erro e deve parar

A decisão é tomada na ordem das regras abaixo. A primeira regra que corresponder ao alvo determina a saída.

---

## Regras de Proteção

### Arquivos protegidos (sempre BLOQUEAR)

Qualquer tentativa de escrita nos arquivos abaixo deve ser bloqueada:

- `constitution.md` — alterações exigem consenso humano (Artigo VIII.4)
- `CLAUDE.md` — instrução de comportamento de agentes; alteração não autorizada invalida o processo
- `.claude/settings.json` — configuração de permissões e hooks
- `docs/adr/*.md` — ADRs são imutáveis após aprovação (Artigo VII.3)
- `.github/workflows/*.yml` — pipelines de CI/CD não devem ser modificados por agentes sem revisão humana

### Credenciais e dados sensíveis (sempre BLOQUEAR)

Bloquear qualquer operação de escrita que contenha:

- Padrões de secrets: `ghp_`, `sk-`, `AKIA`, `-----BEGIN`, `password=`, `secret=`
- Variáveis de ambiente com valores reais (permitido apenas referências como `$VAR`)
- Dados pessoais identificáveis (CPF, e-mail, telefone de pessoas reais)

### Operações destrutivas em specs aprovadas (ALERTAR)

Alertar quando o agente tentar:

- Deletar ou sobrescrever `specs/<ID>/spec.md` de uma feature com status `aprovada`
- Modificar critérios de aceite em spec já implementada sem instrução explícita do humano

### Operações fora do escopo da feature ativa (ALERTAR)

Alertar quando o agente tentar escrever em `specs/<ID>/` de uma feature diferente da que está em execução.

---

## Exemplo de Regra Preenchida

Regra: bloquear escrita em `constitution.md`

```bash
#!/usr/bin/env bash
# pre-tool-use hook — exemplo de regra para constitution.md

TOOL_NAME="$1"       # ex: "Edit", "Write"
TARGET_FILE="$2"     # ex: "constitution.md"

if [[ "$TOOL_NAME" == "Edit" || "$TOOL_NAME" == "Write" ]]; then
  if [[ "$TARGET_FILE" == "constitution.md" ]]; then
    echo "BLOQUEADO: constitution.md só pode ser alterada por instrução explícita do humano responsável (Artigo VIII.4)." >&2
    exit 1  # sinaliza bloqueio ao Claude Code
  fi
fi

exit 0  # permite
```

Adapte o script adicionando blocos `if` para cada regra listada acima. Registre o script em `.claude/settings.json`:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "bash .claude/hooks/pre-tool-use.sh"
          }
        ]
      }
    ]
  }
}
```

---

## Adicionando Novas Regras

Para adicionar uma regra, siga o padrão:

```
Alvo: <arquivo, padrão ou operação>
Saída: PERMITIR / ALERTAR / BLOQUEAR
Motivo: <artigo da constituição ou razão de negócio>
Condição: <quando a regra se aplica>
```

Registre a regra neste documento antes de implementar no script shell.
