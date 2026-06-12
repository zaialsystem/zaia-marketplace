# Novidades

Este e o canal "o que da pra fazer agora" do plugin. A Za.ia Legal System edita este arquivo a cada semana com as novidades, o que mudou, o que passou a funcionar, o que vale a pena experimentar. As rotinas leem este arquivo a cada run, entao e por aqui que as novidades chegam ate o assinante sem precisar reinstalar nada.

Cada entrada e datada e fica registrada abaixo, da mais recente para a mais antiga.

## 2026-06-12, v3.1.0

A pergunta de escopo (triagem, ou triagem + prospecção) agora está cravada no procedimento da skill criar-roteiro-atendimento (Passo 1.2), não só na base. É mudança estrutural: atualize o plugin em Customizar > Plugins para garantir.

## 2026-06-12, v3.0.3

O criar-roteiro-atendimento agora pergunta o ESCOPO antes de montar: só triagem, ou triagem + prospecção.

- **Triagem**: empatia e conexão, coleta os dados e tira dúvidas, sem vender (mais leve). Molde: exemplo-trabalhista-completo.
- **Prospecção** (depois da triagem, quando o assinante quer vender pela IA): autoridade + prova social + ancoragem de valor na dor/prejuízo. Molde: exemplo-tributario-completo.

## 2026-06-12, v3.0.2

Dois exemplos-ouro entraram na biblioteca de roteiros (anonimizados), para o criar-roteiro-atendimento partir de um molde real bem-feito:

- **exemplo-tributario-completo**: recepção com classificação em lista fechada, triagem de débito em 6 blocos e uma consulta consultiva (SPIN) completa.
- **exemplo-trabalhista-completo**: recepção separando cliente de lead, e triagem de rescisão indireta por tema (assédio, gestante, saúde, acidente, e outros), todos os passos em 6 blocos com recapitulação e validação.

## 2026-06-12, v3.0.1

Dois aprendizados do uso real entraram na base, e o criar-roteiro-atendimento já os aplica:

- **Classificação é lista fechada, não pergunta aberta**: todo passo de classificação/roteamento lista os motivos com gatilhos observáveis, em ordem (primeiro "já é cliente?"), mapeia cada um para um roteiro nominado, salva um campo de valores fechados e só conclui quando esse campo está preenchido. (regras/recepcao.md)
- **Todo passo de coleta conduz e termina em pergunta**: nada de passo "só de acolhimento" sem pergunta (a empatia vira a primeira linha do passo que já pergunta); o fechamento é recapitulação + validação terminando em pergunta antes do TRANSFER_HUMAN. (regras/roteiro-geral.md)

## 2026-06-09, v3.0.0

O plugin esta com sete skills funcionando. Em uma linha cada:

- **configurar-zaia**: faz o setup inicial do assinante (conecta o que precisa, registra as preferencias e liga ou desliga as rotinas).
- **diagnosticar-atendimento-ia**: explica por que a IA respondeu de um jeito e ensina a ajustar o roteiro ou a FAQ para o erro nao se repetir.
- **criar-roteiro-atendimento**: cria roteiros de atendimento por IA, entrevistando o assinante sobre o negocio antes de gerar o fluxo pronto para importar.
- **criar-faq-atendimento**: monta a base de conhecimento (perguntas e respostas) que a IA consulta durante o atendimento.
- **recomendar-faq-da-semana**: olha os atendimentos da semana e sugere FAQs novas a partir do que mais se repetiu nas conversas.
- **rotina-analise-diaria**: varre o dia e entrega ao assinante pendencias, contatos em silencio, follow-ups e lembretes, ja com rascunhos de mensagem para aprovar.
- **automacao-pecas**: detecta atendimentos de coleta concluidos e gera a peca (em .docx) para o assinante revisar e assinar.

Nota de manutencao: edite este arquivo a cada semana com novidades; mudanca estrutural de skill sobe pluginVersion no manifest.
