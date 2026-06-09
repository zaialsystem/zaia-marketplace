# Mecânica da plataforma e leitura correta do trace

Leia ANTES de interpretar qualquer trace (`ler_atendimento_ia`). Vários erros de diagnóstico nascem de ler o trace como se a IA decidisse coisas que, na verdade, foram um humano ou a mecânica da plataforma. Este arquivo evita esses enganos.

## Fuso horário (corrija sempre)

Os timestamps do trace e da conversa vêm em **UTC** (terminam com "Z", ex.: `2026-06-09T00:34:19Z`). O escritório opera no horário de **Brasília (America/Sao_Paulo, UTC-3)**. Sempre **subtraia 3 horas** ao citar horários para o assinante. Exemplo: `00:34Z` é **21:34** em Brasília; `23:19Z` é **20:19**. Nunca cite o horário UTC cru: o assinante vai dizer que está errado. (O Brasil não tem horário de verão desde 2019, então é UTC-3 o ano todo.)

## Gatilhos rápidos (departamento começando com `//`)

Quando o campo `departamento` de uma execução começa com `//` (ex.: `//debito`, `//liziane`), **isso é um gatilho rápido (ação rápida) disparado MANUALMENTE por um membro da equipe**, não uma decisão da IA e não um roteiro de verdade.

- Não diga que "a IA pulou para o departamento X" nem que "a IA inventou um departamento". Diga que "um membro acionou um gatilho rápido".
- Sinal no `pensamento`: textos como "A ação rápida foi acionada para dar continuidade ao atendimento sem esperar uma nova mensagem do cliente".
- Gatilhos rápidos normalmente **não têm passos nem regra anti-loop**, então a IA, ao ser jogada neles, perde a blindagem do roteiro e tende a emitir transição genérica (filler). Quando um gatilho rápido aparece no meio do fluxo, ele costuma ser **a causa** do desvio, não um sintoma da IA.
- Correção típica: parar de usar gatilhos rápidos para conduzir o fluxo e deixar a Recepção/roteiros transferirem sozinhos; ou, se o gatilho for necessário, dar a ele passos e a regra anti-filler.

## Mensagem de membro/WhatsApp DESATIVA a IA (efeito "duas vozes")

Atenção, a lógica é o contrário do intuitivo: **quando um membro responde, ou quando alguém responde pelo WhatsApp (web/mobile), a IA é DESATIVADA** naquela conversa. Ou seja, assim que um humano entra, a IA para sozinha.

Então, se no trace a IA voltou a responder DEPOIS de um humano ter assumido, **é porque alguém a REATIVOU MANUALMENTE** (em geral disparando um gatilho rápido `//` ou religando a IA no painel). O efeito "duas vozes" (IA respondendo por cima do humano) **não é automático**: é consequência de reativações manuais repetidas.

- Não diga que "a mensagem do membro religou a IA". Diga que "a IA foi reativada manualmente" (e, quando houver `//` no `departamento`, foi via gatilho rápido).
- Correção: para a conversa ficar só com o humano, basta **não reativar a IA** enquanto estiver atendendo (a resposta do membro já a desativa). Se quiser uma trava extra contra reativação acidental, use uma etiqueta de "atendimento humano" que bloqueie a IA. Em todo caso, é configuração/operação, não erro de roteiro.

## Como ler os campos do trace (não tire conclusão errada)

- **`departamento`**: a fonte de verdade de qual roteiro (ou gatilho rápido) estava ativo naquela execução. Use ESTE campo para reconstruir o caminho, em ordem cronológica (o trace vem do mais recente para o mais antigo, então leia de baixo para cima).
- **`fluxoSeguido`**: frequentemente vem `null` MESMO quando a IA estava claramente dentro de um roteiro (aparece null até em execuções de `recepcao`). **Não conclua "a IA nunca entrou no roteiro" a partir de `fluxoSeguido: null`.** Esse campo não é confiável como prova de ancoragem.
- **`dadosColetados`**: vem `{}` quando o roteiro não tem `dataKeys` configuradas naquele passo, ou quando a IA não preencheu. Vazio **não significa** necessariamente que a IA ignorou o passo. Confirme olhando se o roteiro define `dataKeys` (via `ler_roteiro`); se não define, não há o que coletar.
- **`faqConsultada`**: se vazio em todas as execuções, a IA não consultou a FAQ. Aí cabe checar a ponte roteiro/FAQ (ver `integracao-roteiro-faq.md`).
- **`pensamento.texto`**: mostra o raciocínio e costuma revelar se foi ação rápida, se a IA achou que "já respondeu antes", etc.
- **`eventosGestao`**: criação/atualização de card no CRM, útil para ver a progressão comercial.

## Por que a Recepção às vezes não transfere sozinha

A transferência costuma ser a ação de conclusão (`TRANSFER_DEPARTMENT`) do ÚLTIMO passo da Recepção. Ela só dispara quando a IA conclui o passo de classificação e avança para o passo de transferência. Se a IA **responde a dúvida do cliente dentro do passo de classificação** (em vez de confirmar o tema em uma frase e concluir), ela nunca chega ao passo de transferência, e o caso "trava" na Recepção até alguém forçar um gatilho rápido.

Ao diagnosticar isso, verifique no `ler_roteiro` da recepção se o passo de classificação proíbe explicar o processo (costuma proibir). Se proíbe e a IA explicou mesmo assim, a causa-raiz é "instrução não respeitada porque o passo não conclui de forma determinística": a correção é reforçar que, ao identificar o tema, a IA **não emite mais nada além de uma frase de confirmação e conclui o passo**, e considerar encurtar o passo para a regra não ficar enterrada (ver "Instrução enterrada" em `causas-raiz.md`).

## Checklist de mecânica (rode antes de concluir o diagnóstico)

1. Converti todos os horários de UTC para Brasília (menos 3h)?
2. Algum `departamento` começa com `//`? Então foi gatilho rápido manual de um membro, não decisão da IA.
3. A IA voltou a responder depois de um humano? Então foi REATIVADA MANUALMENTE (mensagem de membro/WhatsApp desativa a IA). O efeito "duas vozes" vem de reativações manuais, não é automático.
4. Reconstruí o caminho pelo campo `departamento` (de baixo para cima), e não por `fluxoSeguido`?
5. `dadosColetados` vazio: confirmei se o roteiro tem `dataKeys` antes de dizer que a IA não coletou?
