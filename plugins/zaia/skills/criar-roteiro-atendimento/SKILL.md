---
name: criar-roteiro-atendimento
description: Cria roteiros de atendimento por IA no formato JSON da plataforma (chave "flows"), entrevistando o assinante sobre o negócio e o serviço antes de gerar. Use SEMPRE que o assinante disser "criar roteiro", "novo roteiro", "montar o fluxo de atendimento", "roteiro para o serviço X", "preciso de um script para a IA", "montar a recepção/triagem/prospecção", "fazer o atendimento de [serviço]", "quero melhorar o atendimento da IA", "melhorar a conversão", ou qualquer variação que envolva produzir ou refazer um roteiro para o agente de IA atender clientes (em geral no WhatsApp). A skill primeiro conhece o negócio, depois gera o JSON pronto para importar, já com a ponte explícita para a FAQ nos temas importantes (preços, prazos, exemplos de casos). É complementar à skill `criar-faq-atendimento` (gere as duas juntas quando o assinante estiver montando um serviço novo) e à `diagnosticar-atendimento-ia` (que conserta roteiros existentes).
---

# Criar roteiro de atendimento por IA

Esta skill gera roteiros de atendimento no formato da plataforma (JSON com a chave `flows`), prontos para o assinante importar. Um roteiro é um script que o agente de IA segue para conduzir a conversa com o cliente.

O público é o assinante, em geral não técnico. Conduza com clareza, uma pergunta por vez, e só gere o JSON depois de entender o negócio. Um roteiro genérico atende mal; um roteiro nascido do contexto real do escritório atende bem.

## Princípio que orienta a escrita

A IA é probabilística: ela segue melhor instruções concretas, com exemplos e destino-padrão para o que não se encaixa. Roteiro vago gera atendimento imprevisível. Por isso cada passo deve dizer o que fazer, como, e o que salvar, sem deixar lacuna para improviso. Detalhes de boa escrita estão em `references/formato-json-roteiro.md`.

Leia também `regras/integracao-roteiro-faq` do manifest: roteiro e FAQ trabalham juntos, e o roteiro precisa pedir consulta explícita à FAQ nos temas importantes. Isso é parte obrigatória da geração, não um extra.

## Base de conhecimento e atualizacao (consultar SEMPRE no inicio)

Antes de agir, leia o indice central da Za.ia Legal System (atualizado sem reinstalar o plugin). Use WebFetch para ler:

`https://raw.githubusercontent.com/zaialsystem/zaia-marketplace/main/knowledge/manifest.json`

Do manifest:
1. Para cada arquivo relevante A ESTA skill (filtre pelo campo `aplica`, e por segmento/tipo/categoria quando houver), monte a URL com `rawBase + path` e leia via WebFetch SO o que precisar. Para esta skill, os arquivos que costuma usar sao: as regras de roteiro `regras/roteiro-geral`, `regras/recepcao` e `regras/prospeccao`, a integracao roteiro/FAQ `regras/integracao-roteiro-faq`, e a biblioteca `roteiros/` (escolha, pelo segmento/tipo informados no manifest, um roteiro-exemplo proximo do negocio do assinante para usar como MOLDE de partida depois da entrevista).
2. Em conflito com o conteudo embutido neste plugin, o conteudo do GitHub PREVALECE; o embutido e so fallback minimo. Nao invente conteudo de arquivo que nao foi lido.
3. Se o fetch falhar, siga com o fallback minimo embutido e avise o assinante que esta sem a base atualizada.

Aviso de atualizacao: leia a versao instalada em `${CLAUDE_PLUGIN_ROOT}/.claude-plugin/plugin.json` e compare com `manifest.pluginVersion`. Se o manifest for maior, avise em linguagem simples: "Saiu uma atualizacao do plugin Za.ia. Veja as novidades e atualize em Customizar > Plugins." (cite o resumo de `novidades.md`). Se `manifest.skills` listar uma skill que voce nao tem instalada, avise que ha skill nova a instalar pelo mesmo caminho.

## Workflow

### Passo 1: Conhecer o negócio antes de gerar

Não gere nada antes de entender o escritório. Pergunte, uma por vez, e incentive respostas detalhadas:

1. Nome do escritório, tempo de atuação, missão e diferenciais.
2. Áreas de atuação e, dentro de cada uma, os serviços específicos (um roteiro por serviço, não por área genérica).
3. Quem são os advogados e suas especialidades (autoridade).
4. Perfil do cliente ideal e as dores mais comuns.
5. Como é o atendimento hoje: o que funciona, o que não funciona, reclamações frequentes.
6. Diferenciais do serviço: consulta gratuita, atende online, formas de pagamento, prazo de retorno.
7. Bons atendimentos de referência: peça ao assinante para apontar conversas reais que ele considera ótimas (por ID `claude-conversa-N`, nome ou telefone do contato, ou "os melhores da semana"). Elas dão o tom, a linguagem e as perguntas que já funcionam.

Diga ao assinante que quanto mais detalhe, mais assertivo o roteiro.

### Passo 1.1: Aprender o tom com os bons atendimentos (via MCP)

Esta etapa é o que evita roteiros com mensagens superficiais. O assinante está conectado ao MCP da plataforma, então não invente o tom: aprenda com os atendimentos reais dele.

1. Peça bons atendimentos para análise. Duas formas: (a) ele indica atendimentos vinculados à conta da Zaia (por ID `claude-conversa-N`, nome ou telefone, e você lê com `buscar_contatos` + `listar_conversas_por_contato` + `ler_conversa`); ou (b) ele cola a transcrição de um bom atendimento. Peça de 1 a 3.
2. Leia essas conversas com `ler_conversa`. Foque nas mensagens do atendente humano (sender `USER`) e da IA aprovada (`AGENT`): é ali que está o tom bom e as perguntas de validação que convertem.
3. Extraia e reaproveite: o tom de voz (acolhedor, consultivo, firme), a forma de confirmar a dor antes de perguntar, as perguntas de validação no estilo SPIN (implicação, necessidade de solução), e as quebras de objeção. Capture frases reais boas para servir de molde.
4. Use esse material como PARÂMETRO ao escrever as mensagens do roteiro, principalmente na triagem e na prospecção. As mensagens geradas devem ter a mesma profundidade e o mesmo tom dos bons atendimentos, nunca mais rasas.

Exemplos do tipo de mensagem consultiva/validação que se costuma encontrar nesses bons atendimentos (use como referência de profundidade, adaptando ao tom real do assinante):

- "Deixa eu te perguntar uma coisa: se você acabasse escolhendo um caminho agora, por exemplo aderir a um parcelamento, e só depois descobrisse que parte desse débito poderia ter sido reduzida ou discutida, o quanto isso seria um problema pra você?"
- "Só preciso ser honesta com você, e é isso que diferencia o trabalho do escritório: ninguém sério consegue te dizer agora, por mensagem, quanto dá pra reduzir. Isso depende da origem do débito, da fase da execução e do que os documentos mostram. [...] Por isso te pergunto: faz sentido analisar o débito a fundo antes de decidir?"
- "Hoje, o que mais te preocupa nessa situação: o valor, o prazo, um possível bloqueio, ou a insegurança de decidir sem saber quais são as opções?"

Peça também exemplos de mensagens que o assinante considera boas para CADA função de persuasão, para você calibrar o tom de voz:
- problematização (mensagem que faz o lead enxergar o problema/risco real);
- reconhecimento do problema (mensagem que valida a dor e a torna concreta);
- prova social (casos semelhantes que o escritório já resolveu, sem prometer resultado);
- perguntas de validação/implicação (estilo SPIN, que levam o lead a concluir que precisa agir).

Se o assinante não tiver bons atendimentos nem exemplos para indicar (conta nova), avise que o tom será montado a partir do que ele descrever, e que vale revisar depois com base nos primeiros atendimentos reais.

### Passo 1.2: Definir o escopo (triagem, ou triagem + prospecção)

Antes de desenhar o fluxo, PERGUNTE ao assinante o escopo que ele quer para este serviço. Não assuma:

- **Só triagem**: empatia e conexão. Acolhe, cria vínculo, coleta os dados essenciais e tira algumas dúvidas factuais (via FAQ), qualifica o caso e encaminha. Não vende, não gera autoridade, não ancora valor. Mais leve.
- **Triagem + prospecção**: além da triagem, uma etapa comercial consultiva (SPIN) que gera autoridade (o método do escritório), prova social (FAQ "Exemplos de casos", sem prometer resultado) e faz a ancoragem de valor na dor/prejuízo do lead (o custo de não resolver), conduzindo à oferta.

Monte conforme a resposta. Se for só triagem, NÃO empurre prospecção. Se for triagem + prospecção, monte a triagem e, depois, o roteiro de prospecção/consulta que recebe a transferência (a prospecção nunca vem antes da triagem). A regra completa e os moldes de cada escopo estão na base de conhecimento: `regras/roteiro-geral.md`, e na biblioteca `roteiros/` o `exemplo-trabalhista-completo` é triagem e o `exemplo-tributario-completo` é triagem + prospecção.

### Passo 2: Verificar funcionalidades premium ativas

Pergunte o que o assinante já tem ativo, porque muda o que o roteiro pode usar:

- **Agendamento automático (Google Calendar)**: a IA agenda consultas verificando a disponibilidade real.
- **Painel CRM**: a IA cria e atualiza cards conforme o atendimento avança.
- **Resposta por áudio**: a IA responde por mensagem de voz; na instrução do passo, escreva `Responda por ÁUDIO neste passo.`

Se o assinante não tem uma funcionalidade, não a referencie no roteiro. Se não tem nenhuma, explique rapidamente como cada uma potencializaria o atendimento (ele pode se interessar).

### Passo 3: Definir o que é roteiro e o que é FAQ

Antes de escrever, separe o material coletado (regra completa em `regras/integracao-roteiro-faq` do manifest):

- Condução, perguntas do fluxo, coleta de dados, tom, transferência: **roteiro**.
- Informação factual e repetível (preços, prazos, documentos, objeções, exemplos de casos): **FAQ**.

Para todo tema factual importante que aparecer no roteiro, o passo correspondente deve **mandar a IA consultar a FAQ** (nominando a categoria), em vez de embutir o dado fixo. Caso clássico: exemplos de casos na prospecção vão para a FAQ e o roteiro instrui a IA a escolher o mais parecido com o caso do cliente.

Ao final, avise o assinante de tudo que você identificou como FAQ, para ele gerar a base com a skill `criar-faq-atendimento`.

### Passo 4: Validar o fluxo (tema e importância de cada passo) antes de escrever

PRIMEIRO construa e valide o ESQUELETO do fluxo com o assinante, antes de redigir qualquer mensagem. Liste, para cada roteiro, os passos na ordem e, para cada passo, apresente:
- o TEMA do passo (qual única informação ele coleta ou qual etapa de prospecção ele cumpre);
- POR QUE esse passo importa no atendimento (o papel dele na qualificação ou na conversão).

Peça ao assinante para confirmar, cortar ou reordenar os temas. Só depois que o esqueleto (temas + importância) estiver validado, parta para redigir as mensagens de cada passo. Isso evita refazer texto à toa e garante que cada passo tem propósito claro.

Use a estrutura recomendada: acolhimento, contextualização, coleta específica (um dado por passo), recapitulação/validação, e por fim o encaminhamento. Limites: 4 a 8 passos por roteiro (mais que isso cansa no WhatsApp). REGRA: o ÚLTIMO passo é EXCLUSIVAMENTE a ação de transferência/encerramento (`TRANSFER_HUMAN`, `TRANSFER_DEPARTMENT` ou `END_CONVERSATION`), SEM pergunta, para a IA não emendar duas perguntas ao transferir. Toda recapitulação ou validação fica em um passo próprio antes. Se faltar lugar para a costura na triagem, acrescente um passo a mais e mantenha o último só para transferir.

### Passo 4.0: Prospecção, costura com as dores e quebra de objeções

A prospecção não é um bloco genérico de venda: ela COSTURA com a triagem, usando as PRÓPRIAS dores que o lead informou como argumento de ancoragem. Ao montar a prospecção:

- Recupere as dores e dados que a triagem salvou (ex.: medo de bloqueio, venda travada, urgência) e use-os explicitamente nas mensagens de ancoragem do problema. A prospecção deve soar como continuação do que o lead já disse, não um discurso pronto.
- A prospecção PRECISA ter passos de QUEBRA DE OBJEÇÕES. Para escrevê-los bem, ENTREVISTE o assinante: pergunte como ELE responde a cada objeção comum. No mínimo cubra: "está caro", "não consigo pagar agora", "vou pensar", "já tenho advogado/contador", "vou ver depois / depois eu te chamo", "quanto eu vou conseguir reduzir/receber?".
- Para cada objeção, registre a resposta no tom do assinante (aprendido nos bons atendimentos), sempre sem prometer resultado, sem cravar valor que depende de análise, e reancorando na dor do lead. Quando o tema for factual (preço, prazo), o passo manda consultar a FAQ; para casos semelhantes, consultar a categoria de FAQ "Exemplos de casos".
- Cada objeção é tratada como uma etapa no formato de 4 pontos (com exemplos de resposta), e as variações trazem a referência principal + a variação.

HIERARQUIA DA PROSPECÇÃO (a prospecção tem etapas em ORDEM, e os passos internos de cada etapa também são ordenados). Ordem recomendada das etapas:

1. Recapitulação e validação (costura com a triagem: confirma a dor que o lead trouxe).
2. Ampliação de consciência / problematização (mostra o risco real, usando a dor do lead).
3. Pergunta de implicação (SPIN: o que acontece se não resolver).
4. Necessidade de solução (o lead conclui que precisa agir).
5. Apresentação do método e autoridade (como o escritório resolve + prova social de casos semelhantes, via FAQ "Exemplos de casos").
6. Oferta / proposta.
7. Quebra de objeções.
8. Fechamento ativo.
9. Encaminhamento (ação final do roteiro).

ORDEM INTERNA da etapa de quebra de objeção (cada objeção segue esta sequência, nesta ordem): (a) acolher a objeção sem confrontar; (b) validar que é legítima; (c) reancorar na dor que o lead já trouxe; (d) responder no tom do assinante, sem prometer resultado nem cravar valor; (e) reperguntar para retomar o avanço (ex.: "faz sentido pra você?"). Numere os passos internos de cada etapa e mantenha a hierarquia (etapa principal e seus sub-passos), para a IA seguir a ordem e não pular fases.

### Passo 4.1: Escolher o layout do passo

A plataforma tem dois layouts (detalhe em `references/formato-json-roteiro.md`):

- Roteiro de **serviço** (triagem, prospecção, temas): instrução no `responseText`.
- Roteiros **padrão** Recepção, Suporte e Agendamento: instrução no `userComplement`, com `responseText: ""`.

Se estiver ATUALIZANDO um roteiro existente, leia-o antes (`ler_roteiro`) e **mantenha o layout que ele já usa**. Recriou a Recepção ou o Suporte? Use o layout `userComplement`.

### Passo 4.2: Passo de classificação da Recepção (regra de clareza e não contradição)

O passo que identifica o motivo do contato e direciona (a classificação da Recepção) é o que mais erra quando fica confuso. Escreva-o assim:

UMA REGRA POR TEMA, EXPLÍCITA, no padrão fixo (uma por linha, sem parágrafo corrido):

```
Caso o contato informe que [sinais claros e específicos daquele tema], classifique o motivo do contato como [nome do tema] e transfira para [slug-do-roteiro].
```

Exemplo concreto (claro):
```
Caso o contato informe que tem clínica, é médico, faz parte de sociedade médica ou quer equiparação hospitalar, classifique o motivo como Clínica Médica e transfira para clinica_medica.
```

Evite o que estava confuso: um parágrafo único, longo, com vários temas, sinais e slugs misturados separados por ponto e vírgula. Isso espalha a probabilidade e a IA erra o destino. Quebre em uma frase por tema.

VERIFICAÇÃO DE NÃO CONTRADIÇÃO (obrigatória antes de entregar a Recepção): liste os temas e os sinais de cada um e confira que:

- Cada sinal aponta para UM único tema. Se um mesmo sinal (ex.: "imóveis") cabe em dois temas (Reforma vs. Holding), NÃO deixe nos dois: crie uma regra de desambiguação explícita (uma pergunta única antes de classificar) ou uma regra de prioridade.
- Existe regra de prioridade quando o lead cita mais de um tema ao mesmo tempo. Padrão recomendado: havendo qualquer menção a débito, dívida, cobrança, execução, autuação, protesto ou passivo, o tema é sempre Débitos, mesmo que cite outro tema junto.
- Existe um destino-padrão para o que não se encaixa (em geral Suporte ou a triagem consultiva).
- Os slugs citados existem de fato (confirme com `listar_roteiros`). Slug inexistente quebra a transferência.

Se achar duas regras que podem disparar para destinos diferentes com o mesmo sinal, isso é uma contradição: resolva antes de entregar.

### Passo 5: Gerar o JSON no formato exato

Gere o arquivo seguindo `references/formato-json-roteiro.md` à risca (estrutura, campos obrigatórios, campos do último passo). Pontos que mais quebram a importação:

- O JSON começa com `{ "flows": [ ... ] }`, nunca um array solto. Vários roteiros vão todos no mesmo `flows`.
- `responseText` é a instrução para o agente, não o texto que o cliente lê.
- No `TRANSFER_HUMAN`, a mensagem ao cliente vai em `transferResponse`, separada da instrução.
- `description` do roteiro precisa ser bem específica (a IA a usa para decidir quando ativar o roteiro).
- Salvar dados é correto e desejável (configure por passo com `saveData: true` + `dataKey`). A RECEPÇÃO deve salvar nome, motivo e o que o lead já trouxe, para os próximos roteiros não repetirem perguntas (regra do já-sabe). O que NÃO se faz é escrever uma "regra de salvamento global" em prosa na descrição ou repetida em todo passo: a descrição serve só para a IA saber quando ativar/rotear, não é instrução de atendimento.

REGRA OBRIGATÓRIA: sempre que o roteiro for de **triagem** ou de **prospecção** de um serviço, escreva cada item (triagem) ou cada etapa (prospecção) no **formato de 4 pontos**: Instrução, Como perguntar, Como responder, O que salvar e como.

REGRA AINDA MAIS IMPORTANTE, **UMA informação por passo**: cada passo (`question`) coleta UMA única informação e faz UMA única pergunta, salvando UM único campo. A plataforma só controla o avanço passo a passo; se você juntar vários dados ou várias perguntas num mesmo passo, a IA pode pular algum, porque não há controle dentro do passo. Então transforme cada dado em um passo separado. Exemplo do que NÃO fazer: um passo "Identificação" que coleta fase_cobranca, orgao_envolvido e tipo_pessoa de uma vez. Exemplo certo: um passo só para fase_cobranca, outro passo só para orgao_envolvido, outro só para tipo_pessoa, cada um com seus 4 pontos. As "variações" do Como perguntar são apenas formas diferentes de fazer a MESMA pergunta conforme o contexto (todas salvam o mesmo campo), nunca perguntas de campos diferentes.

Isso vale tanto ao criar do zero quanto ao atualizar um roteiro existente desses tipos. Para os demais tipos (recepção, suporte), antes de redigir, busque no manifest (lista `roteiros`) um exemplo do mesmo tipo/segmento e use como molde, adaptando ao negócio do assinante.

### Passo 5.1: Validar o JSON ANTES de entregar (obrigatório)

O importador do Zaia rejeita qualquer JSON fora do formato. Nunca entregue sem validar. Você roda em ambiente com execução de código: **salve o JSON em um arquivo e rode um parser** (ex.: `python3 -c "import json; json.load(open('arquivo.json'))"`) para garantir que é sintaticamente válido. Só entregue depois que passar.

Antes de entregar, percorra esta checklist (a mesma do importador):

- [ ] O JSON começa com `{ "flows": [` e termina com `] }`.
- [ ] Há um ÚNICO objeto `flows`, com TODOS os roteiros dentro do mesmo array (um único arquivo, nunca vários `flows` nem vários arquivos).
- [ ] Nunca é um array solto no topo (ex.: `[ { ... } ]`).
- [ ] Cada roteiro tem `name` (string não vazia), `description` (string) e `questions` (array com ao menos 1 passo).
- [ ] Cada passo tem `questionText` (não vazio), `responseText` (instrução para a IA, não texto ao cliente) e `saveData`.
- [ ] Todos os booleanos são `true`/`false` de verdade, nunca `"true"`/`"false"` entre aspas.
- [ ] O último passo de cada roteiro tem `completionAction` válido.
- [ ] O último passo é SÓ a transferência/ação: não faz pergunta nem coleta dado (a recapitulação/validação está num passo anterior).
- [ ] Em `TRANSFER_HUMAN`, a mensagem ao cliente está em `transferResponse` (não dentro de `responseText`).
- [ ] JSON sintaticamente válido (vírgulas, aspas e chaves balanceadas) confirmado pelo parser.
- [ ] Triagem/prospecção: cada passo coleta UMA só informação e faz UMA só pergunta (nenhum passo junta vários dados/campos).
- [ ] Se há roteiro de Recepção/classificação: cada tema tem UMA regra explícita ("Caso o contato informe X, classifique como Y e transfira para Z"), sem parágrafo corrido confuso.
- [ ] Recepção sem contradição: nenhum sinal aponta para dois temas sem regra de prioridade/desambiguação; há destino-padrão; todos os slugs de destino existem (`listar_roteiros`).

### Regra de saída (formato da entrega)

Entregue o resultado como um **arquivo `.json` puro**, pronto para importar. O conteúdo do arquivo deve ser **apenas o JSON**: sem texto antes ou depois, sem comentários, e **sem cercas de código markdown** (nada de ```json). Se for colar o JSON em algum lugar a pedido do assinante, cole o JSON cru, igual. O texto explicativo fica na sua mensagem do chat, nunca dentro do arquivo.

### Passo 6: Entregar e orientar próximos passos

Entregue o arquivo `.json` pronto para importar. Avise o assinante sobre:

- Criar as mídias na plataforma antes de vincular, se o roteiro referenciar mídia (senão dá erro na importação).
- Gerar as FAQs correspondentes (skill `criar-faq-atendimento`), em especial as categorias que o roteiro manda consultar.

## Arquivos de referência

A spec do formato fica embutida no plugin; o restante do conhecimento (regras e modelos) vem do índice central (manifest.json) da Za.ia, lido via WebFetch conforme a seção "Base de conhecimento e atualizacao".

- **`references/formato-json-roteiro.md`** (embutida): o formato JSON completo, todos os campos, regras do último passo e boas instruções por passo. Leia antes de gerar.
- **`regras/integracao-roteiro-faq`** (manifest): como roteiro e FAQ se encaixam e como escrever a consulta explícita à FAQ. Obrigatório.
- **biblioteca `roteiros/`** (manifest): modelos reais por segmento/tipo (recepção, coleta, e os que forem sendo adicionados) para copiar a forma.
- **`regras/roteiro-geral`, `regras/recepcao`, `regras/prospeccao`** (manifest): regras de escrita por tipo de roteiro (formato de 4 pontos, classificação da recepção sem contradição, hierarquia da prospecção e quebra de objeção).

## Observação de estilo

A Za.ia não usa travessão (—). Use vírgula, dois-pontos ou parênteses. Vale também para o texto dos roteiros gerados.
