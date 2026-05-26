# Perguntas Respondidas — FEATURE-006

## P1 — O Issue Template deve usar o formato legado (arquivo único) ou o novo formato de formulário YAML do GitHub?

**Contexto:** O GitHub suporta dois formatos de Issue Template: o formato legado (arquivo Markdown em `.github/ISSUE_TEMPLATE/`) e o novo formato de formulário (`issue_form.yml`) com campos estruturados (dropdowns, checkboxes, textareas obrigatórios). O formulário YAML oferece validação nativa mas tem sintaxe mais complexa e nem todas as instalações GitHub Enterprise suportam.

**Resposta:** Formato legado Markdown. É mais simples, universalmente suportado, e se alinha com o princípio do projeto de "Markdown puro". O formulário YAML adicionaria complexidade sem ganho proporcional para o caso de uso — o template Markdown com instruções em comentário HTML é suficiente.

## P2 — O modo `publish` deve incluir o plano e as tasks no Issue, ou apenas a spec?

**Contexto:** Incluir o plano e as tasks no Issue tornaria o artefato mais completo para stakeholders técnicos. Mas tornaria o Issue muito longo para stakeholders não-técnicos que acompanham pelo GitHub.

**Resposta:** Apenas a spec (contexto, problema, critérios de aceite). O plano e as tasks são artefatos internos de implementação, não de comunicação com stakeholders. O Issue deve ser legível por qualquer pessoa interessada no produto. Link para `specs/<ID>/spec.md` dá acesso completo a quem quiser se aprofundar.

## P3 — O modo `scaffold` deve determinar o próximo ID automaticamente ou perguntar ao usuário?

**Contexto:** Determinar automaticamente o próximo ID (contando diretórios em `specs/`) é conveniente mas pode gerar conflitos se dois agentes rodam simultaneamente. Perguntar ao usuário é mais seguro mas adiciona fricção.

**Resposta:** Determinar automaticamente e apresentar ao usuário para confirmação antes de criar o arquivo. O agente lista os IDs existentes, propõe o próximo (ex: `FEATURE-007`), e aguarda confirmação. Isso é conveniente e seguro.

## P4 — O comentário adicionado no Issue pelo modo `scaffold` deve mencionar o agente ou apenas o link?

**Contexto:** Mencionar que um agente criou a spec pode ser útil para rastreabilidade, mas pode causar estranheza em times que não estão familiarizados com o workflow AI-first.

**Resposta:** Mencionar brevemente. O comentário deve ser profissional e informativo: "Spec criada em `specs/<ID>/spec.md` a partir deste Issue. [Ver spec](link)." — sem jargão de IA. O tom é o de um desenvolvedor que completou um passo do processo.

## P5 — O label `spec` deve ser criado automaticamente pelo agente se não existir no repositório?

**Contexto:** Se o repositório não tiver o label `spec`, o `gh issue create --label spec` falhará. O agente poderia criar o label automaticamente, mas isso é uma ação destrutiva no repositório.

**Resposta:** O agente deve tentar criar com o label e, se falhar, criar sem o label e avisar o usuário para criar o label manualmente. Não criar labels automaticamente — isso requer permissões extras e pode ter efeitos colaterais inesperados.

## P6 — O modo `scaffold` deve criar apenas `spec.md` ou também `perguntas-respondidas.md` vazio?

**Contexto:** Criar apenas `spec.md` é o mínimo necessário. Criar `perguntas-respondidas.md` vazio lembra o usuário que esse arquivo precisa ser preenchido antes de avançar para o plano.

**Resposta:** Apenas `spec.md`. O arquivo de perguntas é gerado pelo `/speckit.clarify` como parte natural do fluxo. Criar arquivos vazios antecipadamente vai contra o princípio de não criar artefatos antes de serem necessários.
