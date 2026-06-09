---
name: rotina-analise-diaria
description: Rotina diaria que varre os atendimentos e entrega ao assinante um painel de pendencias e oportunidades, com mensagens ja rascunhadas, sem disparar nada na Za.ia sem aprovacao. Cobre pendencias do escritorio (o que foi prometido e nao cumprido), contatos que pararam de responder, follow-up/remarketing, e lembretes/avisos. Use quando o assinante disser "analise diaria", "o que esta pendente", "quem parou de responder", "quem preciso retornar", "follow-up dos contatos", "remarketing", "o que prometemos e nao fizemos", "rodar a rotina do dia", ou quando disparada pela tarefa agendada. Leitura via MCP da Za.ia; nada e enviado ao cliente sem o assinante aprovar.
---

# Rotina de analise diaria (painel de pendencias e oportunidades)

Esta skill roda **todo dia** no lado do assinante e entrega um **painel** do que precisa de atencao: o que o escritorio prometeu e ainda nao cumpriu, quais contatos pararam de responder, quem vale a pena retomar (follow-up e remarketing) e quais lembretes/avisos estao chegando. Para cada coisa que merece uma acao, ela ja deixa uma **mensagem rascunhada**, pronta para o assinante revisar.

O ponto mais importante: a rotina **nao dispara nada sozinha**. Ela le os atendimentos (somente leitura) e propoe. Nenhuma mensagem vai para o cliente, nenhuma tarefa ou agendamento e criado na Za.ia, sem o assinante **aprovar item a item**.

O publico desta skill e o **proprio assinante**, em geral um advogado que nao e tecnico. Fale de forma clara, direta e pratica. Apresente o painel como um resumo do dia, sem jargao. A ideia e que, ao bater o olho, ele saiba exatamente o que esta pendente e o que pode disparar com um "sim".

## Base de conhecimento e atualizacao (consultar SEMPRE no inicio)

Antes de agir, leia o indice central via WebFetch:

`https://raw.githubusercontent.com/zaialsystem/zaia-marketplace/main/knowledge/manifest.json`

Do manifest, leia (via `rawBase` + path):

1. `rotinas/analise-diaria` (as heuristicas de como detectar pendencias, contatos frios, follow-up/remarketing e lembretes). Filtre os itens cujo campo `aplica` inclui esta skill.
2. `novidades` (o resumo do que mudou no plugin).

Em conflito com o conteudo embutido neste plugin, o conteudo do GitHub **prevalece**; o embutido e so fallback minimo. Nao invente conteudo de arquivo que nao foi lido. Se o fetch falhar, siga com o fallback minimo embutido e avise o assinante que esta sem a base atualizada.

Aviso de atualizacao: compare `manifest.pluginVersion` com a versao instalada em `${CLAUDE_PLUGIN_ROOT}/.claude-plugin/plugin.json`. Se o manifest for maior, prepare um aviso em linguagem simples ("Saiu uma atualizacao do plugin Za.ia. Veja as novidades e atualize em Customizar > Plugins.") com o resumo de `novidades`, e **inclua esse aviso no painel diario** (secao "Novidades do plugin").

## Config e estado (a skill LE o que a `configurar-zaia` gravou)

Esta rotina nao decide nada sobre pastas, partes ou horarios: ela apenas **le** o config que a skill `configurar-zaia` gravou. O schema ja e fixo no plugin.

- **Ponteiro fixo**: `${HOME}/.zaia-skills/config-pointer.json`, na forma `{ "configPath": "<abs>" }`. Leia o ponteiro, depois leia o `zaia-config.json` que ele aponta. Se o ponteiro ou o config nao existir, **pare** e oriente o assinante a rodar a skill `configurar-zaia` primeiro (sem ela, a rotina nao sabe onde guardar estado nem o que o assinante quer ver).
- **Campos relevantes do config**:
  - `analiseDiaria.ativa`: se estiver `false`, avise que a analise diaria esta desligada no config e nao rode (oriente a religar pela `configurar-zaia` se ele quiser).
  - `analiseDiaria.partes`: subconjunto de `["pendencias", "contatos_frios", "follow_up_remarketing", "lembretes"]`. **Compute somente as partes que estiverem nessa lista.**
  - `analiseDiaria.horario`: o horario em que a tarefa agendada dispara (informativo para a rotina).
  - `analiseDiaria.limiarContatoFrioDias`: a partir de quantos dias sem resposta um contato entra como "frio".
  - `paths.estado`: pasta onde vive o `zaia-estado.json` (cursor e itens ja tratados).
  - `paths.artefatos`: pasta onde a rotina salva o relatorio do dia.
- **Estado/cursor LOCAL** em `zaia-estado.json` na pasta `paths.estado`, na forma:

```json
{ "analiseDiaria": { "ultimoRun": "ISO", "itensTratados": [] }, "pecas": { "conversasGeradas": [] } }
```

  - Use `analiseDiaria.ultimoRun` para fazer a **leitura incremental** (so o que mudou desde a ultima execucao).
  - Use `analiseDiaria.itensTratados` para **nao repetir** uma proposta que ja foi tratada num painel anterior.
  - **NUNCA grave estado na Za.ia.** O estado e do plugin, fica local, e nunca vai para a plataforma nem para Git.

## Principios (deixe explicito e nao quebre)

- **Somente leitura na Za.ia por padrao.** A rotina, sozinha, so le os atendimentos. Nada de escrita sem aprovacao.
- **Nada criado na Za.ia sem aprovacao.** Mensagem, tarefa, agendamento: cada item proposto so vira acao na plataforma depois que o assinante disser "sim" para aquele item especifico.
- **Estado local, nunca na Za.ia.** O cursor e a lista de tratados ficam no `zaia-estado.json` do assinante.
- **Ler via MCP, nunca a API do WhatsApp.** Toda leitura passa pelas ferramentas MCP da Za.ia. A rotina jamais chama a API do WhatsApp direto.

## Workflow

### Passo 1: Ler o config

Leia o ponteiro `${HOME}/.zaia-skills/config-pointer.json` e, a partir dele, o `zaia-config.json`. Se nao existir, **pare** e mande o assinante rodar a skill `configurar-zaia` primeiro. Se existir mas `analiseDiaria.ativa` for `false`, avise que a analise esta desligada e pare (ou rode so se o assinante pediu explicitamente agora).

### Passo 2: Ler as heuristicas e as novidades

Leia, do manifest, `rotinas/analise-diaria` (heuristicas) e `novidades`. Prepare o aviso de atualizacao do Passo "Novidades" comparando `manifest.pluginVersion` com o `plugin.json` instalado.

### Passo 3: Ler os atendimentos pela plataforma (MCP), somente leitura e incremental

Leia a partir de `analiseDiaria.ultimoRun` (so o que mudou desde a ultima execucao). Use estas ferramentas MCP, **todas de leitura**:

- **`listar_conversas`**: as conversas, para ver o que esta em aberto e quem ficou sem resposta.
- **`listar_atendimentos_ia`**: o que a IA atendeu, para achar o que foi prometido/encaminhado e o que ficou pendente de uma acao humana.
- **`listar_tarefas`**: as tarefas do escritorio, para achar as vencidas ou proximas do vencimento.
- **`listar_contatos`** / **`buscar_contatos`**: os contatos, para cruzar quem esfriou e quem vale retomar.
- **`ler_conversa`**: quando precisar do detalhe de uma conversa especifica (por exemplo, confirmar exatamente o que foi prometido antes de rascunhar a mensagem).

**NUNCA** use a API do WhatsApp. Toda leitura e pelo MCP da Za.ia.

### Passo 4: Computar SO as partes ligadas no config

Calcule apenas as partes presentes em `analiseDiaria.partes`, aplicando `analiseDiaria.limiarContatoFrioDias` onde fizer sentido. As quatro partes:

- **pendencias** (pendencias do escritorio): o que foi **prometido e nao cumprido** (a IA ou o atendente disse que ia retornar, enviar, agendar, e nao consta que cumpriu) somado as **tarefas vencidas** (`listar_tarefas`). E a lista do que o escritorio "deve" para os clientes.
- **contatos_frios**: contatos que **pararam de responder** ha mais dias do que `limiarContatoFrioDias` (a ultima mensagem foi do escritorio/IA e o cliente nao voltou). Quem ficou no vacuo.
- **follow_up_remarketing**: oportunidades de **retomar contato**: quem demonstrou interesse e esfriou, quem pediu para pensar, quem nao fechou. Vale um toque para reaquecer.
- **lembretes**: compromissos, prazos e **avisos** que estao chegando (audiencias, reunioes, datas combinadas na conversa), para o assinante nao perder.

Para cada item que merece acao, deixe uma **mensagem ja rascunhada** (no tom do assinante, sem travessao), pronta para ele revisar e aprovar.

### Passo 5: Entregar o RELATORIO ao assinante

No app, entregue um **resumo claro** do painel, organizado pelas partes calculadas, com cada item e sua mensagem rascunhada. **Salve o relatorio** na pasta `paths.artefatos` (um arquivo por dia, ex.: `analise-diaria-AAAA-MM-DD.md`), para o assinante poder reabrir depois. Inclua no relatorio a secao "Novidades do plugin" (Passo 7).

Antes de cada item, deixe explicito que **nada foi enviado ainda** e que ele aprova um a um.

### Passo 6: Aprovacao (nada vai para a Za.ia nem para o cliente sem o assinante aprovar)

Apresente os itens e peca a aprovacao **item a item**. Nenhuma mensagem ao cliente, nenhuma tarefa, nenhum agendamento e criado na Za.ia sem o "sim" do assinante para aquele item.

- Ao **aprovar** um item, execute a acao correspondente via MCP: **`enviar_mensagem`** (mandar a mensagem rascunhada), **`agendar_mensagem`** (programar para depois) ou **`criar_tarefa`** (registrar a pendencia como tarefa).
- Depois de executar, **registre o id do item em `analiseDiaria.itensTratados`** no estado local, para a proxima rodada nao propor de novo.
- Se ele **recusar ou adiar** um item, nao execute nada; pode registrar como tratado se ele pedir para nao ver mais, ou deixar para reaparecer amanha.

### Passo 7: Incluir o bloco "Novidades do plugin"

Se o aviso de atualizacao do Passo 2 indicou versao nova, inclua no relatorio uma secao curta "Novidades do plugin" com o resumo de `novidades` e a orientacao de atualizar em Customizar > Plugins.

### Passo 8: Atualizar o cursor ao fim

Ao terminar, grave `analiseDiaria.ultimoRun` com a data/hora atual (ISO) no `zaia-estado.json`. Assim a proxima execucao le so o que mudou desde agora. Esse arquivo fica **local**, nunca na Za.ia.

### Observacao sobre execucao agendada (idempotencia e tick perdido)

Esta rotina roda no Cowork do assinante e depende do app dele estar aberto no horario. Por isso ela e **idempotente e tolerante a tick perdido**: se o horario passou com o app fechado, nada se perde. Na proxima execucao, o **cursor** (`ultimoRun`) faz a rotina recuperar tudo que aconteceu desde a ultima rodada de fato, e a lista `itensTratados` impede propostas repetidas. Rodar duas vezes no mesmo dia nao duplica nada: o que ja foi tratado nao volta, e o que ja foi lido nao e reproposto.

## Formato do relatorio

Use uma estrutura assim, mostrando so as partes que o config ligou:

```
## Analise diaria de [data]

Nada foi enviado ainda. Voce aprova cada item antes de qualquer disparo.

### Pendencias do escritorio (prometido e nao cumprido + tarefas vencidas)
- [Contato/caso]: [o que foi prometido e quando]. Mensagem rascunhada:
  "[texto pronto para revisar]"

### Contatos que pararam de responder (mais de [N] dias)
- [Contato]: ultima resposta ha [X] dias. Mensagem rascunhada:
  "[texto pronto para revisar]"

### Follow-up e remarketing
- [Contato]: [por que vale retomar]. Mensagem rascunhada:
  "[texto pronto para revisar]"

### Lembretes e avisos
- [Compromisso/prazo] em [data].

### Novidades do plugin
[Se houver versao nova: resumo das novidades e como atualizar.]
```

## Observacao sobre estilo de escrita

A Za.ia nao usa travessao na escrita. Ao redigir as mensagens rascunhadas e o proprio relatorio, use virgula, dois-pontos ou parenteses no lugar.
