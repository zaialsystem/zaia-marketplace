---
name: criar-faq-atendimento
description: Cria a base de conhecimento (FAQ) do agente de IA no formato JSON da plataforma (chave "faqs"), incluindo categorias, objeções e a categoria de exemplos de casos consultável pelos roteiros. Use SEMPRE que o assinante disser "criar FAQ", "montar a base de conhecimento", "cadastrar perguntas frequentes", "a IA precisa saber responder sobre X", "adicionar respostas de preço/prazo/documentos", "criar objeções", "exemplos de casos para a IA usar", ou qualquer variação que envolva produzir entradas de FAQ para o agente consultar durante o atendimento. É complementar à skill `criar-roteiro-atendimento` (gere as duas juntas ao montar um serviço, garantindo que toda categoria que o roteiro manda consultar exista no FAQ) e à `recomendar-faq-da-semana` (que sugere novas FAQs a partir das conversas).
---

# Criar FAQ do atendimento por IA

Esta skill gera a base de conhecimento (FAQ) no formato da plataforma (JSON com a chave `faqs`), pronta para importar. A FAQ é o conhecimento que o agente consulta durante o atendimento para responder dúvidas sem sair do fluxo do roteiro.

Público: o assinante, em geral não técnico. Conduza com clareza e gere o JSON só depois de entender o negócio e os serviços.

## Base de conhecimento e atualizacao (consultar SEMPRE no inicio)

Antes de agir, leia o indice central da Za.ia Legal System (atualizado sem reinstalar o plugin). Use WebFetch para ler:

`https://raw.githubusercontent.com/zaialsystem/zaia-marketplace/main/knowledge/manifest.json`

Do manifest:
1. Para cada arquivo relevante A ESTA skill (filtre pelo campo `aplica`, e por segmento/tipo/categoria quando houver), monte a URL com `rawBase + path` e leia via WebFetch SO o que precisar. Para esta skill, os arquivos que costuma usar sao: a integracao roteiro/FAQ `regras/integracao-roteiro-faq` e a biblioteca `faqs/` (use as FAQs de exemplo como modelo de estilo e estrutura, adaptando o conteudo ao negocio do assinante).
2. Em conflito com o conteudo embutido neste plugin, o conteudo do GitHub PREVALECE; o embutido e so fallback minimo. Nao invente conteudo de arquivo que nao foi lido.
3. Se o fetch falhar, siga com o fallback minimo embutido e avise o assinante que esta sem a base atualizada.

Aviso de atualizacao: leia a versao instalada em `${CLAUDE_PLUGIN_ROOT}/.claude-plugin/plugin.json` e compare com `manifest.pluginVersion`. Se o manifest for maior, avise em linguagem simples: "Saiu uma atualizacao do plugin Za.ia. Veja as novidades e atualize em Customizar > Plugins." (cite o resumo de `novidades.md`). Se `manifest.skills` listar uma skill que voce nao tem instalada, avise que ha skill nova a instalar pelo mesmo caminho.

## Como a FAQ funciona na plataforma

- **Pipeline**: ao receber uma mensagem, o agente consulta as FAQs para achar a melhor resposta.
- **Busca semântica**: o sistema acha a FAQ certa mesmo sem as mesmas palavras. Por isso escreva as perguntas em linguagem natural, do jeito que o cliente fala.
- **Categorias**: organizam e ajudam a IA a contextualizar (ex.: pergunta sobre custo, prioriza a categoria "Financeiro").
- **Objeções**: hesitações ("é caro", "vou pensar") são FAQs da categoria "Objeções", respondidas de forma acolhedora e persuasiva.
- **Complemento ao roteiro**: o roteiro coleta e conduz, a FAQ responde dúvidas. Trabalham juntos (ver `regras/integracao-roteiro-faq` do manifest).

## Workflow

### Passo 1: Entender o negócio e mapear dúvidas

Pergunte (uma por vez): áreas e serviços, valores e formas de pagamento, prazos, documentos por tipo de caso, dados institucionais (advogados, endereço, horários), e, para cada serviço, as dúvidas que os clientes mais fazem. Pergunte também as objeções reais ("o que o cliente fala para não fechar?").

### Passo 2: Decidir o que é FAQ (e o que é roteiro)

Vai para a FAQ: informação factual e repetível (escritório, valores, prazos, documentos, dúvidas por serviço, objeções, exemplos de casos). Vai para o roteiro: condução, perguntas do fluxo, tom, transferência. Regra completa em `regras/integracao-roteiro-faq` do manifest.

### Passo 3: Garantir as categorias que o roteiro consulta

Este é o ponto de integração. Se houver roteiros, verifique quais categorias eles mandam consultar (ex.: "Exemplos de casos", "Financeiro") e **garanta que cada uma exista na FAQ com conteúdo suficiente**. Uma instrução de consulta no roteiro sem a categoria correspondente no FAQ resulta em IA sem resposta. Crie sempre:

- Uma **categoria por serviço** (ex.: "Rescisão Trabalhista") para as dúvidas específicas daquele serviço.
- A categoria **"Objeções"** com as hesitações comuns.
- A categoria **"Exemplos de casos"** quando os roteiros de prospecção forem consultá-la: uma entrada por caso, descrevendo o perfil e o desfecho em linguagem que a IA possa casar com a situação do cliente (ver o padrão de ouro em `regras/integracao-roteiro-faq` do manifest).

### Passo 4: Escrever boas FAQs

- Pergunta específica e natural: "Quanto custa uma consulta trabalhista?" é melhor que "Quanto custa?".
- Resposta completa mas enxuta (1 a 3 parágrafos), linguagem acessível, tom acolhedor.
- Inclua o próximo passo quando fizer sentido ("traga RG e CPF na consulta").
- A resposta guarda o dado; a conduta de como entregar é do roteiro. Não misture.

### Passo 5: Gerar o JSON no formato exato

Siga `references/formato-json-faq.md`. Gere o arquivo, cobrindo as categorias relevantes e pelo menos 5 objeções. Retorne o arquivo pronto para importar.

### Passo 5.1: Validar antes de entregar (obrigatório)

Valide o JSON antes de entregar, igual ao roteiro. Salve em arquivo e rode um parser (`python3 -c "import json;json.load(open('arquivo.json'))"`). Checklist:

- [ ] Começa com `{ "faqs": [` e termina com `] }`; um único array `faqs`.
- [ ] Cada entrada tem `questionText` (não vazio); `responseText` recomendado; `category` (string).
- [ ] JSON válido (vírgulas, aspas, chaves), sem cercas markdown e sem texto fora do JSON.

Entregue como arquivo `.json` puro (somente o JSON, sem ```json e sem texto antes/depois). O texto explicativo vai na mensagem do chat, nunca no arquivo.

### Passo 6: Fechar o ciclo com o roteiro

Avise o assinante de quais categorias foram criadas e confirme que batem com o que os roteiros mandam consultar. Se faltou alguma ponte, sinalize para ajustar o roteiro (skill `criar-roteiro-atendimento`) ou completar a FAQ.

## Arquivos de referência

A spec do formato fica embutida no plugin; o restante vem do índice central (manifest.json) da Za.ia, lido via WebFetch conforme a seção "Base de conhecimento e atualizacao".

- **`references/formato-json-faq.md`** (embutida): o formato JSON completo, campos e categorias sugeridas. Leia antes de gerar.
- **`regras/integracao-roteiro-faq`** (manifest): como FAQ e roteiro se encaixam, e o padrão de exemplos de casos consultáveis. Obrigatório.
- **biblioteca `faqs/`** (manifest): FAQs de exemplo (exemplos de casos, preços e prazos) para usar como modelo de estilo e estrutura.

## Observação de estilo

A Za.ia não usa travessão (—). Use vírgula, dois-pontos ou parênteses. Vale também para o texto das FAQs geradas.
