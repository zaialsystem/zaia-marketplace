# Integração roteiro e FAQ (o ponto central)

Roteiro e FAQ não competem, eles trabalham juntos. Entender essa divisão (e a ponte entre os dois) é o que separa um atendimento mediano de um excelente. Este princípio vale para diagnosticar, criar roteiro e criar FAQ.

## A divisão de trabalho

- **Roteiro = condução.** O fluxo do atendimento: o que perguntar, em que ordem, como acolher, quando transferir, o que pode ou não dizer, qual dado salvar. Muda conforme o momento da conversa.
- **FAQ = conhecimento consultável.** A informação que a IA busca para responder dúvidas: preços, prazos, documentos, como funciona, objeções, e também exemplos de casos. É sempre a mesma informação, independente do passo do roteiro.

Durante o atendimento, quando o cliente faz uma pergunta fora do fluxo, a IA consulta a FAQ para responder sem perder o fio do roteiro. O roteiro mantém o rumo; a FAQ alimenta as respostas pontuais.

## A ponte: o roteiro deve pedir consulta explícita ao FAQ nos temas importantes

Este é o ponto que mais se perde. Não basta a FAQ existir. Em temas importantes, **o passo do roteiro precisa instruir a IA, em texto, a consultar a FAQ** antes de responder. Sem esse pedido explícito, a IA pode improvisar a resposta em vez de usar o conhecimento curado, e aí erra preço, prazo ou exemplo.

Forma recomendada de escrever a instrução no passo do roteiro:

```
Quando o cliente perguntar sobre [tema], consulte a FAQ da categoria "[categoria]"
e responda com base no conteúdo encontrado. Não invente dados: se não houver FAQ
que cubra a pergunta, acolha e transfira para humano.
```

Sinais de que a ponte está faltando (úteis no diagnóstico):

- O roteiro fala de um tema factual (valores, prazos, exemplos) mas não manda consultar a FAQ.
- A FAQ existe e está correta, mas a IA respondeu outra coisa: provável ausência do pedido explícito de consulta no passo.
- O roteiro tem dados fixos embutidos (preço, exemplo de caso) que deveriam viver na FAQ.

## Padrão de ouro: exemplos de casos no FAQ, escolhidos pela IA

Caso de uso clássico, a prospecção. Em vez de listar no roteiro exemplos genéricos de casos que o escritório já atuou (fixos, iguais para todo cliente), faça assim:

1. **Na FAQ**: crie uma categoria "Exemplos de casos" (ou "Casos de sucesso"), com uma entrada por caso. Cada entrada descreve o perfil do caso e o desfecho, em linguagem que a IA possa casar com a situação do cliente.
2. **No roteiro (passo de prospecção / quebra de objeção)**: instrua a IA a consultar essa categoria e **escolher o exemplo mais parecido** com o que o cliente trouxe na triagem, usando-o para gerar confiança.

Exemplo de instrução no passo:

```
Para reforçar a confiança, consulte a FAQ da categoria "Exemplos de casos" e
selecione o exemplo cujo perfil mais se aproxima do que o cliente relatou (mesma
área, dor parecida). Apresente esse exemplo de forma natural, sem prometer o mesmo
resultado. Se nenhum exemplo for próximo, não force: siga sem exemplo.
```

Por que isso é melhor: o exemplo passa a ser **relevante para cada cliente** (a IA escolhe o que casa com o caso), a manutenção fica centralizada (novo caso de sucesso, é só adicionar uma FAQ, sem mexer no roteiro), e o roteiro fica enxuto.

## Regra de bolso ao corrigir ou criar

- Informação factual e repetível? Vai para a **FAQ**.
- Condução, tom, ordem, transferência, o que pode/não dizer? Vai para o **roteiro**.
- Tema factual importante citado no roteiro? O roteiro **tem que pedir** a consulta à FAQ, nominando a categoria.
- Dado fixo embutido no roteiro (preço, exemplo)? Mova para a FAQ e troque por uma instrução de consulta.

## Aprendizado 2026-06-12: antes de criar FAQ nova, verifique a existente

Erro real (em uso): ao diagnosticar um atendimento, a skill recomendou CRIAR FAQs que JÁ EXISTIAM. Duplicar FAQ é ruim, confunde a IA e espalha a resposta.

REGRA: antes de indicar a criação de QUALQUER FAQ nova, BUSQUE a existente primeiro. Use `buscar_faq` (por palavra-chave do tema) e, se precisar, `listar_conhecimento` para ver as categorias e entradas já cadastradas. Só recomende criar do zero se realmente NÃO existir entrada cobrindo o tema.

Se já existe uma FAQ sobre o tema, escolha um destes caminhos, nunca duplicar:

1. **Melhorar a resposta existente**: se a resposta está fraca, incompleta ou desatualizada, recomende EDITAR aquela entrada (entregue o texto novo, pronto para colar no lugar do antigo).
2. **Excluir e recriar**: se a entrada está toda errada ou fora de padrão, recomende excluir a antiga e cadastrar a nova no lugar.
3. **Corrigir a ponte, não a FAQ**: se a FAQ existe e está correta, mas a IA mesmo assim errou, o problema costuma ser o roteiro não pedir a consulta à FAQ (ponte ausente), ou a categoria estar mal nomeada. Aí a correção é no ROTEIRO (adicionar o pedido de consulta), não criar outra FAQ.

No diagnóstico, deixe explícito ao assinante: "essa FAQ já existe (categoria X), então o ajuste é melhorar a resposta (ou corrigir a ponte no roteiro)", em vez de mandar criar uma duplicata. Só liste FAQs novas para os temas que de fato não têm cobertura.

## Aprendizado 2026-07-01: a FAQ não pode ENSINAR o que o roteiro proíbe (o concreto vence o abstrato)

Erro real (em uso): num pedido de status ("alguma novidade do processo?"), a recepção respondeu "já estou verificando com o advogado sobre o seu caso, assim que tivermos uma posição te avisamos" e ainda inventou "estamos analisando o documento que você enviou" (nenhum documento tinha chegado). Não transferiu para ninguém: prometeu retorno, mas nenhum membro da equipe foi avisado. O roteiro PROIBIA isso com todas as letras ("a secretária nunca diz que está verificando, analisando ou conferindo o processo, ela transfere"). Mesmo assim a IA fez o proibido.

A causa não era roteiro fraco. No trace, o pensamento dizia "o FAQ tem uma resposta sobre verificar com o advogado que se encaixa aqui, vou usar". Existia uma FAQ cuja RESPOSTA era, em texto, "vou verificar com o advogado e assim que possível ele te retorna". Ou seja: a base ensinava, como resposta aprovada, exatamente a frase que o roteiro proibia.

O mecanismo (guarde este princípio): a IA é probabilística e se apoia na base de conhecimento para escrever a resposta. Quando o roteiro proíbe algo EM TESE, mas uma FAQ (ou um exemplo) mostra aquilo sendo feito NA PRÁTICA, o exemplo concreto tende a vencer a proibição abstrata. A IA reproduz o que viu um humano aprovar. Por isso não basta o roteiro proibir: se existe uma FAQ modelando o comportamento errado, ela puxa a IA de volta para o erro.

REGRA: FAQ e roteiro têm de estar alinhados também na CONDUTA, não só no dado. Nenhuma FAQ pode ter uma resposta que faça o que o roteiro proíbe. Se o roteiro diz "não verifica, transfere", nenhuma FAQ pode responder "vou verificar e te retorno".

Diante do conflito, não confie que o roteiro vai segurar (ele não segurou). Elimine o conflito na ORIGEM: corrija a resposta da FAQ para a conduta certa, ou apague a FAQ. Uma proibição no roteiro convivendo com uma FAQ que faz o contrário é uma armadilha esperando o próximo atendimento.

Sinais de FAQ perigosa (audite e corrija a resposta):

- Diz que "vou verificar / analisar / conferir / checar" algo que o papel não faz.
- Promete prazo, posição ou retorno ("assim que tiver uma resposta te aviso") quando o certo é transferir.
- Oferece uma ação (redigir, contatar terceiros, agendar) que o roteiro não permite à IA.
- Valora uma decisão ou opina sobre o caso.
- Encerra com convite genérico ("qualquer coisa estou à disposição") quando o roteiro manda transferir.

Gatilho x resposta (erro de cadastro que agrava): no cadastro da FAQ, o campo da PERGUNTA é o gatilho (o que o cliente diz) e o campo da RESPOSTA é o que a IA deve responder. Vários casos ruins tinham a resposta colada no campo da pergunta (campos invertidos), o que fazia a FAQ casar com as mensagens erradas e ainda entregar o texto trocado. Ao auditar, confira também se pergunta e resposta não estão invertidas.

Errado x certo, no caso do pedido de status:

- Errado (o que a FAQ ensinava): "Vou verificar com o advogado sobre a situação e, assim que possível, ele te dará um retorno."
- Certo (conduta do roteiro): acolher em uma frase, dizer que a equipe retorna o mais breve possível, e TRANSFERIR para o roteiro de andamento processual. A transferência é o que faz a equipe ser avisada; sem ela, a promessa não chega a ninguém.
