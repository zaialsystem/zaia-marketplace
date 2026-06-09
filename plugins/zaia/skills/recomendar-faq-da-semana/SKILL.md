---
name: recomendar-faq-da-semana
description: Lê os atendimentos de um período (a semana, por padrão) pela plataforma e recomenda novas FAQs a partir das informações que se repetiram nas conversas, ou que a equipe teve que informar manualmente. Use SEMPRE que o assinante disser "o que virou FAQ essa semana", "analisa os atendimentos da semana", "que perguntas se repetiram", "o que a IA não soube responder", "sugere FAQs novas", "minerar FAQ das conversas", "revisão semanal do atendimento", ou qualquer variação que envolva olhar o conjunto de atendimentos para descobrir o que falta na base de conhecimento. Também dispara automaticamente quando rodar como tarefa agendada (ex.: toda sexta). É complementar à `criar-faq-atendimento` (que formaliza as FAQs sugeridas) e à `diagnosticar-atendimento-ia` (que trata um atendimento isolado).
---

# Recomendar FAQs da semana

Esta skill faz a varredura dos atendimentos de um período e devolve uma lista de FAQs sugeridas: perguntas que se repetiram, informações que a equipe teve que dar manualmente, e temas em que a IA improvisou ou desviou por falta de conhecimento cadastrado. O objetivo é transformar o que aconteceu nas conversas em conhecimento reutilizável, fechando o ciclo de melhoria contínua.

Público: o assinante. Entregue uma lista clara e acionável, pronta para virar FAQ.

## Pré-requisito

Esta skill funciona ao vivo pelo conector (MCP) da plataforma. Sem ele, não há como ler os atendimentos. Ferramentas usadas: `listar_atendimentos_ia`, `ler_atendimento_ia`, `ler_conversa`, e `buscar_faq` (para checar o que já existe).

## Workflow

### Passo 1: Definir o período

Padrão: últimos 7 dias. Se o assinante pedir outro intervalo, use o pedido. Em execução agendada, use a semana corrente.

### Passo 2: Levantar os atendimentos do período

Use `listar_atendimentos_ia` com o `periodo` (início e fim). Priorize os atendimentos com sinais de lacuna: status de erro, conversas longas, ou casos que terminaram em transferência para humano logo após uma pergunta factual. Para detalhar, use `ler_atendimento_ia` (o trace mostra quando a IA não achou FAQ) e `ler_conversa` (a pergunta real do cliente).

### Passo 3: Identificar candidatos a FAQ

Procure padrões, não casos isolados:

- **Perguntas factuais repetidas** por clientes diferentes (preço, prazo, documento, como funciona).
- **Informação que a equipe humana teve que dar** depois de assumir, que poderia ter sido respondida pela IA.
- **Temas em que a IA improvisou ou desviou** por falta de FAQ (o trace ajuda: ausência de FAQ consultada, ou consulta sem resultado).
- **Objeções recorrentes** que travaram a contratação.

Para cada candidato, confirme com `buscar_faq` se já não existe uma FAQ cobrindo o tema (evita duplicar). Se existir mas a IA mesmo assim errou, o problema pode ser a FAQ fraca ou o roteiro não pedir a consulta: nesse caso, sinalize para a skill `diagnosticar-atendimento-ia`.

### Passo 4: Priorizar

Ordene por frequência e impacto: o que apareceu mais vezes e o que mais atrapalhou o atendimento vem primeiro. Não devolva uma lista enorme: foque nos candidatos que realmente se repetiram.

### Passo 5: Entregar as sugestões no formato padrão

Para cada FAQ sugerida, entregue:

```
### [Tema da FAQ]
- Quantas vezes apareceu no período e em quantos atendimentos.
- Pergunta sugerida (linguagem natural do cliente): "..."
- Resposta sugerida (rascunho, a ser validado pelo assinante): "..."
- Categoria sugerida: [...]
- Evidência: IDs das conversas onde apareceu.
- Já existe FAQ parecida? [não / sim, mas fraca / sim, ajustar roteiro para consultar]
```

Ao final, ofereça formalizar as escolhidas como JSON pronto para importar, via skill `criar-faq-atendimento`, e sinalizar para a `diagnosticar-atendimento-ia` os casos em que o problema não é falta de FAQ, e sim a ponte roteiro/FAQ.

## Rodar toda semana automaticamente

Esta skill é boa candidata a tarefa agendada (ex.: toda sexta de manhã). Quando rodar agendada, faça o período ser a semana corrente e entregue a lista priorizada. Se o assinante quiser, ofereça configurar o agendamento.

## Observação de estilo

A Andreza não usa travessão (—). Use vírgula, dois-pontos ou parênteses.
