# Taxonomia das causas-raiz de erro da IA

Use este arquivo no Passo 5 do diagnóstico. Para cada erro, encontre a categoria que melhor explica **por que a resposta errada foi a mais provável** dado o que a IA recebeu. Um mesmo atendimento pode ter mais de uma causa; trate todas.

Lembre da ideia central: a IA escolhe a resposta mais provável a partir do roteiro, da FAQ e da mensagem do cliente. Cada categoria abaixo é uma forma de o roteiro/FAQ ter empurrado a probabilidade para o lado errado.

---

## 1. Instrução ausente (o caso não estava previsto)

**O que é:** o cliente trouxe uma situação que o roteiro simplesmente não cobria. Sem instrução, a IA improvisa, e improviso é onde mais aparece resposta indesejada (inventar, prometer, opinar, oferecer serviço).

**Sinais:**
- No trace, o "pensamento" da IA mostra hesitação ou criação de regra própria.
- Você lê o roteiro inteiro do passo e não acha nada sobre aquele tipo de mensagem.
- A resposta da IA é "plausível mas fora do tom" do escritório/empresa.

**Como corrigir:** adicione um caso novo ao passo correspondente, descrevendo a situação e a conduta esperada, com um exemplo. Se for uma situação que deve sair do fluxo automático, defina a ação (transferir para humano, encerrar, etc.).

**Exemplo de erro:** cliente pergunta "vocês parcelam no cartão?" e não havia nada sobre formas de pagamento. A IA chuta "sim, parcelamos em até 12x" sem base.

---

## 2. Instrução ambígua ou vaga

**O que é:** a instrução existia, mas era genérica demais ("seja acolhedora", "responda de forma natural", "ajude o cliente"). Frase vaga admite muitas leituras, e a IA pode escolher uma que você não queria.

**Sinais:**
- A instrução usa adjetivos abstratos sem dizer o que fazer na prática.
- Dois atendimentos parecidos saíram bem diferentes (sinal clássico de vaguidade: a probabilidade está espalhada).

**Como corrigir:** troque o abstrato pelo concreto. Diga o que fazer e o que não fazer, com exemplo. "Seja acolhedora" vira "acolha em uma frase curta e transfira; não ofereça soluções nem opine sobre o caso. Ex.: ...".

**Exemplo de erro:** "console o cliente" levou a IA a dar conselhos emocionais longos, quando o esperado era acolher e transferir.

---

## 3. Instruções em conflito

**O que é:** dois trechos do roteiro (ou roteiro x FAQ) pedem coisas incompatíveis. A IA precisa escolher um lado, e às vezes escolhe o que você não queria.

**Sinais:**
- Você encontra duas regras que, naquele caso, não cabem juntas.
- O trace mostra a IA "pesando" entre duas instruções.

**Como corrigir:** elimine o conflito. Defina a precedência explicitamente ("esta regra prevalece sobre qualquer outra") ou reescreva um dos trechos. Roteiros bem-feitos costumam ter um bloco de "regras invioláveis" que vence em caso de choque.

**Exemplo de erro:** um trecho diz "seja proativa e ofereça ajuda" e outro diz "nunca ofereça executar ações". Diante de "não sei o que responder no e-mail", a IA ofereceu redigir o e-mail, seguindo o primeiro trecho.

---

## 4. Instrução enterrada (existia, mas perdeu peso)

**O que é:** a regra certa estava no roteiro, porém soterrada no meio de um bloco enorme, longe do caso onde importava. Quanto mais longe e mais diluída, menos peso ela tem na decisão.

**Sinais:**
- Você acha a regra, mas teve que caçar.
- A regra está num parágrafo de "padrão geral" e não no caso específico onde precisava agir.

**Como corrigir:** suba a regra para perto do caso onde ela atua, e/ou coloque-a no bloco de regras invioláveis no topo. Repita a regra curtinha no ponto de uso. Redundância proposital ajuda a IA.

**Exemplo de erro:** "nunca prometer retorno imediato" estava só no rodapé de padrões de comunicação; no caso de urgência emocional, a IA prometeu "já vou resolver agora".

---

## 5. Falta exemplo do caso concreto

**O que é:** a regra estava escrita em tese, mas sem um exemplo "errado x certo" daquele caso. Exemplos ancoram a IA muito mais do que regras abstratas.

**Sinais:**
- A regra existe e é clara, mas o erro ainda acontece em um caso específico.
- O roteiro tem poucos ou nenhum exemplo.

**Como corrigir:** adicione um par "errado x certo" com o texto real que apareceu na conversa. Use o erro que de fato aconteceu como o exemplo de "errado". Isso transforma o caso real em vacina.

**Exemplo de erro:** a regra "não valorar decisões" existia, mas sem exemplo. A IA disse "é importante manter postura técnica" diante de uma derrota do cliente. Vira exemplo de "errado".

---

## 6. FAQ ausente ou fraca

**O que é:** o cliente fez uma pergunta factual (preço, prazo, como funciona, documentos, horário) cuja resposta deveria estar na base de conhecimento (FAQ) e não estava, ou estava mal escrita/incompleta. Sem o dado, a IA inventa ou desvia.

**Sinais:**
- `buscar_faq` com o termo da pergunta não retorna nada útil.
- No trace, a IA não consultou FAQ ou consultou e não achou.
- A resposta tem número/dado que você nunca cadastrou (sinal de invenção).

**Como corrigir:** crie ou reescreva a entrada de FAQ com a informação correta, em linguagem direta. Cubra também as variações de como o cliente pergunta a mesma coisa (sinônimos).

**Exemplo de erro:** "qual o valor da análise?" sem FAQ de preços levou a IA a dar um valor errado.

---

## 7. FAQ e roteiro desalinhados

**O que é:** a FAQ informa uma coisa e o roteiro orienta outra. A IA pode misturar as duas e gerar resposta inconsistente.

**Sinais:**
- O dado da FAQ contradiz o que o roteiro manda dizer.
- Respostas certas no conteúdo mas no tom/conduta errados, ou vice-versa.

**Como corrigir:** alinhe os dois. Decida qual é a fonte da verdade para aquele ponto e ajuste o outro. Em geral: FAQ guarda o dado, roteiro guarda a conduta de como entregar o dado.

**Aprendizado 2026-07-01 (subtipo perigoso: a FAQ ensina a conduta proibida):** o desalinhamento mais grave não é de dado, é de CONDUTA. Quando o roteiro proíbe uma conduta em tese ("a secretária não verifica o processo, ela transfere") mas existe uma FAQ cuja RESPOSTA faz exatamente isso ("vou verificar e te retorno"), a IA copia a FAQ e viola o roteiro: o exemplo concreto vence a proibição abstrata. Sinais de FAQ perigosa: respostas que dizem "vou verificar/analisar/conferir", que prometem prazo ou posição, que oferecem ação (redigir, contatar, agendar), que valoram uma decisão, ou que encerram com convite genérico quando o roteiro manda transferir. No diagnóstico, olhe no trace qual FAQ/exemplo foi consultado: muitas vezes a frase errada veio copiada de lá. Correção: não conte com o roteiro para segurar, corrija ou apague a FAQ na origem (ver `integracao-roteiro-faq.md`). Cheque também se pergunta e resposta não estão invertidas no cadastro (resposta colada no campo da pergunta casa com mensagens erradas).

---

## 8. Ponte roteiro/FAQ ausente

**O que é:** a FAQ existia e estava correta, mas o passo do roteiro não pediu, em texto, para a IA consultar a FAQ daquele tema. Sem o pedido explícito, a IA improvisa em vez de usar o conhecimento curado.

**Sinais:**
- Tema factual no roteiro (preço, prazo, exemplo de caso) sem instrução de "consulte a FAQ da categoria X".
- A FAQ certa existe, mas o trace mostra que a IA não a consultou naquele passo.

**Como corrigir:** adicione ao passo a instrução de consulta explícita, nominando a categoria, e a salvaguarda de não inventar (ver `integracao-roteiro-faq.md`).

---

## 9. Decisão de classificação/transferência errada (erro lá atrás)

**O que é:** o erro visível foi consequência de uma escolha anterior: a IA mandou a conversa para o passo ou roteiro errado, ou não transferiu quando devia. O sintoma aparece depois, mas a causa está na bifurcação.

**Sinais:**
- A resposta final está "certa para o roteiro errado".
- O trace mostra a classificação inicial equivocada.
- O cliente acabou num fluxo que não era o caso dele.

**Como corrigir:** ajuste as condições de roteamento/transferência no passo de recepção/classificação. Deixe explícitos os gatilhos de cada destino e o destino-padrão na dúvida (em geral, suporte/humano). Reforce quando transferir para humano.

**Exemplo de erro:** cliente já tinha processo e queria status, mas caiu no fluxo de "novo cliente / vendas", porque o gatilho de "atualização processual" estava frouxo.

---

## 10. Último passo com pergunta (duas mensagens ao transferir)

**O que é:** o último passo do roteiro, além de transferir, faz uma pergunta ou validação. No momento da transferência a IA acaba enviando duas mensagens juntas: a pergunta deste passo e a primeira pergunta do roteiro de destino.

**Sinais:** no trace, duas mensagens em sequência com perguntas logo na virada de departamento; o último passo do roteiro tem pergunta/validação além da ação.

**Como corrigir:** deixar o último passo EXCLUSIVAMENTE para a ação de transferência (sem pergunta) e mover a recapitulação/validação para um passo próprio antes. Se for triagem, adicionar um passo a mais para a costura e manter o último só como transferência.

---

## 11. Identidade do interlocutor trocada (mensagem encaminhada de terceiro)

**O que é:** o cliente encaminha para o escritório uma mensagem que ele recebeu de um terceiro (o plano de saúde, uma auditoria, um médico, um perito, a parte contrária). Essa mensagem vem escrita em primeira pessoa por esse terceiro. A IA adota a identidade do terceiro como se fosse o interlocutor atual, responde a ele e chega a se passar pelo próprio cliente, executando ações que o roteiro proíbe (agendar, autorizar, negociar, confirmar dados). É um erro perigoso: a IA passa a falar por conta do cliente com um terceiro externo.

**Sinais:**
- No trace, o pensamento diz algo como "o contato é [nome ou função do terceiro]", trocando quem é o cliente.
- A resposta cumprimenta e responde ao terceiro ("Bom dia, [terceiro]! Pode prosseguir..."), não ao cliente.
- A IA confirma a identidade do beneficiário, negocia ou agenda com o terceiro.
- A mensagem do cliente costuma vir com "recebi essa mensagem", "encaminhando", "olha o que me mandaram", ou aparece marcada como encaminhada.

**Como corrigir:** no passo de recepção, criar regra de prioridade máxima que ancore a identidade no dono do número (o cliente que a IA já conhece), nunca no terceiro citado dentro do texto encaminhado. Uma mensagem encaminhada não muda com quem a IA fala; se o terceiro cita o nome do beneficiário e ele é o próprio cliente, isso só confirma que a IA fala com o cliente. Proibir explicitamente: responder ao terceiro, se passar pelo cliente ou beneficiário, confirmar dados, negociar, autorizar ou agendar. Tratar como comunicação recebida: acolher em uma frase e transferir para o fluxo de atualização/andamento. Adicionar um exemplo errado x certo do caso.

**Exemplo (errado x certo):**
Cliente encaminha: "Bom dia, meu nome é [Auditor], falo da auditoria do plano. Falo com o beneficiário [Cliente]? Podemos agendar a perícia?" e escreve "recebi essa mensagem".
ERRADO: "Bom dia, [Auditor]! Sim, pode prosseguir. Você está falando com o número do beneficiário. Qual sua disponibilidade de datas?" (a IA se passou pelo cliente e foi negociar com o terceiro).
CERTO: "Recebi, [Cliente]. Vou repassar essa mensagem do plano para a equipe e retornaremos o mais breve possível." e transferir para o fluxo de atualização processual.

---

## Checklist rápido de diagnóstico

Ao analisar um erro, percorra:

1. O roteiro previa essa situação? (se não, causa 1)
2. A instrução era concreta? (se não, causa 2)
3. Havia outra instrução brigando com essa? (se sim, causa 3)
4. A regra certa estava perto do caso? (se não, causa 4)
5. Tinha exemplo desse caso? (se não, causa 5)
6. Era pergunta factual? A FAQ tinha a resposta? (se não, causa 6)
7. FAQ e roteiro concordam? (se não, causa 7)
8. O roteiro pediu consulta à FAQ no tema? (se não, causa 8)
9. A conversa entrou no fluxo certo lá atrás? (se não, causa 9)
10. O último passo do roteiro faz pergunta além de transferir? (se sim, causa 10)
11. A mensagem era de um terceiro encaminhada pelo cliente? A IA manteve a identidade no cliente e não respondeu/agendou pelo terceiro? (se não, causa 11)
