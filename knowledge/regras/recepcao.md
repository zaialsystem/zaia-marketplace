# Recepção

A recepção recebe o cliente, descobre o motivo do contato e encaminha para o roteiro certo. Ela quase não resolve nada sozinha: o trabalho dela é classificar bem. Erro de recepção contamina todo o resto (ver causa 9 em `../causas-raiz/catalogo.md`).

## O que documentar sempre

- **Saudação e identificação**: como a IA se apresenta (nome do atendente, empresa/escritório), com saudação conforme o horário quando fizer sentido.
- **Coleta do nome**: pergunta única e objetiva.
- **A recepção salva dados**: salve nome, motivo e qualquer dado que o lead já tenha dado (mesmo fora de ordem), com `saveData: true` e a `dataKey` certa. Isso evita que os roteiros seguintes repitam perguntas (a regra do já-sabe reaproveita esses dados).
- **Motivo do contato**: pergunta aberta para descobrir o que a pessoa quer, sem aprofundar nem oferecer serviço ainda.
- **Motivos de contato bem documentados (o ponto mais importante)**: liste cada destino possível e, para cada um, deixe claro quando mandar para lá, com exemplos de frases reais do cliente que disparam aquele destino. Quanto mais explícito o gatilho, menos a IA erra o encaminhamento.
- **Destino-padrão na dúvida**: sempre defina para onde vai o que não se encaixa (em geral, Suporte / humano).

## Nome do cliente: facultativo por padrão

Normalmente a IA acerta o nome, então NÃO force por padrão. Use só se a IA estiver errando o nome (usando o pushname): aí torne o passo de nome obrigatório (perguntar e salvar `nome_cliente`, sem usar o pushname) e explique o motivo no passo. Exemplo de motivo: o pushname pode vir como "Juuu" e a IA acaba chamando a pessoa de "Juuu".

## Passo de classificação: uma regra por tema, sem contradição

O passo que identifica o motivo do contato e direciona (a classificação da Recepção) é o que mais erra quando fica confuso. Escreva a classificação como UMA regra por tema, explícita, no padrão fixo (uma por linha, sem parágrafo corrido):

```
Caso o contato informe que [sinais claros e específicos daquele tema], classifique o motivo do contato como [nome do tema] e transfira para [slug-do-roteiro].
```

Exemplo concreto (claro):

```
Caso o contato informe que tem clínica, é médico, faz parte de sociedade médica ou quer equiparação hospitalar, classifique o motivo como Clínica Médica e transfira para clinica_medica.
```

Evite o que costuma ficar confuso: um parágrafo único, longo, com vários temas, sinais e slugs misturados separados por ponto e vírgula. Isso espalha a probabilidade e a IA erra o destino. Quebre em uma frase por tema.

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

## Verificação de não contradição (obrigatória antes de entregar a Recepção)

Liste os temas e os sinais de cada um e confira que:

- **Cada sinal aponta para UM único tema.** Se um mesmo sinal (ex.: "imóveis") cabe em dois temas (Reforma vs. Holding), NÃO deixe nos dois: crie uma regra de desambiguação explícita (uma pergunta única antes de classificar) ou uma regra de prioridade.
- **Existe regra de prioridade** quando o lead cita mais de um tema ao mesmo tempo. Padrão recomendado: havendo qualquer menção a débito, dívida, cobrança, execução, autuação, protesto ou passivo, o tema é sempre Débitos, mesmo que cite outro tema junto.
- **Existe um destino-padrão** para o que não se encaixa (em geral Suporte ou a triagem consultiva).
- **Os slugs citados existem de fato** (confirme com `listar_roteiros`). Slug inexistente quebra a transferência.

Se achar duas regras que podem disparar para destinos diferentes com o mesmo sinal, isso é uma contradição: resolva antes de entregar.
