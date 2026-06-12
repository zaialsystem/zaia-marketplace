# Regras gerais de roteiro

A IA é probabilística: ela segue melhor instruções concretas, com exemplos e destino-padrão para o que não se encaixa. Roteiro vago gera atendimento imprevisível. Estas são as regras gerais que valem para qualquer roteiro, mais o formato detalhado de cada passo de triagem e prospecção.

## Princípios gerais

- **Uma informação por passo**: cada passo coleta um único dado e faz uma única pergunta, salvando um único campo. A plataforma só controla o avanço passo a passo; juntar vários dados num passo faz a IA pular dados (não há controle dentro do passo).
- **O último passo é só a transferência/ação** (`TRANSFER_HUMAN`, `TRANSFER_DEPARTMENT` ou `END_CONVERSATION`), sem pergunta. Senão a IA manda duas perguntas juntas ao transferir (a deste passo e a primeira do roteiro de destino). A recapitulação/validação vai num passo antes.
- **Regra do já-sabe**: se o dado já está no histórico, a IA salva e pula o passo, sem perguntar nem mandar transição. Em triagem, o padrão é pular.
- **CONTEXTUALIZE**: toda pergunta precisa fazer sentido com o que já foi dito; confirme a dor específica do lead antes de perguntar.
- **Nada de mensagens genéricas** ("já registrei", "anotei", "perfeito", "ok"). Melhor não responder nada do que mandar genérico.
- **Salvar dados é correto**, inclusive na Recepção, configurado por passo (`saveData: true` + `dataKey`). Não escrever "regra de salvamento global" em prosa na descrição nem repetida em todo passo. A `description` do roteiro serve só para a IA saber quando ativar/rotear, não é instrução de atendimento.

## Roteiro ou FAQ? Onde colocar cada coisa

| Pergunte | Se sim | Vai para |
|---|---|---|
| É sobre como a IA deve se comportar, conduzir, transferir, o tom, o que pode/não pode dizer ou oferecer? | conduta | **Roteiro** |
| É uma informação factual que o cliente pergunta e a resposta é sempre a mesma (preço, prazo, documentos, horário, como funciona)? | dado | **FAQ** |
| A resposta muda conforme o momento/situação da conversa? | depende | **Roteiro** |
| A resposta é sempre idêntica, independentemente do contexto? | fixa | **FAQ** |

Regra de bolso: **FAQ guarda o dado, roteiro guarda a conduta.** Preço da análise é FAQ. "Como apresentar o preço sem pressionar o cliente" é roteiro. A divisão completa, e a ponte entre os dois, está em `integracao-roteiro-faq.md`.

## Como escrever instrução de roteiro

A IA segue melhor instruções que são concretas, imperativas e ancoradas em exemplos. Princípios:

### 1. Diga o que fazer, no imperativo, e o que não fazer
Evite adjetivos soltos. "Seja acolhedora" é fraco. Prefira: "Acolha em uma frase curta, informe que vai repassar à equipe e transfira. Não ofereça soluções nem opine sobre o caso."

### 2. Liste os casos previstos e o que fazer em cada um
Organize o passo por situações ("a) atualização de processo", "b) urgência emocional", "c) comprovante de pagamento"...) e dê a conduta de cada uma. Quanto mais o roteiro mapeia situações reais, menos a IA improvisa.

### 3. Tenha um destino-padrão para o que não se encaixa
Sempre defina o "se nada acima se aplicar, faça X" (em geral: acolher de forma neutra e transferir para humano/suporte). Isso elimina o improviso nos casos não previstos.

### 4. Use exemplos "errado x certo" com texto real
Este é o recurso mais poderoso. Para cada regra importante, mostre:

```
Errado: "[a resposta indesejada que de fato apareceu]"
Certo: "[a resposta que você queria]"
Por quê: [uma linha explicando o princípio]
```

Transforme cada erro real diagnosticado em um par desses. O erro de hoje vira a vacina de amanhã.

### 5. Coloque as regras que não podem ser quebradas em um bloco no topo
Um bloco de "regras invioláveis" no início do passo, dizendo que elas prevalecem sobre qualquer outro trecho, resolve a maioria dos conflitos. Ex.: neutralidade, nunca prometer, sempre nomear o responsável, não oferecer ações.

### 6. Repita o essencial perto do ponto de uso
Redundância proposital ajuda. Se "não prometer retorno imediato" é crítico, repita curtinho dentro de cada caso onde a IA tende a escorregar, não só no rodapé.

### 7. Uma ideia por mensagem
Para evitar mensagens longas e repetidas, instrua: "uma ideia por mensagem; antes de enviar, verifique se já não disse isso na mensagem anterior; se já disse, não repita."

### 8. Padronize o tom
Defina explicitamente: idioma e correção gramatical, proibição de caixa alta, como tratar o cliente (pelo primeiro nome), proibição de gerúndio para futuro ("vou verificar", não "vou estar verificando"), e proporcionalidade nas despedidas (não escalar um "obrigada" curto em três frases).

## Como escrever entrada de FAQ

### 1. Uma pergunta, uma resposta objetiva
Cada entrada cobre uma dúvida. Resposta direta, com o dado correto. Sem rodeio.

### 2. Cubra as variações de como perguntam
A IA precisa achar a entrada. Inclua sinônimos e formas diferentes da mesma pergunta (ex.: "valor", "preço", "quanto custa", "quanto fica"). Se a plataforma permite palavras-chave/tags, capriche nelas.

### 3. Escreva o dado, não a conduta
A FAQ informa. Como entregar isso (com ou sem pressão, em que momento) é papel do roteiro. Não misture.

### 4. Mantenha atualizado e sem contradição com o roteiro
Preço, prazo e condições mudam. FAQ desatualizada faz a IA dar informação errada com confiança. E confira se o que a FAQ diz bate com o que o roteiro manda dizer.

### 5. Diga o que a IA deve fazer quando a FAQ não cobre
Deixe claro (no roteiro) que, se a pergunta factual não estiver na FAQ, a IA não deve inventar: deve acolher e transferir. Inventar dado é um dos piores erros, e a causa quase sempre é FAQ ausente somada à ausência dessa salvaguarda.

## Formato de 4 (a 6) pontos por passo (triagem e prospecção)

Cada passo de triagem (por dado) ou de prospecção (por etapa) é escrito assim, enumerado:

1. **Instrução** (por que precisamos / objetivo do passo): explique o objetivo do dado, para a IA entender o que está fazendo (ex.: "o caso precisa ter ocorrido nos últimos 5 anos, acima disso há risco de prescrição").
2. **Como perguntar** (uma única pergunta): a frase exata (ou um modelo) que a IA deve usar, no tom do atendimento. Variações só com gatilho REAL e observável que a IA consiga avaliar (o histórico da conversa, ex.: "se o lead falou em protesto", "se pediu parcelamento", "se enviou documento"). Indique o gatilho de cada variação e mantenha todas perguntando a MESMA coisa. Evite critério vago. Conecte a pergunta ao que já foi coletado nos passos anteriores, não use boilerplate isolado.
3. **Aguarde a resposta do usuário.**
4. **O que salvar e como** (campo + valores aceitos, ex.: `sim` | `nao` | `nao_se_aplica`): o aviso **"Nunca deduza, salve só o que o cliente disse"** vem logo ABAIXO do campo a que se refere, não no fim. Diga o que salvar quando o item não se aplica (`nao_se_aplica`) e quando pular o item. Dado opcional deve ser marcado como **Facultativo** (captado em silêncio, ex.: `tipo_pessoa` que o lead deu sozinho, sem perguntar).
5. **Regra de conclusão e anti-loop**: diga quando o passo conclui (campo preenchido, com valor ou `nao_sabe`), que toda mensagem termina com a pergunta daquele dado, e proíba frases vazias ("o primeiro passo é identificar...", "depois podemos analisar...", "vamos seguir com isso").
6. **Como responder** (transição), por último, porque é enviada JUNTO com a pergunta do próximo passo, numa só mensagem, nunca como mensagem solta. SEMPRE com EXEMPLOS concretos de fala por tipo de resposta. PROIBIDO confirmação genérica ("já registrei", "anotei", "registrado", "perfeito", "ok"): é melhor NÃO responder nada e ir direto para a próxima pergunta. A transição só existe se acrescentar algo real e específico. Acolhe sem prometer solução, sem orientar, sem calcular prazo, sem falar preço.

O passo declara no título que coleta APENAS aquele dado e o que NÃO perguntar ali (ex.: "Coleta APENAS a fase da cobrança. Não pergunte órgão, documento, prazo ou valor aqui; cada um tem seu passo depois").

**CONTEXTUALIZE em todo passo (regra geral).** Toda pergunta precisa fazer sentido com o que já foi perguntado e informado. Quando o contato já trouxe uma dor/situação, confirme-a em uma linha e emende a pergunta do passo. Exemplos: (a) o contato mencionou receio de bloqueio, confirme o receio e pergunte se há mais algo que o aflige; (b) o contato disse que a venda de um imóvel está travada, confirme o travamento e pergunte o objetivo. Ex.: "Você havia mencionado que esses débitos na Prefeitura estão travando a venda do imóvel, certo? Há mais algum objetivo que te levou a procurar o escritório, como, por exemplo, ver a possibilidade de parcelamento desses valores?"

Esse formato (pontos enumerados, um dado por passo) é o padrão-ouro. Ao corrigir uma triagem, se faltar qualquer ponto em algum passo, ou se um passo juntar mais de um dado, esse é quase sempre o ponto de origem do erro.

## Padrão da Triagem

A triagem aprofunda o caso depois da recepção: acolhe, coleta os dados essenciais e qualifica (o caso serve ou não). É o roteiro que mais coleta dados, então a disciplina de coleta é tudo.

**Regra número um, UMA informação por passo.** Cada passo coleta UM único dado e faz UMA única pergunta, salvando UM único campo. Se você juntar vários dados numa instrução só, a IA pula algum. Então cada dado (fase, órgão, tipo de pessoa, valor) é um passo separado. As variações da pergunta são só formas diferentes de perguntar a MESMA coisa conforme o contexto.

**O passo declara o que coleta.** Comece o passo dizendo que ele coleta APENAS aquele dado e o que NÃO perguntar ali.

**Regra do já-sabe.** Antes de perguntar, a IA verifica se o dado já está no histórico. Se já estiver, SALVA e PULA para o próximo passo, sem perguntar e sem mensagem de transição. Em triagem o padrão é pular; confirmar (uma frase) só quando o objetivo do assinante pedir.

Cada passo de triagem é escrito no formato de pontos enumerados acima (um dado por passo). Sem todos os pontos, a IA improvisa a pergunta, aceita resposta vaga ou salva dado errado.

## Padrão do Suporte

O suporte é o fim de linha: encerra o atendimento automático e passa para humano. O risco aqui é prometer demais ou tratar mal o encerramento. Documente sempre:

- **Casos bem definidos com a mensagem pronta**: cada situação previsível (comprovante de pagamento, envio de documentos, pedido de falar com a equipe, caso genérico) com a frase exata de resposta.
- **Não prometer retorno imediato**: deixar explícito que o retorno é "em breve", não "agora". Nunca dizer que a pessoa responsável "já está ciente".
- **Ajuste por horário**: mensagem diferente dentro e fora do horário de atendimento (no fora do horário, informar quando será o retorno com base nos dias/horários cadastrados).
- **Não pedir dados fora do escopo nem fazer mais perguntas** depois de já ter decidido transferir.

## Layouts de passo (importante para o importador)

A plataforma tem dois layouts de passo:

- Roteiros de **serviço** (triagem, prospecção, temas): instrução no `responseText`.
- Roteiros **padrão** (Recepção, Suporte, Agendamento): instrução no `userComplement`, com `responseText` vazio (`""`).

Ao atualizar um roteiro, mantenha o layout que ele já usa. Recriou a Recepção ou o Suporte? Use o layout `userComplement`.

## Lembrete final

Nenhuma instrução garante 100%, porque a IA é probabilística. Mas instrução concreta + exemplo real + destino-padrão + FAQ atualizada empurram a probabilidade do acerto para perto do teto. Ao entregar a correção, sempre explique ao assinante **por que** aquele ajuste torna o erro menos provável: isso ensina ele a pescar, não só a receber o peixe.

## Aprendizado 2026-06-12: todo passo de coleta conduz e termina em pergunta (nada de passo só de acolhimento)

Erro real (em uso): passos de triagem com uma frase só de instrução, e passos isolados de "Empatia e acolhimento" (`saveData: false`) que não terminavam em pergunta. Resultado: a IA mandava uma mensagem que não conduzia o atendimento, e o passo não dizia o que salvar nem como responder na transição.

Reforço das regras de passo, com duas proibições novas:

- **Nenhum passo de coleta sem os 4 blocos**: objetivo; como perguntar (uma pergunta contextualizada); o que salvar (campo + valores + "nunca deduza"); como responder (a transição viaja JUNTO da pergunta do próximo passo, na MESMA mensagem). Faltou um bloco, o passo está incompleto e a IA improvisa.
- **Toda mensagem termina com pergunta.** Um passo de coleta nunca envia uma mensagem que não conduz.
- **Proibido passo "só de acolhimento/empatia" sem pergunta.** A empatia não é um passo: ela é a PRIMEIRA LINHA do passo que já faz a pergunta. Em vez de um passo "Empatia" seguido de um passo "Pergunta", junte os dois.
- **O fechamento é um passo de recapitulação + validação que termina em pergunta** ("Esse resumo faz sentido para você?" ou "Está certo assim?") ANTES do `TRANSFER_HUMAN`. Essa recapitulação costura para o próximo roteiro e já é a primeira fala dele.
- **Únicas exceções** a "termina com pergunta": o passo puramente de roteamento (que só transfere para outro roteiro, que então pergunta) e o passo de encerramento (`END_CONVERSATION` ou a ação final, sem pergunta).

```
Errado: um passo "Empatia e acolhimento" (saveData:false) só com "Acolha o cliente com empatia." e nada mais.
Certo:  a empatia entra como primeira linha do passo de coleta que já pergunta o dado e termina com a pergunta:
        "Faz sentido sua preocupação, [Nome]. [pergunta do dado deste passo]?"
Por quê: passo sem pergunta não conduz; a IA fica sem saber o que coletar nem como avançar.
```

## Aprendizado 2026-06-12 (escopo): triagem só, ou triagem + prospecção

ANTES de montar o roteiro de um serviço, pergunte ao assinante o ESCOPO que ele quer: **só a triagem**, ou **triagem + prospecção**. Não assuma: o escopo muda o que você gera. Pergunte logo no início da entrevista e monte conforme a resposta.

**Triagem** (mais leve): empatia e conexão. Acolhe a pessoa, cria vínculo, coleta os dados essenciais do caso e tira algumas dúvidas factuais (via FAQ), qualificando se o caso serve. NÃO vende, NÃO gera autoridade, NÃO ancora valor. Termina recapitulando, validando e encaminhando (transferência para humano ou para o próximo roteiro). É o suficiente para muitos escritórios. Molde na biblioteca: `exemplo-trabalhista-completo` (recepção + triagem por tema, sem venda).

**Prospecção** (vem DEPOIS da triagem, só quando o assinante quer vender pela IA): é a etapa comercial consultiva. Ela gera **autoridade** (como o escritório trabalha, o método), **prova social** (casos semelhantes, via FAQ "Exemplos de casos", sem prometer resultado) e faz a **ancoragem de valor na dor/prejuízo** do lead (o custo de não resolver: bloqueio, perda de prazo, dinheiro, oportunidade), conduzindo à oferta. É o SPIN consultivo. A hierarquia completa (recapitulação, problematização, implicação, necessidade, método + autoridade + prova social, oferta, objeções, fechamento, encaminhamento) está em `prospeccao.md`. Molde na biblioteca: `exemplo-tributario-completo` (triagem + consulta consultiva SPIN).

Regra: se o assinante escolher só triagem, NÃO empurre prospecção (mantenha leve, no estilo do molde trabalhista). Se escolher os dois, monte a triagem e, depois, o roteiro de prospecção/consulta que recebe a transferência (no estilo do molde tributário). A prospecção nunca vem antes da triagem.
