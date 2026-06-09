# Como gerar uma peca a partir do atendimento

Esta e a metodologia para transformar um atendimento ja concluido em um documento pronto para o assinante revisar. Ela vale para qualquer tipo de peca: o assinante decide o que faz (procuracao, contrato, substabelecimento, parecer, declaracao, ou outro documento), e o mesmo metodo se aplica a todos. A regra que nunca muda: nada e enviado ao cliente e nada e gravado na Za.ia sem o assinante aprovar antes.

## Principio que rege tudo: so use o que foi dito

A peca se monta com o que o cliente realmente informou no atendimento, nunca com suposicao. Se um dado nao apareceu na conversa, ele nao entra na peca por deducao. Faltou, sinaliza para o assinante (ver "Quando falta um dado"). Inventar dado em documento juridico e erro grave, entao a metodologia inteira parte da leitura fiel do que foi registrado.

## Passo 1: extrair os dados do atendimento (somente leitura, via banco da plataforma)

Toda a leitura acontece pelo banco de dados da plataforma (MCP da Za.ia), nunca pela API do WhatsApp direto. As ferramentas de leitura usadas aqui:

- `ler_atendimento_ia`: traz o atendimento conduzido pela IA, com os dados que a IA coletou e organizou durante a conversa (o resumo estruturado do caso, os campos preenchidos passo a passo).
- `ler_conversa`: traz a conversa completa, mensagem por mensagem, para conferir no proprio texto o que o cliente disse, com as palavras dele.

Use as duas em conjunto: o atendimento da IA fornece os dados ja organizados, e a conversa serve para confirmar e para pescar detalhes que o cliente deu por escrito mas que talvez nao tenham virado um campo estruturado. Extraia apenas o que esta ali. Nome, qualificacao, fatos, datas, valores, pedidos: tudo sai do que foi efetivamente dito, na forma como foi dito.

Ao terminar a extracao, monte uma lista clara de "o que eu tenho" (cada dado com a sua origem na conversa) e "o que falta" para a peca que o assinante quer.

## Passo 2: usar o modelo do proprio assinante como base

A peca deve sair com a cara do escritorio. No setup, o assinante envia os documentos modelo dele (os que ele ja usa no dia a dia). A geracao funciona por copia do modelo: pega o documento que o assinante enviou para aquele tipo de peca e preenche os campos com os dados extraidos no passo 1, mantendo o texto, a estrutura, o cabecalho e o estilo que ja sao dele.

- Se existe modelo do assinante para aquele tipo de peca: use esse modelo como base e so preencha os espacos com os dados do caso. Nao reescreva o que e padrao do escritorio.
- Se nao existe modelo para aquele tipo: use uma estrutura generica de fallback (uma versao neutra e correta daquele documento), deixando claro ao assinante que foi usado um modelo padrao porque ele ainda nao enviou o dele, e que vale a pena cadastrar o modelo proprio para as proximas.

## Quando falta um dado: sinalizar, nunca preencher por conta propria

Se a peca precisa de um dado que nao apareceu no atendimento, a peca nao e "completada na marra". O caminho e:

1. Deixar o ponto marcado no documento de forma visivel (um campo em branco sinalizado, por exemplo entre colchetes), para o assinante ver na hora exatamente onde falta.
2. Listar para o assinante, em texto, o que ficou faltando e por que o documento precisa daquilo.
3. Quando fizer sentido, sugerir ao assinante coletar esse dado no atendimento (por exemplo, uma informacao que o roteiro poderia passar a perguntar, ou um documento que o cliente ainda precisa enviar). Essa sugestao e so uma recomendacao para o assinante, nada e pedido ao cliente automaticamente.

O documento sai assim mesmo, com as lacunas sinalizadas, para o assinante decidir se completa, se pede ao cliente, ou se ajusta. Melhor uma peca honesta com os buracos a mostra do que uma peca preenchida com chute.

## Passo 3: entregar em .docx para revisao e assinatura

A saida e um arquivo .docx, pronto para o assinante abrir, conferir e ajustar. Ao entregar, deixe sempre explicito:

- O assinante deve revisar e assinar a peca. A responsabilidade pelo conteudo e dele, e a peca e um rascunho que ele valida.
- A peca NUNCA e enviada ao cliente pela rotina. Quem decide enviar, e quando, e o assinante.
- Nada e gravado na Za.ia (nem mensagem, nem tarefa, nem nada) por causa da geracao da peca, a menos que o assinante aprove explicitamente alguma acao depois.

## Resumo do metodo

1. Le o atendimento pelo banco da plataforma (`ler_atendimento_ia` e `ler_conversa`), so leitura, sem deduzir o que nao foi dito.
2. Usa o modelo do proprio assinante como base (copia e preenche); se nao houver, usa estrutura generica de fallback e avisa.
3. Nao inventa dado faltante: sinaliza no documento e lista para o assinante o que falta, sugerindo coletar no atendimento quando couber.
4. Entrega .docx para o assinante revisar e assinar, sem enviar ao cliente e sem gravar nada na Za.ia sem aprovacao.
