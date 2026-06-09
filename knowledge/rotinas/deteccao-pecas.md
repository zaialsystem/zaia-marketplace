# Deteccao de pecas a gerar (heuristicas)

Esta rotina descobre, sozinha, quando um atendimento de coleta terminou e ja da para gerar a peca correspondente para o assinante. Ela varre os atendimentos do roteiro de coleta configurado, identifica os que concluiram, e gera a peca apenas para os novos. Como sempre na Za.ia: nada e escrito na plataforma, a peca vai para o assinante revisar e assinar.

## Passo 1: buscar os atendimentos do roteiro configurado (somente leitura)

Toda a leitura acontece pelo banco da plataforma (MCP da Za.ia), nunca pela API do WhatsApp direto. No setup, o assinante configura qual roteiro de coleta alimenta a geracao de pecas (por exemplo, um roteiro que coleta os dados de uma procuracao). A rotina busca os atendimentos desse roteiro com:

- `listar_atendimentos_ia`: lista os atendimentos conduzidos pela IA. O campo `departamento` de cada atendimento indica o roteiro (o slug) por onde a conversa passou, entao e por ele que a rotina filtra os atendimentos do roteiro configurado.
- `listar_conversas`: lista as conversas, util como apoio para cruzar e localizar o atendimento certo.

A rotina so olha os atendimentos do slug configurado pelo assinante. Atendimento de outro roteiro nao entra.

## Passo 2: saber que a coleta concluiu

Nem todo atendimento do roteiro virou peca: muitos ainda estao no meio da conversa. A rotina so considera os que de fato terminaram a coleta. Como reconhecer que concluiu:

- **Chegou ao passo final do roteiro de coleta**: o atendimento percorreu o roteiro ate o ultimo passo, ou seja, a IA terminou de coletar tudo o que aquele roteiro pede.
- **Os dados necessarios estao preenchidos**: os campos que a peca precisa (os dataKeys que o roteiro coleta) estao todos presentes no atendimento. Se ainda falta um dado obrigatorio, a coleta nao concluiu de verdade, mesmo que a conversa pareca ter acabado.

Use os dois sinais juntos: chegou ao fim do roteiro e os dataKeys necessarios estao la. So entao o atendimento e candidato a gerar peca.

## Passo 3: cruzar com o estado local para gerar so as novas

A rotina mantem, no lado do assinante (no workspace dele), um registro local das conversas que ja geraram peca. Esse estado e sempre local, nunca gravado na Za.ia. Antes de gerar, a rotina cruza os atendimentos concluidos com esse registro:

- Atendimento concluido que ainda nao esta no registro local: peca nova, gera.
- Atendimento concluido que ja esta no registro: ja foi gerado antes, pula.

Assim a rotina e idempotente (rodar de novo nao gera a mesma peca duas vezes) e tolerante a tick perdido (se um run nao aconteceu, o proximo pega os atendimentos que concluiram nesse meio tempo, porque a deteccao olha o estado atual e nao depende de ter rodado em todo intervalo). A leitura tambem e incremental a partir do cursor local, processando o que mudou desde o ultimo run.

## Passo 4: gerar a peca e entregar ao assinante

Para cada atendimento novo e concluido, a rotina gera a peca seguindo a metodologia de geracao (extrai os dados do atendimento sem deduzir, usa o modelo do proprio assinante, sinaliza o que faltar). A peca sai em .docx e vai para o assinante revisar e assinar.

## Lembrete que vale para a rotina inteira

Nada e escrito na Za.ia. A rotina so le (atendimentos e conversas) pela plataforma e mantem o seu estado local no lado do assinante. A peca gerada e entregue ao assinante, nunca enviada ao cliente, e nenhuma acao na plataforma acontece sem o assinante aprovar.
