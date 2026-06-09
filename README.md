# Marketplace Za.ia (zaia-loverly)

Marketplace de skills (`zaia-loverly`) com o plugin `zaia` (7 skills) para o assinante da plataforma de atendimento por IA da Za.ia (Za.ia Legal System).

O plugin ajuda o assinante a refinar e operar o atendimento de ponta a ponta: configurar, diagnosticar, criar roteiros e FAQs, recomendar FAQs novas, rodar a análise diária de pendências e automatizar a geração de peças. Princípio que orienta tudo: a IA é probabilística, o comportamento dela é consequência do roteiro e da FAQ, então a correção é sempre no roteiro/FAQ, nunca "culpando a plataforma".

## As 7 skills

- `configurar-zaia`: setup do plugin, o assinante decide tudo que é dele (paths, partes da análise, horários, limiares, branding das peças, modo/cadência) e grava num config local.
- `diagnosticar-atendimento-ia`: explica por que a IA respondeu de um jeito e ensina a ajustar o roteiro ou a FAQ para o erro não se repetir.
- `criar-roteiro-atendimento`: entrevista o assinante sobre o negócio e gera o roteiro no formato JSON da plataforma (chave `flows`), pronto para importar.
- `criar-faq-atendimento`: monta a base de conhecimento (FAQ) no formato JSON (chave `faqs`) que a IA consulta durante o atendimento.
- `recomendar-faq-da-semana`: lê os atendimentos da semana e sugere FAQs novas a partir do que mais se repetiu nas conversas.
- `rotina-analise-diaria`: varre o dia e entrega pendências, contatos em silêncio, follow-ups e lembretes, já com rascunhos de mensagem para aprovar.
- `automacao-pecas`: detecta atendimentos de coleta concluídos e gera a peça em `.docx` para o assinante revisar e assinar.

## Estrutura do repositório

- `.claude-plugin/marketplace.json`: manifesto do marketplace (`zaia-loverly`).
- `plugins/zaia/`: o plugin (manifesto em `.claude-plugin/plugin.json`, as 7 skills em `skills/`).
- `knowledge/`: base de conhecimento viva (regras, causas-raiz, roteiros e FAQs de exemplo, rotinas, metodologia de peças e `novidades.md`), indexada por `knowledge/manifest.json`. As skills leem a base ao vivo via WebFetch do RAW do GitHub, então a base atualiza sem reinstalar o plugin.

## Como o assinante instala (uma vez)

No app do Claude: Customizar > Plugins > botão "+" > Add marketplace > from a repository > cole a URL `https://github.com/zaialsystem/zaia-marketplace` > instale o plugin `zaia`. As skills aparecem na aba de habilidades.

## Como atualizar

São dois canais, conforme o que muda:

- Conteúdo e exemplos (editar arquivos em `knowledge/`): edite o arquivo, suba `contentVersion` no `knowledge/manifest.json` e dê push. O efeito é imediato, porque as skills leem o RAW via WebFetch. Não precisa reinstalar.
- Estrutura (mudança em uma skill ou skill nova): suba `pluginVersion` no `knowledge/manifest.json` e a `version` em `plugins/zaia/.claude-plugin/plugin.json` e em `.claude-plugin/marketplace.json`, e dê push. A IA do assinante compara a versão instalada com o manifest e avisa para atualizar em Customizar > Plugins.

## Observações

- Repo público, 100% estático (markdown + JSON), sem GitHub Actions.
- Documentação completa do projeto em `DOCS.md`.
- Repo: https://github.com/zaialsystem/zaia-marketplace
