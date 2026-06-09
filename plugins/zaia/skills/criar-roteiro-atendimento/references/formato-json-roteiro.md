# Formato JSON do roteiro (especificação da plataforma)

Siga este formato à risca. O importador rejeita estruturas fora do padrão.

## Estrutura do arquivo

- O JSON começa com um objeto com a chave `"flows"`: `{ "flows": [ ... ] }`.
- Nunca envie um array solto no topo (ex.: `[ { "name": ... } ]`): o importador rejeita.
- Vários roteiros vão TODOS dentro do mesmo array `flows`, separados por vírgula. Um único arquivo, um único `flows`.

```json
{ "flows": [ { "name": "Roteiro 1" }, { "name": "Roteiro 2" } ] }
```

## Campos do roteiro

| Campo | Obrigatório | Descrição |
|---|---|---|
| `name` | Sim | Nome do serviço (ex.: "Rescisão Trabalhista"). Um roteiro por serviço. |
| `description` | Sim | Descrição detalhada e específica para a IA saber QUANDO ativar o roteiro. Quanto mais específica, melhor. |
| `questions` | Sim (mínimo 1) | Array de passos, na ordem. |

`description` boa x ruim:

- Boa: "Atendimento de clientes demitidos sem justa causa, que pediram demissão ou tiveram rescisão indireta. Inclui verbas rescisórias, FGTS, seguro-desemprego e aviso prévio."
- Ruim: "Casos trabalhistas."

REGRA SOBRE A `description` (não confundir o papel dela): a `description` serve SÓ para a IA decidir QUANDO ativar/rotear este roteiro. Ela NÃO carrega instruções de atendimento. NÃO injete na descrição (nem repita em todo passo, como bloco de prosa) uma "regra de salvamento global" do tipo "a IA salva imediatamente qualquer informação válida, mesmo dita fora de ordem".

Atenção, isso NÃO quer dizer que a IA não deve salvar. Salvar é correto e desejável: configure o salvamento POR PASSO, com `saveData: true` e a `dataKey` daquele passo. A própria RECEPÇÃO deve salvar (nome, motivo e qualquer dado que o lead já tenha dado, mesmo fora de ordem), porque assim os roteiros seguintes não repetem perguntas (a regra do já-sabe usa exatamente esses dados salvos). O erro é só transformar isso numa regra global escrita em prosa na descrição; o certo é salvar via configuração do passo (`saveData` + `dataKey`).

## Campos de cada passo

| Campo | Obrigatório | Descrição |
|---|---|---|
| `questionText` | Sim | Título do passo (nome curto, ex.: "Acolhimento e contexto inicial"). NÃO é a pergunta ao cliente. |
| `responseText` | Sim | Instrução para o AGENTE de IA (o que perguntar, por que, como interpretar, tom). NÃO é o texto que o cliente lê. Parte mais importante: quanto mais detalhada, melhor. |
| `saveData` | Não (padrão false) | `true` se a resposta deve ser salva na memória do contato. |

Existe um segundo campo de passo, `userComplement`, e a plataforma usa DOIS layouts diferentes (ver a seção "Dois layouts de passo" abaixo). Em resumo: nos roteiros de serviço a instrução vai em `responseText`; nos roteiros padrão (Recepção, Suporte, Agendamento) a instrução vai em `userComplement` e o `responseText` fica vazio (`""`).

## Boas instruções por passo (o conteúdo de responseText)

A instrução é dirigida ao agente. Inclua sempre que fizer sentido:

- O que perguntar (a pergunta exata ou o tema).
- Por que perguntar (o contexto que justifica).
- Como interpretar a resposta.
- Tom esperado (empático, profissional, acolhedor).
- Quando salvar dados.
- Quando consultar a FAQ (nominando a categoria) para temas factuais importantes.
- Quando usar áudio, se premium: `Responda por ÁUDIO neste passo.`

Estrutura recomendada do roteiro: acolhimento, contextualização (perguntas abertas), coleta específica (datas, valores, documentos), encaminhamento. De 4 a 8 passos.

REGRA OBRIGATÓRIA, UMA informação por passo: cada passo (`question`) coleta UM único dado e faz UMA única pergunta, salvando UM único campo (`dataKey`). Regra do já-sabe: se o dado já está no histórico, a IA salva e pula o passo, sem perguntar nem mandar transição (padrão em triagem; confirmar só se o objetivo do assinante pedir). A plataforma só controla o avanço de passo em passo; juntar vários dados/perguntas num mesmo passo faz a IA pular dados, porque não há controle dentro do passo. Então cada informação vira um passo próprio. REGRA OBRIGATÓRIA, triagem E prospecção: cada passo é escrito no FORMATO DE 4 PONTOS (um passo por dado na triagem; um passo por etapa na prospecção). Detalhes que não podem faltar: o passo começa declarando que coleta APENAS aquele dado e o que NÃO perguntar ali; o ponto "Como responder" SEMPRE traz exemplos concretos de fala; no ponto "O que salvar", o aviso "Nunca deduza" vem logo abaixo do campo, e todo dado opcional é marcado como Facultativo (captado em silêncio, sem perguntar). Enumere os pontos (1 a 5, incluindo a regra de conclusão/anti-loop). Modelo canônico em `exemplos-roteiros-corretos.md`. O ponto "Como responder" (transição) é enviado JUNTO com a pergunta do próximo passo (não numa mensagem isolada) e SEMPRE com exemplos concretos por tipo de resposta, nada genérico. Os quatro pontos são: (1) Instrução (o objetivo daquele item/etapa e por que importa); (2) Como perguntar (uma única pergunta; variações apenas com gatilho REAL e observável que a IA consiga avaliar, como o histórico da conversa, todas perguntando a mesma coisa); (3) Como responder (como conduzir conforme a reação do cliente: comentário empático/transição na triagem, condução da objeção na prospecção, com variações por tipo de resposta); (4) O que salvar e como (campo, formato/valores aceitos, regra de quando esclarecer, nunca deduzir; na prospecção, salvar o estado, ex.: proposta_apresentada, objeção registrada, aceite). Modelos em `exemplos-roteiros-corretos.md`.

## Último passo: ação obrigatória

No passo final (e em qualquer passo de prospecção com variações por tipo de resposta), cada variação traz a **referência principal + a variação**, nunca só o acréscimo solto. Ou seja: a mensagem principal daquela etapa continua presente, e a variação só complementa conforme a resposta do contato.

REGRA CRÍTICA, o ÚLTIMO passo é SÓ a transferência. O último passo de todo roteiro carrega APENAS a ação (`completionAction`) e a mensagem de transferência ao cliente (`transferResponse`, quando `TRANSFER_HUMAN`). Ele NÃO faz pergunta nem coleta dado. Motivo: se o último passo tiver uma pergunta, a IA tende a enviar DUAS mensagens juntas no momento de transferir (a pergunta deste passo + a primeira pergunta do roteiro de destino). Por isso:

- Toda recapitulação, validação ("faz sentido?"), costura ou última coleta vai em um passo PRÓPRIO, imediatamente ANTES do passo de transferência.
- O passo de transferência tem `responseText`/`userComplement` apenas com a instrução de transferir (e, em `TRANSFER_HUMAN`, a `transferResponse`), sem pergunta.
- Em triagem: se faltava onde encaixar a costura/validação, ADICIONE um passo a mais para isso e deixe o último passo exclusivamente para a transferência.


O último passo de todo roteiro DEVE ter `completionAction`. Escolha uma das três.

### Opção 1: Transferir para humano (mais comum)

Separe a instrução (para o agente) da mensagem ao cliente: `responseText` é a instrução; `transferResponse` é o texto literal que o cliente recebe.

```json
{
  "questionText": "Encaminhamento para o advogado",
  "responseText": "Instrução: faça um resumo interno do que foi conversado. Demonstre confiança no escritório. Em seguida, envie ao cliente a mensagem de transferResponse e finalize para um advogado assumir.",
  "saveData": false,
  "completionAction": "TRANSFER_HUMAN",
  "titulo": "Novo caso de Rescisão Trabalhista",
  "contextText": "Cliente relatou [resumo]. Documentos: [lista]. Dados coletados: [campos salvos].",
  "transferDeadline": "48 horas",
  "transferResponse": "Obrigada pela confiança! O advogado especialista já recebeu as informações do seu caso e entrará em contato em até 48 horas."
}
```

### Opção 2: Transferir para outro roteiro

```json
{
  "questionText": "Direcionar para roteiro específico",
  "responseText": "Instrução: com base no coletado, identifique o roteiro mais adequado e informe ao cliente que será direcionado a um atendimento especializado.",
  "saveData": false,
  "completionAction": "TRANSFER_DEPARTMENT",
  "completionTarget": "slug-do-roteiro-destino"
}
```

### Opção 3: Encerrar conversa (roteiros informativos)

```json
{
  "questionText": "Finalização do atendimento",
  "responseText": "Instrução: resuma o que foi informado, pergunte se restou dúvida, agradeça e encerre de forma cordial.",
  "saveData": false,
  "completionAction": "END_CONVERSATION"
}
```

## Campos exclusivos do último passo

| Campo | Quando | Descrição |
|---|---|---|
| `completionAction` | Sempre no último passo | `TRANSFER_HUMAN`, `TRANSFER_DEPARTMENT` ou `END_CONVERSATION`. |
| `completionTarget` | Só com `TRANSFER_DEPARTMENT` | Slug do roteiro de destino. |
| `titulo` | Só com `TRANSFER_HUMAN` | Título da tarefa criada para a equipe. |
| `contextText` | Só com `TRANSFER_HUMAN` | Contexto da tarefa com resumo dos dados. |
| `transferDeadline` | Só com `TRANSFER_HUMAN` | Prazo sugerido de retorno (ex.: "48 horas", "1 dia útil"). |
| `transferResponse` | Só com `TRANSFER_HUMAN` | Mensagem literal enviada ao cliente na transferência (sem "Instrução:", o cliente lê isto). |

## Dois layouts de passo (importante, não misture)

A plataforma exporta/importa os passos em dois layouts. Use o layout certo conforme o tipo de roteiro e, ao ATUALIZAR um roteiro existente, **leia-o antes (`ler_roteiro`) e mantenha o mesmo layout que ele já usa**.

### Layout A, roteiros de serviço (triagem, prospecção, demais temas)
A instrução do agente vai em `responseText`. Use este layout em tudo que você cria do zero para um serviço.

```json
{ "questionText": "Identificação do tipo de débito", "responseText": "Instrução ao agente: ...", "saveData": true }
```

### Layout B, roteiros PADRÃO do sistema (Recepção, Suporte, Agendamento)
A instrução do agente vai em `userComplement`, e `responseText` fica **vazio** (`""`). É assim que a plataforma mantém esses três roteiros. Ao recriar ou atualizar a Recepção ou o Suporte, siga este layout, senão o resultado fica fora do padrão do sistema.

```json
{
  "questionText": "Saudação",
  "responseText": "",
  "saveData": false,
  "userComplement": "Instrução ao agente: cumprimente o lead apenas na primeira mensagem ..."
}
```

Regra de ouro: roteiro de serviço usa `responseText`; Recepção/Suporte/Agendamento usam `userComplement` com `responseText: ""`. Na atualização, espelhe o layout que o roteiro já tem.

## Roteiros padrão do sistema

Já existem e não precisam ser recriados: Recepção, Suporte e Agendamento (Google Calendar, premium, só aparece com o Calendar conectado). Ao gerar, foque nos roteiros de serviço, e ajuste a Recepção/Suporte só se o assinante pedir.

## Mídia

Se um passo referenciar mídia (imagem, vídeo, documento, áudio), a mídia precisa ser criada antes na plataforma (aba "Envio de Mídia"). Referência a mídia inexistente quebra a importação. Oriente o assinante a criar as mídias primeiro.

## Erros de importação mais comuns (checar antes de entregar)

- O arquivo contém algo além do JSON: texto explicativo antes/depois, ou cercas markdown (```json). O arquivo `.json` deve ter SOMENTE o JSON puro.
- Booleano como string: `"saveData": "true"` em vez de `"saveData": true`.
- JSON não validado: sempre rode um parser (`python3 -c "import json;json.load(open('arquivo.json'))"`) antes de entregar.

- Array solto em vez de `{ "flows": [...] }`.
- Último passo sem `completionAction`.
- `transferResponse` faltando num `TRANSFER_HUMAN`, ou mensagem ao cliente colocada dentro de `responseText`.
- `completionTarget` apontando para um slug que não existe.
- `description` genérica demais (a IA não consegue decidir quando ativar).
