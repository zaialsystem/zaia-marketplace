# Za.ia para Claude

Plugin oficial da Za.ia Legal System para quem é assinante da plataforma de atendimento por IA da Za.ia (https://zaials.com.br). Ele ajuda o escritório a refinar e operar o atendimento por IA no WhatsApp: configurar, diagnosticar atendimentos, criar roteiros e FAQs no formato da plataforma, recomendar FAQs novas, rodar a análise diária de pendências e gerar peças a partir dos atendimentos.

Princípio que orienta todas as skills: a IA é probabilística, e o comportamento dela é consequência do roteiro e da FAQ. Por isso a correção proposta é sempre no roteiro ou na FAQ do próprio escritório.

## Requisitos

- Conta ativa na Za.ia (https://app.zaials.com.br).
- O conector da Za.ia, que vem junto com o plugin (veja "O que o plugin acessa"). No primeiro uso o Claude pede o login da sua conta Za.ia e a sua autorização.

## As 7 skills

- `configurar-zaia`: configuração inicial. Você decide onde guardar a configuração, quais partes da análise rodar, horários, limiares e o modelo das peças.
- `diagnosticar-atendimento-ia`: explica por que a IA respondeu de um jeito e mostra como ajustar o roteiro ou a FAQ para o erro não se repetir.
- `criar-roteiro-atendimento`: entrevista você sobre o serviço e gera o roteiro em JSON, pronto para importar na plataforma.
- `criar-faq-atendimento`: monta a base de conhecimento (FAQ) em JSON, pronta para importar.
- `recomendar-faq-da-semana`: lê os atendimentos da semana e sugere FAQs novas a partir do que mais se repetiu.
- `rotina-analise-diaria`: lista pendências, contatos em silêncio, follow-ups e lembretes do dia, com rascunhos de mensagem para você aprovar.
- `automacao-pecas`: gera a peça em `.docx` a partir de um atendimento de coleta concluído, usando os seus próprios modelos, para você revisar e assinar.

## O que o plugin acessa

Tudo o que o plugin lê, grava ou busca está listado aqui.

- **Conector da Za.ia** (`https://mcp.zaials.com.br/mcp`): servidor MCP oficial da Za.ia, com login OAuth na sua conta. As skills usam esse conector para ler conversas, contatos, CRM, roteiros e FAQs do seu escritório. Nada é criado ou alterado na Za.ia (mensagem, tarefa, etiqueta, agendamento) sem a sua aprovação na conversa.
- **Base de conhecimento pública da Za.ia**: as skills leem regras, exemplos e novidades em `https://raw.githubusercontent.com/zaialsystem/zaia-marketplace/main/knowledge/`, começando por `knowledge/manifest.json`. É conteúdo público e só de leitura; nenhum dado do seu escritório é enviado para lá.
- **Arquivos no seu computador**: a configuração e o estado das rotinas ficam em `~/.zaia-skills/config-pointer.json` e na pasta que você escolher no `configurar-zaia`. As peças geradas são salvas como `.docx` na pasta que você definir.

O plugin não traz programas próprios: só instruções (skills) e a referência ao conector. Para gerar o `.docx`, a skill `automacao-pecas` usa as ferramentas de documento que o próprio Claude já tem. O plugin não envia mensagem a nenhum cliente por conta própria e não manda dados para nenhum outro lugar além dos listados acima.

## Primeiros passos

1. Instale o plugin e autorize o conector da Za.ia com o login da sua conta.
2. Peça ao Claude: "configurar a Za.ia". A skill `configurar-zaia` faz as perguntas e grava a sua configuração.
3. Depois use as skills pelo que você precisa, por exemplo "analisa esse atendimento" ou "cria um roteiro para o serviço de aposentadoria".

## Suporte

- E-mail: contato@zaials.com.br
- Site: https://zaials.com.br
- Política de privacidade: https://zaials.com.br/politica-de-privacidade
- Termos de uso: https://zaials.com.br/termos-de-uso

## Licença

Uso exclusivo de assinantes ativos da Za.ia. Veja o arquivo `LICENSE`.
