---
name: automacao-pecas
description: Gera pecas (documentos) a partir dos dados coletados num atendimento, no formato e branding do proprio assinante, em .docx. Modo manual (o assinante aponta um atendimento e pede a peca) e modo automatico (vigia um slug de roteiro e gera as novas pecas a cada 30 a 60 min). Agnostica ao tipo: o assinante decide quais pecas faz (procuracao, contrato, substabelecimento, parecer, declaracao) enviando os modelos dele. Use quando o assinante disser "gerar procuracao", "criar o contrato desse atendimento", "gerar a peca", "montar o documento do cliente", "automatizar as pecas", "gerar documento a partir da conversa", ou quando disparada pela tarefa agendada. Avisa o assinante para revisar/assinar; nada e enviado ao cliente.
---

# Automacao de pecas (gerar documentos a partir do atendimento)

Esta skill **gera pecas** (documentos juridicos) a partir dos dados que um atendimento ja coletou, no **formato e no branding do proprio assinante**, e entrega o arquivo em `.docx` (e PDF, se ele quiser). A peca sai pronta para o assinante **revisar e assinar**.

Ela tem dois modos. No **modo manual**, o assinante aponta um atendimento (id da conversa, telefone ou nome) e pede a peca na hora. No **modo automatico**, ela vigia um roteiro especifico e, a cada 30 a 60 minutos, gera as pecas dos atendimentos novos que ja concluiram a coleta de dados. O assinante pode usar so um modo ou os dois.

A skill e **agnostica ao tipo de peca**: ela nao tem uma lista fixa de documentos. Quem decide quais pecas faz e o assinante, e ele decide isso simplesmente enviando os **modelos dele** (uma procuracao, um contrato, um substabelecimento, um parecer, uma declaracao, o que ele usar). A peca gerada segue o modelo que o assinante forneceu.

O publico desta skill e o **proprio assinante**, em geral um advogado que nao e tecnico. Fale de forma clara, direta e pratica. Entregue o arquivo e avise, sem jargao, o que ele precisa conferir antes de usar.

Dois pontos que nunca mudam, diga com suas palavras quando entregar:

- **Nada vai para o cliente.** A peca e gerada para o **assinante**, que revisa e assina. A skill nunca envia o documento ao cliente.
- **Nada e gravado na Za.ia sem aprovacao.** A leitura dos atendimentos e somente leitura; nenhuma escrita na plataforma acontece sem o assinante autorizar.

## Base de conhecimento e atualizacao (consultar SEMPRE no inicio)

Antes de agir, leia o indice central via WebFetch:

`https://raw.githubusercontent.com/zaialsystem/zaia-marketplace/main/knowledge/manifest.json`

Leia (via `rawBase` + path) `pecas/como-gerar-peca` (metodologia) e `rotinas/deteccao-pecas` (deteccao para o modo automatico), e `novidades`. Em conflito, o GitHub prevalece; se o fetch falhar, use o fallback e avise. Aviso de atualizacao: compare `manifest.pluginVersion` com `${CLAUDE_PLUGIN_ROOT}/.claude-plugin/plugin.json`.

## Config e estado (a skill LE o que a `configurar-zaia` gravou)

Esta skill nao decide nada sobre pastas, modo ou cadencia: ela apenas **le** o config que a skill `configurar-zaia` gravou. O schema ja e fixo no plugin.

- **Ponteiro fixo**: `${HOME}/.zaia-skills/config-pointer.json`, na forma `{ "configPath": "<abs>" }`. Leia o ponteiro, depois leia o `zaia-config.json` que ele aponta. Se o ponteiro ou o config nao existir, **pare** e oriente o assinante a rodar a skill `configurar-zaia` primeiro (sem ela, a skill nao sabe onde estao os modelos, onde salvar as pecas, nem o que o assinante quer).
- **Campos relevantes do config**:
  - `pecas.ativa`: se estiver `false`, avise que a automacao de pecas esta desligada no config e nao rode (oriente a religar pela `configurar-zaia` se ele quiser).
  - `pecas.modo`: `"manual"`, `"automatico"` ou `"ambos"`. Define se a skill espera o assinante pedir, se ela vigia sozinha, ou os dois.
  - `pecas.slugVigiado`: o slug do roteiro que, ao concluir, dispara a peca no modo automatico.
  - `pecas.cadenciaMin`: `30` ou `60` (minutos entre verificacoes no modo automatico).
  - `pecas.formato`: `"docx"` (so o Word) ou `"docx+pdf"` (Word e PDF).
  - `paths.modelos`: pasta com os modelos de branding do assinante (os documentos que ele usa).
  - `paths.artefatos`: pasta onde a skill salva as pecas geradas.
  - `paths.estado`: pasta onde vive o `zaia-estado.json` (controle do que ja foi gerado).
- **Estado/cursor LOCAL** em `zaia-estado.json` na pasta `paths.estado`, na forma:

```json
{ "analiseDiaria": { "ultimoRun": "ISO", "itensTratados": [] }, "pecas": { "conversasGeradas": [] } }
```

  - Use **`pecas.conversasGeradas`** (lista de conversas/atendimentos que ja geraram peca) para **idempotencia**: nunca gere de novo a peca de um atendimento que ja esta nessa lista.
  - **NUNCA grave estado na Za.ia.** O estado e do plugin, fica local, e nunca vai para a plataforma nem para Git.

## Principios (deixe explicito e nao quebre)

- **Nunca enviar a peca ao cliente.** A peca vai para o **assinante** revisar e assinar. A skill jamais manda o documento ao cliente.
- **Nada gravado na Za.ia sem aprovacao.** A leitura dos atendimentos e somente leitura; nenhuma escrita na plataforma sem o assinante autorizar.
- **Nao inventar dado faltante.** Use so o que o cliente de fato disse no atendimento. Se faltar um dado para a peca (CPF, endereco, estado civil, o que o modelo exigir), **sinalize ao assinante exatamente o que falta** e, se util, sugira coletar esse dado no proprio atendimento (por exemplo, ajustando o roteiro de coleta). Nunca preencha um campo com suposicao.
- **Agnostica ao tipo.** Nao existe lista fixa de pecas. O assinante decide quais documentos faz enviando os modelos dele. Voce gera no formato do modelo que ele forneceu.
- **Branding do assinante, nunca de um escritorio especifico.** A peca segue o padrao visual/textual do assinante (parametrizado, generico). Jamais use cabecalho, marca ou identidade de um escritorio especifico que nao seja o do assinante.

## Workflow

### Passo 1: Ler o config

Leia o ponteiro `${HOME}/.zaia-skills/config-pointer.json` e, a partir dele, o `zaia-config.json`. Se nao existir, **pare** e mande o assinante rodar a skill `configurar-zaia` primeiro. Se existir mas `pecas.ativa` for `false`, avise que a automacao de pecas esta desligada e pare (ou rode so se o assinante pediu explicitamente agora).

### Passo 2: Ler a metodologia (e a deteccao, se for o automatico)

Leia, do manifest, **`pecas/como-gerar-peca`** (a metodologia generica de como montar uma peca a partir do atendimento) sempre. Se for rodar o **modo automatico**, leia tambem **`rotinas/deteccao-pecas`** (como detectar quais atendimentos ja estao prontos para gerar peca). Leia `novidades` para o aviso de atualizacao.

### Passo 3 (MODO MANUAL): o assinante aponta um atendimento e pede a peca

Quando o assinante pedir uma peca de um atendimento especifico:

1. **Identifique o atendimento.** Ele aponta por **id da conversa**, **telefone** ou **nome** do cliente. Se vier telefone ou nome, ache a conversa via MCP (por exemplo `buscar_conversa_por_telefone` ou `listar_conversas`) antes de seguir.
2. **Leia o historico, somente leitura, pelo MCP da Za.ia.** Use **`ler_conversa`** (o historico completo) e **`ler_atendimento_ia`** (o trace e os dados coletados pela IA). Extraia os dados do cliente **sem deduzir o que nao foi dito**.
3. **Escolha o modelo.** Procure em `paths.modelos` um modelo do **tipo de peca pedido** (ex.: se ele pediu "procuracao", use o modelo de procuracao dele). Se nao houver um modelo daquele tipo, use uma **estrutura generica de fallback** para aquele tipo de peca e avise o assinante que ele nao tem um modelo proprio ainda (e que vale enviar um pela `configurar-zaia` para as proximas sairem no padrao dele).
4. **Cheque o que falta.** Confira se todos os campos que o modelo exige tem dado no atendimento. Se faltar algo, **liste ao assinante o que falta** e nao invente; gere a peca com o que ha e marque os campos faltantes de forma visivel, ou pergunte antes de gerar, conforme fizer mais sentido.
5. **Gere o arquivo.** Produza o `.docx` (e o PDF, se `pecas.formato` for `"docx+pdf"`) usando a tecnica das skills de documento disponiveis (ver Passo 5 geral abaixo), com o **branding do assinante**. Salve em `paths.artefatos`.
6. **Avise o assinante.** Entregue o caminho do arquivo e diga, claro, que e para **revisar e assinar**, que nada foi enviado ao cliente, e o que (se algo) ficou faltando.
7. **Registre.** Anote o id da conversa em `pecas.conversasGeradas` no estado local.

### Passo 4 (MODO AUTOMATICO): vigiar o slug e gerar so as novas

Rode esta parte quando `pecas.modo` for `"automatico"` ou `"ambos"`, e quando a skill for disparada pela **tarefa agendada** (na cadencia `pecas.cadenciaMin`):

1. **Busque os atendimentos do roteiro vigiado, somente leitura, pelo MCP.** Use `listar_atendimentos_ia` (e `ler_atendimento_ia` no detalhe) filtrando pelo campo **`departamento`** igual a `pecas.slugVigiado`. Considere prontos os atendimentos que **concluiram a coleta** (chegaram ao passo final do roteiro / com os dataKeys preenchidos), conforme as regras de `rotinas/deteccao-pecas`.
2. **Cruze com o estado.** Compare a lista de atendimentos prontos com `pecas.conversasGeradas` (estado local) e **fique so com os novos** (os que ainda nao geraram peca).
3. **Gere so os novos.** Para cada atendimento novo, siga o mesmo metodo do modo manual (escolher modelo em `paths.modelos`, checar dados sem inventar, gerar `.docx` e PDF conforme o formato, salvar em `paths.artefatos`).
4. **Avise o assinante.** Entregue a lista das pecas geradas nesta rodada (com os caminhos), reforce que e para revisar e assinar, e sinalize qualquer atendimento que parecia pronto mas estava com dado faltando.
5. **Registre.** Adicione os ids gerados a `pecas.conversasGeradas`. Essa lista garante **idempotencia**: rodar de novo nao gera peca repetida.

**Idempotente e tolerante a tick perdido.** Esta rotina roda no app do assinante e depende dele estar aberto na hora. Se um horario passou com o app fechado, nada se perde: na proxima rodada, o cruzamento com `pecas.conversasGeradas` faz a skill pegar todos os atendimentos prontos ainda nao gerados, sem duplicar nenhum. Rodar duas vezes na mesma janela nao gera peca repetida.

### Passo 5: Como gerar o .docx (branding do assinante)

Para produzir o arquivo, use a tecnica das **skills de documento disponiveis** (por exemplo a skill `docx`), **MAS** o branding e o do **assinante**: generico e parametrizado a partir dos modelos em `paths.modelos`, nunca a identidade de um escritorio especifico. O modelo do assinante define cabecalho, fontes, blocos e o texto-base; os dados do atendimento preenchem os campos. Siga a metodologia de `pecas/como-gerar-peca` para mapear dado-do-atendimento para campo-do-modelo.

### Passo 6: Aviso de atualizacao do plugin

Se a comparacao de versoes indicou versao nova, inclua no aviso ao assinante uma nota curta ("Saiu uma atualizacao do plugin Za.ia. Veja as novidades e atualize em Customizar > Plugins.") com o resumo de `novidades`.

## Formato da entrega

Ao entregar (manual ou automatico), use uma estrutura assim:

```
## Pecas geradas

Nada foi enviado ao cliente. As pecas sao para voce revisar e assinar.

### [Tipo de peca] - [cliente/atendimento]
- Arquivo: [caminho do .docx (e do PDF, se houver)]
- Modelo usado: [seu modelo de [tipo]] ou [estrutura generica de fallback (voce ainda nao tem um modelo proprio desse tipo)]
- Faltando conferir: [campos sem dado no atendimento, se houver; senao "nada, todos os campos preenchidos"]

### Novidades do plugin
[Se houver versao nova: resumo das novidades e como atualizar.]
```

## Observacao sobre estilo de escrita

A Za.ia nao usa travessao na escrita. Ao conversar com o assinante e ao escrever qualquer texto (avisos, peca, relatorio), use virgula, dois-pontos ou parenteses no lugar.
