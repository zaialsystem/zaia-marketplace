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
