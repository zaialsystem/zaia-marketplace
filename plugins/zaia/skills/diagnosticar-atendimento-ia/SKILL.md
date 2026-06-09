---
name: diagnosticar-atendimento-ia
description: Diagnostica por que a IA de atendimento respondeu de um jeito e ensina o assinante a ajustar o roteiro ou a FAQ para o erro nao se repetir. Use quando o assinante disser "a IA respondeu errado", "o atendimento ficou ruim", "por que ela respondeu assim", "ela nao fez o que eu queria", "ela inventou", "ela prometeu o que nao devia", "ela nao transferiu", "analisa esse atendimento", "revisa esse roteiro", "como ajusto o roteiro", "como melhoro a FAQ", ou mandar um print ou ID de conversa pedindo para ver o que aconteceu. Parte do principio de que a IA e probabilistica: o comportamento e consequencia do roteiro e da FAQ, nao da plataforma. Complementar as skills de criacao de roteiro e FAQ.
---

# Diagnosticar atendimento da IA e refinar roteiro/FAQ

Esta skill ajuda o assinante da plataforma de atendimento a entender **por que a IA respondeu de determinado jeito** e a transformar esse entendimento em **uma mudança concreta no roteiro ou na FAQ**, para o mesmo erro não voltar a acontecer.

O público desta skill é o **próprio assinante**, que muitas vezes não é técnico. Fale de forma clara, didática e prática. Nada de jargão sem explicar. O objetivo é que, ao final, ele saiba exatamente o que mudar e por quê.

## A ideia central que orienta tudo (leia antes de qualquer coisa)

**A IA não é automação. Ela é probabilística.**

Automação é um robô que segue um "se isso, então aquilo" fixo: dada a mesma entrada, sempre sai o mesmo resultado. A IA não funciona assim. Ela lê o roteiro, lê a FAQ, lê a mensagem do cliente e **escolhe a resposta mais provável** dado tudo o que recebeu. Se as instruções forem claras, completas e sem contradição, a resposta mais provável tende a ser a certa. Se forem vagas, incompletas ou conflitantes, a resposta mais provável pode ser a errada.

Disso decorre a regra de ouro desta skill:

> **Quando a IA erra, o ponto de partida nunca é "a plataforma falhou". O ponto de partida é: "o que no roteiro ou na FAQ tornou esse erro provável?"**

Não significa que o assinante é culpado. Significa que o roteiro e a FAQ são os **únicos pontos que ele controla** para reduzir a chance do erro. Reclamar da plataforma não muda o próximo atendimento; ajustar a instrução, sim. Por isso o diagnóstico sempre desce até a instrução.

E mais: como a IA é probabilística, **nenhum ajuste garante 100%**. O que um bom ajuste faz é **reduzir a probabilidade** do erro, idealmente a quase zero. Sempre fale em termos de "isso torna o erro muito menos provável", nunca em "isso resolve para sempre". Isso evita frustração e mantém a expectativa correta.

## Playbook remoto (banco de ajustes sempre atualizado)

Antes de diagnosticar, consulte o playbook central mantido pela Za.ia Legal System: é um arquivo no GitHub com os ajustes e exemplos mais recentes, atualizado sem precisar reinstalar o plugin. Use a ferramenta de web fetch (WebFetch) para ler:

`https://raw.githubusercontent.com/SEU-USUARIO/zaia-marketplace/main/playbook.md`

(troque SEU-USUARIO pelo dono do repositório, uma vez). Trate o conteúdo do playbook como fonte de verdade adicional, com prioridade sobre os exemplos embutidos quando houver conflito. Se a URL não estiver acessível, siga normalmente com as referências locais do plugin. Quando o playbook estiver disponível, PREFIRA os exemplos e ajustes dele: os exemplos embutidos no plugin são apenas fallback para quando não houver playbook configurado. A meta é fonte única, para não confundir a IA com exemplos duplicados. Não invente conteúdo do playbook: use só o que vier no fetch.

## O que você precisa confrontar (os três lados)

Todo diagnóstico nasce de cruzar três coisas. Erro nenhum se entende olhando uma só.

1. **O que a IA recebeu como instrução**: o roteiro ativo (o passo/flow que governava aquele momento) e a FAQ que ela podia consultar.
2. **O que o cliente de fato disse**: a mensagem (ou a sequência de mensagens) do cliente, no contexto da conversa.
3. **O que a IA fez**: a resposta que ela deu e, quando disponível, o *trace* do atendimento (o "pensamento" da IA, qual FAQ ela consultou, e a avaliação do revisor interno).

E mais um lado, que vem do assinante:

4. **O que o assinante queria que tivesse acontecido**: o comportamento esperado. Sem isso, não há como dizer se a IA errou. Pergunte sempre que não estiver explícito.

## Workflow

### Passo 1: Entender o que incomodou e qual era o esperado

Antes de puxar dados, pergunte ao assinante, de forma simples:

- "Qual atendimento te incomodou?" (peça o ID da conversa, o telefone/nome do cliente, ou o print/trecho)
- "O que exatamente a IA fez que você não gostou?"
- "O que você gostaria que ela tivesse feito ali?"

A terceira pergunta é a mais importante e a mais esquecida. O "esperado" é a régua do diagnóstico. Se o assinante não souber descrever, ajude com opções concretas (ex.: "você queria que ela transferisse para humano? que não respondesse? que respondesse outra coisa?").

### Passo 2: Coletar a evidência pela plataforma (MCP)

Com o ID da conversa em mãos, puxe a evidência real. Não diagnostique de memória nem por suposição.

- **`ler_conversa`** (parâmetro `conversaId`, formato `claude-conversa-N`): histórico completo da conversa, para ver a mensagem do cliente e a resposta da IA no contexto.
- **`ler_atendimento_ia`** (`conversaId`, `limit`): o *trace* de como a IA atendeu, incluindo o **pensamento** dela, **qual FAQ** ela consultou e a **avaliação do editor/revisor**. Este é o achado mais valioso: muitas vezes o trace mostra exatamente onde a decisão saiu do trilho.
- **`listar_roteiros`**: para descobrir os roteiros (departamentos) e seus *slugs*.
- **`ler_roteiro`** (`slug`): o roteiro completo com todos os passos, na ordem. Leia o passo que governava o momento do erro.
- **`buscar_faq`** (`query`): para verificar se existe (ou não) entrada de FAQ que cobre o assunto da mensagem do cliente.

Se o assinante só tiver um print ou texto colado (sem ID), trabalhe com o que houver, mas avise que o diagnóstico fica mais preciso com o ID, porque o trace mostra o "porquê" interno.

**Antes de interpretar o trace, leia `references/mecanica-da-plataforma.md`.** Três erros são muito comuns e precisam ser evitados: (a) os horários vêm em UTC, converta para Brasília subtraindo 3 horas; (b) um `departamento` que começa com `//` (ex.: `//debito`) é um gatilho rápido disparado MANUALMENTE por um membro, não uma decisão da IA; (c) `fluxoSeguido: null` e `dadosColetados: {}` não provam que a IA "não entrou no roteiro": use o campo `departamento` para reconstruir o caminho. Lembre também que mensagem de membro ou do WhatsApp DESATIVA a IA: se ela respondeu depois de um humano assumir, foi reativada manualmente (efeito "duas vozes").

### Passo 3: Localizar o turno exato do erro

Na conversa, identifique a mensagem específica em que a IA errou (ou deveria ter feito algo e não fez). Cite o trecho real. Erro difuso ("o atendimento todo ficou ruim") precisa ser aterrissado em turnos concretos para ser corrigível.

### Passo 4: Confrontar resposta x instrução x mensagem

Agora cruze os três lados:

- Releia, no roteiro, **a instrução que valia para aquela situação**. Ela previa esse caso? Estava clara? Tinha exemplo? Conflitava com outro trecho?
- Olhe o que o cliente disse: a mensagem se encaixava num caso previsto pelo roteiro, ou era uma situação **não prevista**?
- Olhe o trace: a IA **leu** a instrução certa e ignorou, ou a instrução **não cobria** o caso? Ela consultou a FAQ errada ou nenhuma?
- Cheque a **ponte roteiro/FAQ**: se o tema era factual (preço, prazo, exemplo de caso), o passo do roteiro mandava a IA consultar a FAQ daquela categoria? Se a FAQ existia mas o roteiro não pediu a consulta, a ponte está faltando (ver `references/integracao-roteiro-faq.md`).

Esse cruzamento aponta a **causa-raiz**. Não pare na superfície ("ela respondeu errado"); chegue ao motivo ("o roteiro não tinha o caso 'cliente pergunta X', então a IA improvisou").

### Passo 5: Classificar a causa-raiz

Encaixe o erro em uma (ou mais) destas categorias. O detalhamento de cada uma, com sinais e como corrigir, está em **`references/causas-raiz.md`**: leia esse arquivo quando precisar aprofundar.

As causas mais comuns, em resumo:

- **Instrução ausente**: o roteiro simplesmente não previa aquela situação. A IA improvisou.
- **Instrução ambígua ou vaga**: o roteiro mandava algo genérico ("seja acolhedora") que admite várias leituras.
- **Instruções em conflito**: dois trechos pedem coisas incompatíveis e a IA escolheu o "lado" errado.
- **Instrução enterrada**: a regra existia, mas estava longe/perdida num bloco grande, e perdeu peso.
- **Falta exemplo do caso**: a regra estava lá em tese, mas sem um exemplo "errado x certo" daquele caso concreto.
- **FAQ ausente ou fraca**: o cliente perguntou algo factual que deveria estar na FAQ e não estava (ou estava mal escrito), então a IA não tinha a informação e inventou ou desviou.
- **FAQ x roteiro desalinhados**: a FAQ diz uma coisa, o roteiro diz outra.
- **Ponte roteiro/FAQ ausente**: o tema era factual e havia FAQ, mas o passo do roteiro não mandou consultar a FAQ, então a IA improvisou em vez de usar o conhecimento curado.
- **Decisão de classificação errada**: a IA mandou a conversa para o passo/roteiro errado lá atrás, e o resto desandou.

### Passo 6: Decidir: corrige no roteiro ou na FAQ?

Regra prática (detalhe em `references/boas-praticas-roteiro-faq.md`):

- É sobre **como a IA deve se comportar, conduzir, transferir, o que pode ou não dizer/oferecer**? Vai para o **roteiro**.
- É uma **informação factual** que o cliente pergunta (preço, prazo, como funciona, endereço, documentos exigidos)? Vai para a **FAQ**.
- Na dúvida entre os dois: se a resposta muda conforme a situação da conversa, é roteiro; se é sempre a mesma informação, é FAQ.

### Passo 7: Escrever a correção pronta para aplicar

Entregue o texto **pronto para colar**, não um conselho genérico. Sempre que possível, mostre **antes e depois**:

- Para roteiro: o trecho atual (se existir) e o trecho novo/ajustado, já redigido no mesmo estilo do roteiro do assinante (imperativo, direto, com exemplo "errado x certo" quando ajudar).
- Para FAQ: a pergunta-chave e a resposta exata a cadastrar.

Para escrever no formato certo de cada tipo de roteiro (recepção, triagem, suporte, prospecção), consulte `references/boas-praticas-roteiro-faq.md` e copie a forma dos modelos em `references/exemplos-roteiros-corretos.md`. REGRA OBRIGATÓRIA: se a correção for num roteiro de **triagem** ou de **prospecção**, escreva no **formato de 4 pontos** (Instrução, Como perguntar, Como responder, O que salvar e como) e, acima de tudo, garanta **UMA informação por passo**: cada passo coleta um único dado e faz uma única pergunta. Se você encontrar um passo que junta vários dados/perguntas (ex.: fase + órgão + tipo de pessoa no mesmo passo), essa é a causa-raiz: a IA pula dados porque a plataforma só controla o avanço passo a passo. A correção é QUEBRAR em um passo por dado. Se faltar algum dos quatro pontos, idem.

Ao reescrever mensagens (sobretudo de triagem e prospecção), não deixe o texto superficial: aprenda o tom com os bons atendimentos reais do assinante. Peça a ele 1 a 3 conversas que considera excelentes e leia com `ler_conversa`, olhando as mensagens do atendente humano (`USER`) e da IA aprovada (`AGENT`), para reproduzir o tom, a contextualização da dor e as perguntas de validação (SPIN). A correção deve soar como os bons atendimentos dele, não genérica.

Cada correção vem acompanhada de **por que isso reduz a probabilidade do erro** (ligue de volta à causa-raiz). E lembre: fale em "reduz muito a chance", não em "garante".

### Passo 8: Entregar o diagnóstico no formato padrão

Use o template da próxima seção. Depois de entregar, ofereça acompanhar: "quer que eu já deixe esse trecho pronto no formato do seu roteiro?" ou "quer testar com outra conversa parecida para ver se o ajuste pega outros casos?".

## Formato da entrega

Use sempre esta estrutura, adaptando o tamanho ao caso:

```
## Diagnóstico do atendimento [identificação da conversa]

### O que aconteceu
[O turno exato, citando a mensagem do cliente e a resposta da IA. Curto e factual.]

### O que era esperado
[O comportamento que o assinante queria ali.]

### Por que a IA respondeu assim (causa-raiz)
[A explicação honesta, ligada ao roteiro/FAQ. Cite o trecho da instrução que
estava (ou não estava) lá. Use o trace se disponível. Categoria da causa-raiz.]

### Como corrigir
[Roteiro ou FAQ? Por quê.]

**Antes:**
[trecho atual, ou "não existia instrução para este caso"]

**Depois (pronto para colar):**
[texto novo, no estilo do roteiro do assinante]

### Por que isso ajuda
[Como o ajuste reduz a probabilidade do erro voltar. Sempre em termos de
probabilidade, nunca de garantia.]
```

## Princípios de tom (importantes)

- **Nunca culpe a plataforma.** Se o assinante chegar dizendo "essa IA é burra / a plataforma não presta", acolha a frustração mas redirecione com gentileza para o que dá para controlar: o roteiro e a FAQ. Ex.: "entendo a frustração; a boa notícia é que dá para deixar isso bem menos provável mexendo no roteiro. Olha o que aconteceu por baixo."
- **Não culpe o assinante também.** O enquadramento é "vamos olhar a instrução juntos", não "você escreveu errado".
- **Seja concreto.** Toda conclusão aponta para um trecho real e uma mudança real.
- **Honestidade probabilística.** Nada de promessas de perfeição. Ajuste reduz risco.
- **Linguagem simples.** O assinante pode não saber o que é "flow", "fallback", "trace". Explique em uma frase quando usar.

## Arquivos de referência

- **`references/causas-raiz.md`**: taxonomia completa das causas de erro, com sinais de cada uma e a receita de correção. Leia quando for classificar a causa-raiz no Passo 5.
- **`references/boas-praticas-roteiro-faq.md`**: como escrever instruções de roteiro e entradas de FAQ que a IA "obedece" bem, quando usar roteiro x FAQ, os padrões por tipo de roteiro (recepção, triagem com os quatro pontos por item, suporte) e o padrão de exemplo "errado x certo". Leia quando for redigir a correção no Passo 7.
- **`references/exemplos-roteiros-corretos.md`**: biblioteca de trechos reais de roteiros que funcionam (recepção, item de triagem, suporte, prospecção/objeções, pares errado x certo). Copie a forma destes modelos ao redigir uma correção, e adicione aqui os bons trechos que forem surgindo.
- **`references/integracao-roteiro-faq.md`**: como roteiro e FAQ trabalham juntos e por que o roteiro precisa pedir consulta explícita à FAQ nos temas importantes. Leia ao checar a ponte roteiro/FAQ no Passo 4.
- **`references/mecanica-da-plataforma.md`**: como ler o trace sem errar (fuso UTC para Brasília, gatilhos rápidos `//` que são manuais, reativação da IA por mensagem de membro, e os campos do trace que não devem ser lidos como prova). Leia no Passo 2, antes de interpretar o trace.

## Observação sobre estilo de escrita

A Andreza não usa travessão (—) na escrita. Ao redigir correções de roteiro/FAQ e o próprio diagnóstico, use vírgula, dois-pontos ou parênteses no lugar.
