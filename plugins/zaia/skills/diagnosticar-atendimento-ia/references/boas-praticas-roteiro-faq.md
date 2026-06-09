# Boas práticas: escrever roteiro e FAQ que a IA obedece bem

Use este arquivo no Passo 7, quando for redigir a correção. O objetivo é escrever a instrução de um jeito que torne o comportamento certo a resposta mais provável da IA.

## Roteiro ou FAQ? Onde colocar a correção

| Pergunte | Se sim | Vai para |
|---|---|---|
| É sobre como a IA deve se comportar, conduzir, transferir, o tom, o que pode/não pode dizer ou oferecer? | conduta | **Roteiro** |
| É uma informação factual que o cliente pergunta e a resposta é sempre a mesma (preço, prazo, documentos, horário, como funciona)? | dado | **FAQ** |
| A resposta muda conforme o momento/situação da conversa? | depende | **Roteiro** |
| A resposta é sempre idêntica, independentemente do contexto? | fixa | **FAQ** |

Regra de bolso: **FAQ guarda o dado, roteiro guarda a conduta.** Preço da análise é FAQ. "Como apresentar o preço sem pressionar o cliente" é roteiro.

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
❌ Errado: "[a resposta indesejada que de fato apareceu]"
✅ Certo: "[a resposta que você queria]"
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

## Padrões por tipo de roteiro

Cada tipo de roteiro tem uma estrutura que costuma funcionar. Use estes padrões ao redigir ou corrigir. Há exemplos reais prontos em `exemplos-roteiros-corretos.md`.

### Recepção

A recepção recebe o cliente, descobre o motivo do contato e encaminha para o roteiro certo. Ela quase não resolve nada sozinha: o trabalho dela é classificar bem. Erro de recepção contamina todo o resto (ver causa 9 em `causas-raiz.md`).

Documente sempre:

- **Saudação e identificação**: como a IA se apresenta (nome do atendente, empresa/escritório), com saudação conforme o horário quando fizer sentido.
- **Coleta do nome**: pergunta única e objetiva.
- **A recepção salva dados**: salve nome, motivo e qualquer dado que o lead já tenha dado (mesmo fora de ordem), com `saveData: true` e a `dataKey` certa. Isso evita que os roteiros seguintes repitam perguntas (a regra do já-sabe reaproveita esses dados).
- **Estratégia OPCIONAL para IA que erra o nome (usa o pushname)**: normalmente a IA acerta o nome, então NÃO force por padrão. Use só se a IA estiver errando: aí torne o passo de nome obrigatório (perguntar e salvar `nome_cliente`, sem usar o pushname) e explique o motivo no passo. Exemplo de motivo: o pushname pode vir como "Juuu" e a IA acaba chamando a pessoa de "Juuu".
- **Motivo do contato**: pergunta aberta para descobrir o que a pessoa quer, sem aprofundar nem oferecer serviço ainda.
- **Motivos de contato bem documentados (o ponto mais importante)**: liste cada destino possível e, para cada um, deixe claro quando mandar para lá, com exemplos de frases reais do cliente que disparam aquele destino. Quanto mais explícito o gatilho, menos a IA erra o encaminhamento.
- **Destino-padrão na dúvida**: sempre defina para onde vai o que não se encaixa (em geral, Suporte / humano).
- **Uma regra explícita por tema, sem contradição**: escreva a classificação como uma frase por tema, no padrão "Caso o contato informe X, classifique como Y e transfira para Z". Nunca um parágrafo corrido com vários temas misturados. Garanta que nenhum sinal aponte para dois temas sem regra de prioridade ou pergunta de desambiguação, e que todos os slugs de destino existam.

Exemplo do nível de detalhe esperado para os motivos:

```
- "Análise de viabilidade": cliente novo, assunto envolve registrar marca, dúvidas
  de como funciona, ou pergunta de valores. Encaminhar aqui NÃO significa que já
  quer contratar; serve tanto para quem só tira dúvida quanto para quem quer avançar.
- "Atualização Processual": cliente já tem processo e quer status (ex.: "como está
  meu processo", "tem novidade", "saiu decisão").
- "Envio de Documentos": cliente só manda documentos/arquivos, sem outra solicitação.
- "Suporte": comprovante de pagamento; pede humano; motivo não se encaixa; conversa
  confusa ou repetitiva.
```

### Triagem

A triagem aprofunda o caso depois da recepção: acolhe, coleta os dados essenciais e qualifica (o caso serve ou não). É o roteiro que mais coleta dados, então a disciplina de coleta é tudo.

**Regra número um, UMA informação por passo.** Cada passo coleta UM único dado e faz UMA única pergunta, salvando UM único campo. A plataforma só controla o avanço passo a passo; se você juntar vários dados numa instrução só, a IA pula algum (não há controle dentro do passo). Então cada dado (fase, órgão, tipo de pessoa, valor) é um passo separado. As variações da pergunta são só formas diferentes de perguntar a MESMA coisa conforme o contexto.

**O passo declara o que coleta.** Comece o passo dizendo que ele coleta APENAS aquele dado e o que NÃO perguntar ali (ex.: "Coleta APENAS a fase da cobrança. Não pergunte órgão, documento, prazo ou valor aqui; cada um tem seu passo depois").

**Regra do já-sabe.** Antes de perguntar, a IA verifica se o dado já está no histórico. Se já estiver, SALVA e PULA para o próximo passo, sem perguntar e sem mensagem de transição. Em triagem o padrão é pular; confirmar (uma frase) só quando o objetivo do assinante pedir.

**Estrutura obrigatória: cada passo precisa documentar os quatro pontos abaixo.** Sem os quatro, a IA improvisa a pergunta, aceita resposta vaga ou salva dado errado.

1. **Instrução** (o que é esse item e por que ele importa): explique o objetivo do dado, para a IA entender o que está fazendo (ex.: "o caso precisa ter ocorrido nos últimos 5 anos, acima disso há risco de prescrição").
2. **Como perguntar**: a frase exata (ou um modelo) que a IA deve usar, no tom do atendimento. Uma pergunta por vez. Quando houver variações da pergunta, cada uma precisa ter um gatilho REAL e observável que a IA consiga avaliar (o histórico da conversa, ex.: "se o lead falou em protesto", "se pediu parcelamento", "se enviou documento"; ou outra condição concreta). Indique o gatilho de cada variação e mantenha todas perguntando a MESMA coisa. Evite critério vago que a IA não tem como decidir. Conecte a pergunta ao que já foi coletado nos passos anteriores, não use boilerplate isolado. Ex.: se a IA já perguntou o tipo e o órgão, o passo de documento deve referenciar isso: "Sobre essa cobrança da [órgão] que você mencionou, você tem algum documento dela em mãos?", em vez de uma pergunta de documento genérica e desconectada.

**CONTEXTUALIZE em todo passo (regra geral).** Toda pergunta precisa fazer sentido com o que já foi perguntado e informado. Quando o contato já trouxe uma dor/situação, confirme-a em uma linha e emende a pergunta do passo. Exemplos: (a) o contato mencionou receio de bloqueio, confirme o receio e pergunte se há mais algo que o aflige; (b) o contato disse que a venda de um imóvel está travada, confirme o travamento e pergunte o objetivo. Ex.: "Você havia mencionado que esses débitos na Prefeitura estão travando a venda do imóvel, certo? Há mais algum objetivo que te levou a procurar o escritório, como, por exemplo, ver a possibilidade de parcelamento desses valores?"
3. **Como responder** (transição): SEMPRE com EXEMPLOS concretos de fala por tipo de resposta. PROIBIDO confirmação genérica ("já registrei", "anotei", "registrado", "perfeito", "ok"): é melhor NÃO responder nada e ir direto para a próxima pergunta do que mandar algo genérico. A transição só existe se acrescentar algo real e específico. É o acolhimento da resposta do cliente, e ele é enviado **junto com a pergunta do próximo passo**, numa só mensagem, nunca como mensagem solta. Acolhe sem prometer solução, sem orientar, sem calcular prazo, sem falar preço. Sem exemplos, este ponto está incompleto. (Pense neste ponto como a ponte entre um passo e o seguinte; por isso, no modelo canônico, ele aparece por último.)
4. **O que salvar e como**: o nome do campo e os valores aceitos (ex.: `sim` | `nao` | `nao_se_aplica`). O aviso **Nunca deduza, salve só o que o cliente disse** vem logo ABAIXO do campo a que se refere, não no fim. Diga o que salvar quando o item não se aplica (`nao_se_aplica`) e quando pular o item. Dado opcional deve ser marcado como **Facultativo** (ex.: captar em silêncio um dado que o lead deu sozinho, como `tipo_pessoa`, sem perguntar).

5. **Regra de conclusão e anti-loop**: diga quando o passo conclui (campo preenchido, com valor ou `nao_sabe`), que toda mensagem termina com a pergunta daquele dado, e proíba frases vazias ("o primeiro passo é identificar...", "depois podemos analisar...", "vamos seguir com isso").

Esse formato (5 pontos enumerados, um dado por passo) é o padrão-ouro. Ao corrigir uma triagem, se faltar qualquer ponto em algum passo, ou se um passo juntar mais de um dado, esse é quase sempre o ponto de origem do erro. Modelo pronto em `exemplos-roteiros-corretos.md`.

### Prospecção

A prospecção converte o lead qualificado em cliente: apresenta o escritório, faz a oferta, quebra objeções e busca o aceite. **Use o mesmo formato de 4 pontos da triagem, agora por ETAPA** (apresentação, oferta, cada objeção, fechamento), não por dado coletado:

Dois pilares da prospecção: (1) **costura com as dores do lead**, use as dores e dados que a triagem salvou como argumento de ancoragem, para a prospecção soar como continuação do que o lead disse; (2) **quebra de objeções com passos próprios**, e a skill ENTREVISTA o assinante sobre como ele responde a cada objeção ("está caro", "não consigo pagar", "vou pensar", "já tenho advogado/contador", "depois eu vejo", "quanto vou reduzir/receber?"). Cada objeção vira uma etapa no formato de 4 pontos, no tom dos bons atendimentos, sem prometer resultado e sem cravar valor que depende de análise.

1. **Instrução**: o objetivo daquela etapa e por que ela existe (ex.: "ancorar autoridade antes da oferta", "quebrar a objeção de preço sem dar valor em reais").
2. **Como perguntar / como dizer**: a fala ou a oferta exata daquela etapa, uma ideia por mensagem.
3. **Como responder**: SEMPRE com EXEMPLOS de fala. Como conduzir conforme a reação do cliente, com variações (aceitou, hesitou, pediu desconto, quer pensar). Reaproveite as dores trazidas na triagem.
4. **O que salvar e como**: o estado daquela etapa (ex.: `proposta_apresentada = sim`, objeção registrada, `aceite = sim/nao`), no formato e valores definidos.

Regras fortes da prospecção: nunca prometer resultado, não cravar valores que dependem de análise (consultar a FAQ quando o tema for factual) e, para exemplos de casos, consultar a categoria de FAQ "Exemplos de casos" em vez de fixar exemplos no roteiro (ver `integracao-roteiro-faq.md`).

**Hierarquia da prospecção (etapas em ordem):** 1 recapitulação/validação, 2 problematização (com a dor do lead), 3 implicação (SPIN), 4 necessidade de solução, 5 método + autoridade + prova social, 6 oferta, 7 quebra de objeções, 8 fechamento, 9 encaminhamento. Os passos internos de cada etapa também são ordenados e numerados. A quebra de objeção segue a sub-ordem: acolher, validar, reancorar na dor, responder no tom do assinante, reperguntar.

**Variações no passo final (prospecção/fechamento) trazem a referência principal + a variação, não só a variação.** Cada variação por tipo de resposta deve conter a mensagem principal daquela etapa e acrescentar o trecho específico, para a IA nunca enviar só o acréscimo solto, sem o contexto principal. **Tom não superficial:** as mensagens de prospecção devem ter a profundidade dos bons atendimentos reais do assinante (aprenda o tom lendo as conversas que ele indicar via MCP).

### Suporte

O suporte é o fim de linha: encerra o atendimento automático e passa para humano. O risco aqui é prometer demais ou tratar mal o encerramento. Documente sempre:

- **Casos bem definidos com a mensagem pronta**: cada situação previsível (comprovante de pagamento, envio de documentos, pedido de falar com a equipe, caso genérico) com a frase exata de resposta.
- **Não prometer retorno imediato**: deixar explícito que o retorno é "em breve", não "agora". Nunca dizer que a pessoa responsável "já está ciente".
- **Ajuste por horário**: mensagem diferente dentro e fora do horário de atendimento (no fora do horário, informar quando será o retorno com base nos dias/horários cadastrados).
- **Não pedir dados fora do escopo nem fazer mais perguntas** depois de já ter decidido transferir.

## Lembrete final

Nenhuma instrução garante 100%, porque a IA é probabilística. Mas instrução concreta + exemplo real + destino-padrão + FAQ atualizada empurram a probabilidade do acerto para perto do teto. Ao entregar a correção, sempre explique ao assinante **por que** aquele ajuste torna o erro menos provável: isso ensina ele a pescar, não só a receber o peixe.
