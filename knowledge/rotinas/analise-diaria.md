# Analise diaria do escritorio (heuristicas)

A analise diaria varre o que aconteceu nos atendimentos e entrega ao assinante um relatorio com o que precisa de atencao hoje, ja com rascunhos de mensagem prontos para os casos que pedem retorno. E uma rotina de apoio ao assinante, nao um robo que fala com o cliente: nada vai para a Za.ia nem para o cliente sem o assinante aprovar antes.

## Como a rotina le (incremental e somente leitura)

Toda a leitura acontece pelo banco da plataforma (MCP da Za.ia), nunca pela API do WhatsApp direto. A rotina e incremental: guarda um cursor local no lado do assinante (no workspace dele) marcando ate onde ja leu, e a cada run processa apenas o que mudou desde a ultima vez. Esse estado fica sempre local, nunca gravado na Za.ia. Se um run for perdido (computador desligado, dia sem rodar), nada se perde: o proximo run retoma do cursor e cobre o periodo que ficou para tras.

## As quatro analises (cada uma liga ou desliga no setup)

O assinante escolhe no setup quais das quatro analises quer receber. Cada uma e independente: se o assinante desligou uma, a rotina simplesmente nao a executa e nao a inclui no relatorio. Respeite essa escolha em todo run.

### 1. Pendencias do escritorio

Procura compromissos que o escritorio (a IA ou a equipe) prometeu ao cliente e ainda nao cumpriu, alem de tarefas que ja venceram.

- **Compromissos prometidos numa conversa**: sao falas do lado do escritorio que criam uma promessa com prazo ou entrega, e que ficaram em aberto. Como reconhecer na conversa: frases ditas pelo escritorio do tipo "te envio o contrato ainda hoje", "amanha te retorno com a resposta", "na segunda eu te mando o orcamento", "vou verificar e te aviso", "te ligo depois do almoco". O sinal e uma entrega ou retorno prometido pelo nosso lado, com um prazo (mesmo que vago), sem a acao correspondente depois. Se o prazo ja passou e nao houve o retorno, e uma pendencia.
- **Tarefas vencidas**: use `listar_tarefas` para pegar as tarefas com data de vencimento ja passada e ainda em aberto.

Para cada pendencia, a rotina prepara um rascunho de mensagem de retorno ao cliente (cumprindo o que foi prometido ou dando o novo prazo), que entra no relatorio para o assinante aprovar.

### 2. Contatos que pararam de responder

Procura conversas que esfriaram do lado do cliente. O criterio: a ultima mensagem foi do escritorio ou da IA, e o cliente esta em silencio ha N dias ou mais. O N vem da configuracao do assinante (ele define quantos dias de silencio contam como "parou de responder"), entao respeite o valor configurado, nao um numero fixo.

Para cada contato em silencio, a rotina prepara um rascunho de mensagem leve de retomada (um "passando para saber se ainda posso ajudar"), para o assinante aprovar.

### 3. Follow-up e remarketing

Procura leads frios e oportunidades de reengajar: pessoas que demonstraram interesse e nao avancaram, contatos antigos que podem ser reaquecidos, conversas que pararam antes de fechar. A ideia e nao deixar oportunidade morrer por esquecimento.

Para cada oportunidade, a rotina prepara um rascunho de mensagem de reengajamento, adequado ao momento daquele contato, para o assinante aprovar.

### 4. Lembretes e avisos

Procura prazos e compromissos que estao chegando: datas marcadas, retornos agendados, vencimentos proximos. Serve para o assinante nao ser pego de surpresa.

Para cada item, a rotina prepara, quando faz sentido, um rascunho de lembrete (para o cliente ou uma nota para o proprio assinante), para o assinante aprovar.

## A regra que vale para as quatro: rascunho sim, envio nao

Em todas as analises, a rotina gera o rascunho da mensagem pronto, escrito e revisavel, mas nao envia nada. Nenhuma mensagem vai para o cliente e nada e gravado na Za.ia (nem mensagem agendada, nem tarefa, nem etiqueta) por conta da analise diaria. A rotina prepara e enfileira propostas; o assinante revisa, ajusta e aprova; so depois de aprovado a skill escreve via MCP. O estado de idempotencia (o que ja foi processado) fica sempre local, no lado do assinante.

## A entrega: um relatorio para o assinante

O resultado de cada run e um relatorio dirigido ao assinante, organizado pelas analises que estao ligadas. Para cada caso encontrado, o relatorio traz: o que foi detectado, o contato envolvido, a evidencia (de qual conversa veio), e o rascunho de mensagem proposto. O assinante le o relatorio, escolhe o que aprovar, e so o que ele aprovar e que vira acao na plataforma.
