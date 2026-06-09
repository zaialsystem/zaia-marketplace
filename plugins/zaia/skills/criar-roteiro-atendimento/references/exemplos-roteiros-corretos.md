# Exemplos de roteiros corretos (modelos de referência)

Use estes exemplos como molde ao redigir uma correção. São trechos reais de roteiros que funcionam bem, anonimizáveis conforme o cliente. A ideia é "imitar a forma": estrutura, nível de detalhe e tom. Adapte sempre ao negócio do assinante.

Esta biblioteca é viva: vá adicionando aqui os melhores trechos que forem aparecendo.

---

## 1. Recepção com motivos de contato bem documentados

Modelo de passo de transferência (o coração da recepção). Repare que cada destino diz **quando** ir para lá, com gatilhos concretos, e há destino-padrão na dúvida.

```
Saudação: "Olá, tudo bem? Me chamo [Atendente], faço parte do time da [Empresa]
e serei responsável pelo seu atendimento."

Nome: "Qual o seu nome, por favor?"

Motivo do contato: pergunte de forma natural em que pode ajudar. Identifique o
motivo sem aprofundar nem oferecer serviços ainda; esta coleta serve apenas para
direcionar ao roteiro certo.

Transferência: com base no motivo, encaminhe para um dos roteiros. Escolha sempre
o que melhor reflete o motivo principal; na dúvida, use "Suporte".

- "Análise de viabilidade": cliente novo, assunto envolve marca (interesse em
  registrar, dúvida de como funciona, ou pergunta de valores). Encaminhar aqui NÃO
  significa que já quer contratar; atende tanto quem só tira dúvida quanto quem
  quer avançar.
- "Atualização Processual": cliente já tem processo e quer status (ex.: "como está
  meu processo", "tem novidade", "saiu alguma decisão").
- "Envio de Documentos ou Informações": cliente apenas envia documentos/arquivos,
  sem outra solicitação além do recebimento.
- "Suporte": comprovante de pagamento (Pix, transferência, boleto, print/PDF);
  motivo não se encaixa em nenhum acima; pede atendimento humano; conversa confusa
  ou repetitiva.
```

Por que funciona: a IA tem um gatilho explícito por destino e um padrão para a dúvida, então o roteamento erra pouco.

Passo de Nome, ESTRATÉGIA OPCIONAL anti-pushname (use só se a IA estiver errando o nome; normalmente não precisa). Layout da Recepção (`userComplement`):

```json
{
  "questionText": "Nome",
  "responseText": "",
  "saveData": true,
  "userComplement": "É OBRIGATÓRIO perguntar e salvar o nome do contato no primeiro contato, no campo nome_cliente. NÃO use o nome do WhatsApp (pushname) para se dirigir à pessoa. Motivo: o pushname pode vir como 'Juuu' e a IA acabaria chamando a pessoa de 'Juuu'. Pergunte de forma simples, ex.: 'Antes de tudo, como posso te chamar?' e salve a resposta. Só siga depois de ter o nome."
}
```

---

## 2. Item de triagem no padrão de quatro pontos

Cada dado vira UM passo. O passo declara, no título, que coleta APENAS aquele dado e o que NÃO perguntar ali. Estrutura enumerada e classificada de cada passo:

**REGRA DO JÁ-SABE (vale para todo passo):** se a informação já está no histórico da conversa, a IA SALVA e PULA para o próximo passo, sem perguntar e sem a mensagem de transição. Padrão da triagem: pular. Confirmar (em uma frase) só quando o objetivo do assinante pedir.

1. **Instrução** (por que precisamos): o objetivo do dado.
2. **Como perguntar**: SEMPRE uma única pergunta. As variações são escolhidas por um critério REAL e observável que a IA consegue avaliar, o histórico da conversa (ex.: "se o lead já falou em protesto", "se pediu parcelamento", "se enviou documento") ou outra condição concreta. Cada variação diz qual é o seu gatilho e todas perguntam a MESMA coisa. Não use critério vago que a IA não tem como decidir. **TODA pergunta de TODO passo precisa fazer sentido com o que já foi perguntado e com as informações já apresentadas pelo contato.** Quando houver contexto, CONTEXTUALIZE: confirme em uma linha a situação/dor específica que o contato já trouxe e só então faça a pergunta do passo. Não faça perguntas isoladas que ignoram o que ele já disse.
3. **Aguarde a resposta do usuário.**
4. **O que salvar e como**: o campo e os valores aceitos. O "Nunca deduza" vem logo ABAIXO do campo a que se refere. Dado opcional deve ser marcado como **Facultativo** (ex.: captar em silêncio um dado que o lead deu sozinho, como `tipo_pessoa`, ou um sinal de urgência para a prospecção, sem perguntar).
5. **Regra de conclusão e anti-loop**: quando o passo conclui (campo preenchido) e a proibição de frases vazias.
6. **Como responder** (transição): vem por ÚLTIMO de propósito, porque NÃO é mensagem isolada deste passo: ela é enviada JUNTO com a pergunta do PRÓXIMO passo (a IA emenda o acolhimento da resposta atual com a primeira pergunta do passo seguinte, numa só mensagem). SEMPRE com EXEMPLOS concretos por tipo de resposta (nada genérico). Acolhe sem prometer solução, sem orientar, sem calcular prazo, sem falar preço.

MODELO CANÔNICO (copie esta forma):

```
### Fase da cobrança
ITEM DE TRIAGEM (coleta APENAS a fase da cobrança). Faça SÓ esta pergunta neste passo. Não pergunte órgão, documento, prazo, valor ou objetivo aqui; cada um tem seu próprio passo depois.
REGRA DO JÁ-SABE (aplique antes de perguntar): se a fase já estiver clara no histórico da conversa (o contato já informou), SALVE fase_cobranca e PULE para o próximo passo, sem perguntar e sem a mensagem de transição. Por padrão, em triagem, pula. Se o objetivo do assinante for confirmar, confirme em UMA frase curta em vez de pular (depende do objetivo de cada assinante).

1. INSTRUÇÃO (por que precisamos): nem todo débito tem execução fiscal, citação ou prazo judicial. Saber a fase é o que define o encaminhamento.

2. COMO PERGUNTAR (sempre UMA pergunta de fase, escolha a frase conforme o contexto):
- Padrão: "[Nome], entendi. Antes de te direcionar, me diz uma coisa só: esse débito ainda está em cobrança administrativa, já está inscrito em dívida ativa, já virou execução fiscal, ou você recebeu um auto de infração/notificação?"
- Se o histórico de mensagens anteriores do contato fala em protesto: "Esse valor protestado já virou execução fiscal, ainda está em cobrança/dívida ativa, ou você não sabe?"
- Se o contato pede parcelamento: "Antes de falar em parcelamento, preciso saber a fase: esse débito está em cobrança administrativa, dívida ativa, execução fiscal, ou você não sabe?"
- Se o contato menciona bloqueio: "Esse bloqueio veio de uma execução fiscal? Você sabe se já houve citação formal no processo?"
- Se o contato envia documento antes de responder: agradeça em uma linha e MESMO ASSIM faça a pergunta de fase.

3. Aguarde a resposta do usuário.

4. O QUE SALVAR E COMO:
- campo fase_cobranca, salvar como um de: cobranca_administrativa, divida_ativa, execucao_fiscal, auto_infracao, fiscalizacao, bloqueio, parcelamento, transacao, nao_sabe. Nunca deduza a fase.
- Facultativo: caso o contato também diga espontaneamente se é PF ou PJ, salve em silêncio o campo tipo_pessoa (pf, pj), sem perguntar.
- Facultativo (sinal para a prospecção): se a fase for execucao_fiscal ou bloqueio, salve em silêncio urgencia=alta, para a prospecção depois explorar a urgência pelo método SPIN.

5. REGRA DE CONCLUSÃO E ANTI-LOOP: só conclua quando fase_cobranca estiver preenchido (valor ou nao_sabe). Toda mensagem TERMINA com a pergunta de fase. Se já perguntou e o contato não respondeu a fase, pergunte de novo de forma concreta, sem repetir o mesmo texto. PROIBIDO frases vazias e confirmações genéricas como "o primeiro passo é identificar a origem dos protestos", "depois podemos analisar as opções", "vamos seguir com isso", "já registrei", "anotei", "registrado", "perfeito", "ok".

6. COMO RESPONDER (transição: acolha sem prometer solução, sem orientar, sem calcular prazo, sem falar preço). A transição só existe se acrescentar algo REAL e específico ao que o contato disse. PROIBIDO confirmação genérica ("já registrei", "anotei", "registrado", "perfeito", "ok", "entendido"): é melhor NÃO responder nada e emendar direto na próxima pergunta do que mandar algo genérico. EXEMPLOS específicos conforme a resposta:
- Se informa execução fiscal/bloqueio: "Entendi, [Nome]. Quando já está nessa fase, o tempo conta mais, então vale organizar logo."
- Se informa cobrança administrativa/dívida ativa: "Certo, [Nome], então ainda estamos numa fase em que dá pra organizar com calma."
- Se diz que não sabe: "Sem problema, [Nome], muita gente não sabe a fase de cabeça, a gente identifica isso na sequência."
- Se está aflito ou perdido: "Imagino a preocupação, [Nome]. A gente organiza isso juntos, passo a passo."
- Se não houver acolhimento específico e verdadeiro a fazer: não escreva transição, vá direto para a próxima pergunta.
-> Quando existir, a transição é enviada JUNTO com a pergunta do PRÓXIMO passo, numa só mensagem (a IA emenda o acolhimento da resposta atual com a primeira pergunta do passo seguinte). Nunca como mensagem solta.
```

Observação de ordem: o "Como responder" (transição) vem por último de propósito, porque ele NÃO é uma mensagem isolada deste passo, ele cola na pergunta do próximo passo. Pense nele como a ponte entre um passo e o seguinte.

Outros exemplos de passo único (mesma estrutura):

```
### Ano do acidente

- Instrução (por que precisamos): o caso precisa ter ocorrido, no máximo, nos
  últimos 5 anos. Acima disso, há risco de prescrição.
- Como perguntar: "[Nome], o senhor[a] lembra em que ano esse acidente aconteceu?"
- Como responder (empatia/transição, conforme a resposta):
  - Se foi recente: "Nossa, foi bem recente, [Nome]! Ainda bem que já procurou a
    gente logo."
  - Se faz mais tempo: "Faz um tempo então, [Nome], mas fica tranquilo[a], vamos
    analisar a sua situação com cuidado."
- O que salvar e como: campo `ano_acidente`, no formato AAAA ou a expressão que a
  pessoa usar ("este ano"). Não deduza a data. Se a resposta for vaga ("faz uns
  anos"), esclareça apenas se foi há mais ou há menos de 5 anos, sem exigir o ano
  exato. Se já vier o ano, salve e siga.
```

Modelo de campo com valores fixos e regra de "pular o item":

```
### Interesse em pedir a rescisão indireta

- Instrução: se a pessoa ainda está na empresa, não foi demitida, não quer sair e
  não quer pedir rescisão indireta, não há o que fazer agora. Por isso perguntamos.
- Quando perguntar: somente se o vínculo for `ativo` ou `afastado_inss`. Se o
  cliente já saiu, NÃO faça esta pergunta: pule e salve `nao_se_aplica`.
- Como perguntar: "Como o senhor[a] ainda está na empresa, me diz: tem interesse em
  sair de lá, em pedir o desligamento por culpa da empresa, a chamada rescisão
  indireta?"
- Como responder: se quer sair: "Entendi, [Nome]. É uma decisão importante, e o
  senhor[a] não precisa decidir sozinho[a]." Se não quer: "Tudo bem, [Nome]. Cada
  um sabe o momento certo, e a gente respeita isso."
- O que salvar e como: `interesse_rescisao_indireta` com valores `sim` | `nao` |
  `nao_se_aplica`. Salve só o que o cliente disse.
```

Regras de conduta que valem para todo passo de triagem (coloque no topo do passo):

```
Trate o usuário por senhor[a]; uma pergunta por vez; antes de cada nova pergunta,
faça um comentário empático curto sobre a resposta anterior; sem emojis, sem
diminutivos ("tempinho", "direitinho"), sem travessão; não se reapresente (já houve
recepção); se o usuário já respondeu algo espontaneamente, não repita a pergunta.
Nunca deduza: salve apenas o que o usuário disser.
```

---

### Exemplo, passo de documento (adaptado ao contexto já coletado)

```
### Documento da cobrança
ITEM DE TRIAGEM (coleta APENAS se há documento). Não pergunte fase, órgão, valor ou prazo aqui (já têm seus passos).
REGRA DO JÁ-SABE: se o contato já enviou um documento, SALVE e pule, sem perguntar.

1. INSTRUÇÃO (por que precisamos): o documento permite extrair fatos objetivos depois (tipo, datas, número), sem interpretar nada agora.
2. COMO PERGUNTAR (uma pergunta, conectada ao que já foi dito; não use pergunta genérica e solta):
- Padrão (já sabe o órgão): "Sobre essa cobrança da [órgão] que você mencionou, você tem algum documento dela em mãos? Pode ser notificação, auto de infração, CDA ou relatório de débitos. Se tiver, me envia; se não tiver, é só dizer."
- Se ainda não houver órgão definido: "Você tem em mãos algum documento dessa cobrança (notificação, auto de infração, CDA, execução fiscal ou relatório de débitos)? Se tiver, me envia; se não, me avise."
3. Aguarde a resposta do usuário.
4. O QUE SALVAR E COMO:
- campo tem_documento, salvar como: sim, nao. Nunca deduza.
- Facultativo: se enviar o documento, salve em silêncio o anexo/observação, sem comentar conteúdo.
5. CONCLUSÃO E ANTI-LOOP: conclua quando tem_documento estiver preenchido. PROIBIDO confirmação genérica ("já registrei", "anotei").
6. COMO RESPONDER (transição, só se específica; senão vá direto à próxima pergunta): exemplo, se enviou: "Recebi aqui, [Nome], obrigada." (e emenda a próxima pergunta).
```

### Exemplo, passo de objetivo (com CONTEXTUALIZE)

```
### Objetivo principal
ITEM DE TRIAGEM (coleta APENAS o objetivo principal). Não pergunte fase, órgão, documento ou valor aqui (já têm seus passos).
REGRA DO JÁ-SABE: se o objetivo já está claro no histórico, SALVE e pule.

1. INSTRUÇÃO (por que precisamos): o objetivo orienta a abordagem da consulta (regularizar, reduzir, parcelar, defender, entender risco).
2. COMO PERGUNTAR (UMA pergunta de objetivo). CONTEXTUALIZE com base no que o contato já apresentou:
- Padrão (sem dor específica anterior): "[Nome], e hoje qual é o seu objetivo principal: regularizar para destravar algo (como uma venda ou certidão), reduzir o valor, parcelar, se defender, ou só entender o risco?"
- Se o contato mencionou receio de bloqueio: confirme o receio e pergunte se há mais algo que o aflige.
- Se o contato disse que a venda de um imóvel está travada: confirme o travamento e pergunte o objetivo. Ex.: "Você havia mencionado que esses débitos na Prefeitura estão travando a venda do imóvel, certo? Há mais algum objetivo que te levou a procurar o escritório, como, por exemplo, ver a possibilidade de parcelamento desses valores?"
3. Aguarde a resposta do usuário.
4. O QUE SALVAR E COMO:
- campo objetivo_principal, salvar como: regularizar, reduzir, parcelar, defender, entender_risco, outro. Nunca deduza.
5. CONCLUSÃO E ANTI-LOOP: conclua quando objetivo_principal estiver preenchido. PROIBIDO confirmação genérica ("já registrei", "anotei").
6. COMO RESPONDER (transição, só se específica; senão vá direto à próxima etapa do fluxo).
```

## 3. Suporte com casos e mensagens prontas

```
#comprovante de pagamento
Se o cliente enviar comprovante (Pix, transferência, boleto, print/foto/PDF),
apenas agradeça, informe que vai comunicar a equipe e encerre. Não faça mais
perguntas nem peça outros dados.
"Recebi seu comprovante, muito obrigada! Vou comunicar a equipe sobre o pagamento.
Qualquer coisa, estamos à disposição."

#envio de documentos
"Já recebi seus documentos, vou informar a equipe que você os enviou, obrigada.
Eles retornarão o mais breve possível."

#demais casos (ajuste por horário)
Dentro do horário: "Vou transferir seu atendimento para o responsável. Ele vai te
retornar o mais breve possível. Um momento, por favor."
Fora do horário: "Vou encaminhar seu contato para o responsável. Como estamos fora
do horário de atendimento, ele retornará na [próximo dia/horário de atendimento]."
```

Por que funciona: cada caso previsível tem a mensagem pronta, o retorno nunca é prometido como imediato, e há variação por horário.

---

## 4. Prospecção / quebra de objeções

Modelo de objeção com resposta pronta (não promete resultado, não dá valor em reais, não garante gratuidade):

```
#"Por que 30%? Não dá para ser menos?"
"Entendo a pergunta, [Nome]. Esse percentual é cobrado só no êxito, então o
senhor[a] não paga nada agora e o escritório só recebe se ganhar. Nesse modelo, é o
escritório que assume todo o risco e o custo do trabalho do começo ao fim (petição,
audiências, recursos, acompanhamento) e só é remunerado no final, sobre o que o
senhor[a] efetivamente receber."

#"Quanto eu vou receber?"
"Essa conta só a advogada consegue fazer olhando os documentos, [Nome]. Qualquer
número agora seria um chute, e por isso a gente não trabalha com promessa de valor.
Assim que o caso for analisado, ela te explica os cenários com clareza."

#"Vou pensar / depois eu vejo" (indecisão)
Reaproveite uma dor que o cliente trouxe na triagem. "Claro, [Nome], é uma decisão
sua. Só queria retomar uma coisa que o senhor[a] me contou: [dor da triagem].
Quanto antes começarmos, antes o senhor[a] busca o que é seu por direito. Posso
esclarecer mais alguma dúvida?"
```

Para exemplos de casos de sucesso, prefira a categoria de FAQ "Exemplos de casos" e instrua o roteiro a consultá-la (ver `integracao-roteiro-faq.md`), em vez de fixar exemplos no roteiro.

---

## 5. Pares "errado x certo" (a vacina mais eficaz)

Transforme cada erro real em um par destes dentro do roteiro. Exemplos reais:

```
### Valoração de decisão (proibido opinar sobre o mérito)
Cliente relata que perdeu um voto no processo.
❌ Errado: "É importante manter essa postura técnica e fundamentada..."
✅ Certo: "[Nome], vou passar para a Dra. [Nome] o que você informou e ela retorna
   assim que possível."
Por quê: o atendente acolhe, registra e transfere; não opina sobre o mérito.

### Ser proativa em oferecer ações (proibido)
Cliente: "não sei o que responder no e-mail."
❌ Errado: "Posso te ajudar a elaborar uma resposta" / "Quer que eu prepare um
   rascunho?"
✅ Certo: "Entendi, [Nome]. Vou repassar para a Dra. [Nome] e ela retorna assim que
   possível com a orientação."
Por quê: o atendente não executa ações; só registra e transfere.

### Despedidas escaladas e redundantes
Cliente: "Agradeço."
❌ Errado: "Fico à disposição, [Nome]. Qualquer novidade entro em contato
   imediatamente. Tenha um ótimo dia!"
✅ Certo: "Eu que agradeço, [Nome]. Boa tarde!"
Cliente depois: "Um ótimo dia pra você também!"
✅ Certo: [não responder; a conversa já fechou]

### Duas mensagens dizendo a mesma coisa
❌ Errado:
  Msg 1: "Continuamos acompanhando e assim que tiver novidade eu te aviso."
  Msg 2: "Fique tranquila que estamos em cima disso."
✅ Certo (uma só): "Combinado, [Nome]. Qualquer novidade eu te aviso."

### Não cumprimentar na primeira mensagem
❌ Errado: "Claro, vou conferir no seu processo."
✅ Certo: "Bom dia, [Nome]! Vou conferir no seu processo e já te repasso."
```

---

## 6. Recepção padrão do sistema (layout userComplement)

Este é o layout REAL dos roteiros padrão (Recepção, Suporte, Agendamento): a instrução do agente vai em `userComplement` e o `responseText` fica vazio. Copie esta forma ao recriar ou atualizar a Recepção ou o Suporte (não use `responseText` para a instrução nesses três).

```json
{
  "flows": [
    {
      "name": "Recepção",
      "description": "Este roteiro é o ponto de entrada de todo atendimento. Ele saúda o cliente, coleta seu nome e o motivo do contato, e identifica qual roteiro deve receber a transferência com base no motivo informado.",
      "questions": [
        {
          "questionText": "Saudação",
          "responseText": "",
          "saveData": false,
          "userComplement": "Instrução ao agente: cumprimente o lead APENAS na primeira mensagem do contato. \n#NÃO TEM o NOME do contato?\n-> Use quando o usuário apenas enviou um cumprimento, sem o motivo do contato e o nome.\n1. '[Bom dia/Boa tarde/Boa noite]! Que bom te receber por aqui. Sou a Isabela, do Pimentel Advogados. Antes de tudo, como posso te chamar?' Salve o campo nome_cliente (texto)\n2. 'Prazer, [Nome]. E como podemos te auxiliar?'\n-> Identifique o motivo no passo 3 e já transfira para o departamento correto.\n\n#TEM o NOME do contato, mas NÃO TEM o motivo?\n'[Bom dia/Boa tarde/Boa noite], [Nome]! Que bom te receber por aqui. Sou a Isabela, do Pimentel Advogados. Sobre [breve resumo do motivo do contato - mais palavras chaves], podemos te auxiliar sim! [ou alguma mensagem com o contexto correto], mas antes de tudo, como posso te chamar?'\n\n#TEM o MOTIVO do contato, mas NÃO TEM NOME?\n'[Bom dia/Boa tarde/Boa noite]! Que bom te receber por aqui. Sou a Isabela, do Pimentel Advogados. Em que podemos te auxiliar?'\n\n#tem NOME e MOTIVO do contato?\n'[Bom dia/Boa tarde/Boa noite], [nOME]! Que bom te receber por aqui. Sou a Isabela, do Pimentel Advogados. Sobre [breve resumo do motivo do contato - mais palavras chaves], podemos te auxiliar sim!' ou alguma mensagem com o contexto correto.\n-> Identifique o motivo no passo 3 e já transfira para o departamento correto.\n"
        },
        {
          "questionText": "Nome",
          "responseText": "",
          "saveData": false,
          "userComplement": ""
        },
        {
          "questionText": "Motivo do contato",
          "responseText": "",
          "saveData": false,
          "userComplement": ""
        },
        {
          "questionText": "Transferência",
          "responseText": "",
          "saveData": false,
          "userComplement": ""
        }
      ]
    }
  ]
}
```

## 7. Recepção: passo de classificação claro (uma regra por tema)

Escreva a classificação da Recepção com UMA frase por tema, no padrão fixo, sem parágrafo corrido. Cada frase: sinais claros, o tema, e o slug de destino.

```
Regras de classificação e transferência (uma por tema):

Caso o contato informe que tem dívida, débito, cobrança, execução fiscal, auto de infração, dívida ativa, protesto ou penhora, classifique o motivo como Débitos Tributários e transfira para dbito_tributrio__qualificao_inicial. (Esta regra tem PRIORIDADE: havendo qualquer sinal de débito, é sempre este tema, mesmo que cite outro junto.)

Caso o contato informe que tem clínica, é médico, faz parte de sociedade médica ou quer equiparação hospitalar, classifique o motivo como Clínica Médica e transfira para clnica_mdica__qualificao_inicial.

Caso o contato fale em Reforma Tributária, IBS, CBS, IVA, split payment ou impacto em 2026/2027/2033, classifique como Reforma Tributária e transfira para reforma_tributria__qualificao_inicial.

Caso o contato fale em holding, administração de imóveis próprios, integralização, sucessão, ITCMD ou ITBI, classifique como Holding Patrimonial e transfira para holding_... .

Caso o contato fale em imposto pago a maior, restituição, compensação ou tese de crédito SEM dívida junto, classifique como Recuperação de Créditos e transfira para recuperao_de_crditos__qualificao_inicial.

Caso o contato fale em incentivo/benefício fiscal de empresa (Lei do Bem, Reintegra, regime especial), classifique como Benefícios Fiscais e transfira para benefcios_fiscais__qualificao_inicial.

Caso nada acima se aplique e o lead seja vago, faça UMA pergunta de desambiguação. Se ainda não encaixar, transfira para a Triagem Consultiva (triagem_consultiva). Se for fora do escopo tributário, encerre com cordialidade.
```

Regra de desambiguação (quando um sinal cabe em dois temas): faça uma única pergunta antes de classificar. Ex.: "Sua dúvida sobre imóveis é por causa da Reforma Tributária ou de estruturação patrimonial/holding?"

Antes de entregar, confira que nenhum sinal aparece em dois temas sem regra de prioridade, que há destino-padrão, e que todos os slugs existem.

## Como adicionar novos exemplos aqui

Sempre que um diagnóstico gerar uma boa correção, salve o trecho corrigido nesta biblioteca, no formato dos modelos acima. Identifique a qual tipo de roteiro pertence (recepção, triagem, suporte, prospecção) e, quando for um par errado x certo, inclua a linha "Por quê". Assim a skill fica mais inteligente a cada uso.
