# Mineração de FAQ (descobrir FAQs novas nas conversas)

Minerar FAQ é olhar o conjunto de atendimentos de um período (a semana, por padrão) e descobrir o que falta na base de conhecimento. O objetivo é transformar o que se repetiu nas conversas em conhecimento reutilizável, fechando o ciclo de melhoria contínua. Nada é cadastrado sem o assinante aprovar.

## Como identificar temas repetidos

Procure padrões, não casos isolados. Sinais de que um tema merece virar FAQ:

- **Perguntas factuais repetidas** por clientes diferentes (preço, prazo, documentos, como funciona).
- **Informação que a equipe humana teve que dar** depois de assumir, que poderia ter sido respondida pela IA.
- **Temas em que a IA improvisou ou desviou** por falta de conhecimento cadastrado (o trace ajuda: ausência de FAQ consultada, ou consulta sem resultado).
- **Objeções recorrentes** que travaram a contratação.

Priorize os atendimentos com sinais de lacuna: status de erro, conversas longas, ou casos que terminaram em transferência para humano logo após uma pergunta factual.

## Critério para virar FAQ: factual e repetível

Só vira FAQ o que é **factual e repetível**: preço, prazo, documentos, horário, como funciona. A resposta é sempre a mesma, independente do momento da conversa.

O que NÃO é FAQ: **condução e comportamento são roteiro, não FAQ.** Como acolher, em que ordem perguntar, quando transferir, o tom, o que pode ou não dizer, como apresentar um preço sem pressionar, tudo isso é roteiro. Se a resposta muda conforme o momento da conversa, é roteiro.

## Agrupar variações da mesma pergunta

Junte numa entrada só todas as formas de perguntar a mesma coisa. "Quanto custa", "qual o valor", "quanto fica", "tem desconto" são a mesma dúvida de preço: uma entrada de FAQ, não quatro. Nomeie a categoria do tema (ex.: Preços, Prazos, Documentos, Exemplos de casos) para a IA encontrar a entrada.

## Priorizar

Ordene por **frequência** (o que apareceu mais vezes, em mais atendimentos) e por **impacto** (o que mais atrapalhou o atendimento, em especial onde a IA respondeu mal ou improvisou). O que mais apareceu e o que mais atrapalhou vem primeiro. Não devolva uma lista enorme: foque nos candidatos que realmente se repetiram.

Antes de sugerir, confirme com `buscar_faq` se já não existe uma FAQ cobrindo o tema (evita duplicar). Se existir mas a IA mesmo assim errou, o problema pode ser FAQ fraca ou o roteiro não pedir a consulta: nesse caso, é caso de diagnóstico e ajuste da ponte roteiro/FAQ, não de FAQ nova (ver `integracao-roteiro-faq.md`).

## Saída sugerida

Entregue uma lista clara e acionável de FAQs novas para o assinante revisar, cada uma no formato `{ questionText, responseText, category }`:

```
- questionText: a pergunta na linguagem natural do cliente.
- responseText: a resposta sugerida (rascunho, a ser validado pelo assinante).
- category: a categoria do tema.
```

Para cada sugestão, é útil informar também quantas vezes apareceu e em quantos atendimentos, os IDs das conversas onde apareceu (evidência), e se já existe FAQ parecida (não / sim, mas fraca / sim, ajustar roteiro para consultar).

Nada é cadastrado sem o assinante aprovar. Ao final, ofereça formalizar as escolhidas em JSON pronto para importar.
