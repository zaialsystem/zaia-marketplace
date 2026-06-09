# Projeto Za.ia, plugin de skills para o assinante (documentação)

Documento de referência do que o projeto é, como funciona e como mantê-lo. Dono: Za.ia Legal System (contato@zaials.com.br). Repo: https://github.com/zaialsystem/zaia-marketplace

## 1. O que é o projeto

Um marketplace (`zaia-loverly`) com um plugin (`zaia`) para o app do Claude (Cowork/Chat). O plugin reúne 7 skills que ajudam o assinante da plataforma de atendimento por IA da Za.ia a refinar e operar o atendimento de ponta a ponta: configurar o plugin, diagnosticar atendimentos, criar roteiros, criar FAQs, minerar FAQs novas a partir das conversas, rodar a análise diária de pendências e oportunidades, e automatizar a geração de peças.

Princípio que orienta tudo: a IA é probabilística, não automação. O comportamento dela é consequência do que está (ou não está) escrito no roteiro e na FAQ. Por isso a correção é sempre no roteiro/FAQ, nunca "culpando a plataforma". As skills de criação e de diagnóstico existem justamente para mexer no lugar certo.

## 2. As 7 skills

Estas leem dados ao vivo via MCP da Za.ia (marcadas com "ao vivo"): `diagnosticar-atendimento-ia`, `recomendar-faq-da-semana`, `rotina-analise-diaria` e `automacao-pecas`. As de criação (`criar-roteiro-atendimento`, `criar-faq-atendimento`) e a de setup (`configurar-zaia`) funcionam sem o conector.

| Skill | O que faz | Gatilhos (exemplos) |
|---|---|---|
| `configurar-zaia` | Faz o setup inicial e os reajustes. Deixa o assinante decidir tudo que é dele (onde guardar config/estado/artefatos, quais partes da análise diária quer, horários, limiares, branding das peças, modo manual/automático e cadência) e grava num config local que as outras skills leem. | "configurar zaia", "ajustar minhas preferências", "começar", "setup", "primeira vez", "ativar as rotinas" |
| `diagnosticar-atendimento-ia` (ao vivo) | Analisa UM atendimento, acha a causa-raiz do erro e entrega a correção pronta de roteiro/FAQ, ensinando o assinante a evitar a repetição. | "a IA respondeu errado", "analisa esse atendimento", "por que ela respondeu assim", "como ajusto o roteiro", "ela inventou", "ela não transferiu" |
| `criar-roteiro-atendimento` | Entrevista o assinante sobre o negócio e o serviço e gera o roteiro no formato JSON da plataforma (chave `flows`), já com a ponte explícita para a FAQ nos temas importantes (preços, prazos, exemplos de casos). | "criar roteiro", "novo roteiro", "montar o fluxo", "quero melhorar o atendimento da IA", "melhorar a conversão" |
| `criar-faq-atendimento` | Gera a base de conhecimento (FAQ) no formato JSON (chave `faqs`), com categorias, objeções e a categoria de exemplos de casos que os roteiros consultam. | "criar FAQ", "montar a base de conhecimento", "cadastrar perguntas frequentes", "a IA precisa saber responder sobre X", "criar objeções" |
| `recomendar-faq-da-semana` (ao vivo) | Lê os atendimentos do período (a semana, por padrão) e sugere FAQs novas a partir do que se repetiu, do que a equipe teve que informar manualmente e do que a IA não soube responder. Pode rodar agendada. | "o que virou FAQ essa semana", "analisa os atendimentos da semana", "que perguntas se repetiram", "sugere FAQs novas" |
| `rotina-analise-diaria` (ao vivo) | Varre o dia e entrega um painel: pendências do escritório (o que foi prometido e não cumprido), contatos que pararam de responder, follow-up/remarketing e lembretes, cada item já com mensagem rascunhada para aprovar. Roda agendada. | "análise diária", "o que está pendente", "quem parou de responder", "follow-up dos contatos", "rodar a rotina do dia" |
| `automacao-pecas` (ao vivo) | Gera peças (documentos) a partir dos dados coletados num atendimento, no branding do próprio assinante, em `.docx`. Modo manual (aponta um atendimento) e modo automático (vigia um slug de roteiro e gera as novas peças a cada 30 a 60 min). Agnóstica ao tipo: o assinante decide quais peças faz enviando os modelos dele. Roda agendada. | "gerar procuração", "criar o contrato desse atendimento", "gerar a peça", "automatizar as peças" |

## 3. Pré-requisito (conector MCP da Za.ia)

Para as skills que leem ao vivo, o assinante precisa do conector da plataforma ligado na conta dele. Sem o conector, as skills de diagnóstico e recomendação orientam e geram conteúdo, mas não puxam conversas/roteiros reais; e as rotinas não têm o que ler.

Ferramentas usadas (entre outras, conforme a skill): `ler_conversa`, `ler_atendimento_ia`, `listar_atendimentos_ia`, `ler_roteiro`, `listar_roteiros`, `buscar_faq`, `buscar_contatos`, `listar_conversas_por_contato`. A leitura é sempre somente leitura; nenhuma skill escreve na Za.ia sem o assinante aprovar (ver seção 6).

## 4. A base de conhecimento (`knowledge/`) indexada pelo `manifest.json`

A "inteligência" do plugin (as regras de método, as causas-raiz de erro, os roteiros e FAQs de exemplo, as heurísticas das rotinas e a metodologia de peças) vive em `knowledge/`, fora das skills, e é atualizável sem reinstalar nada. O `knowledge/manifest.json` é o índice central: lista cada arquivo com seu `path`, um `resumo`, o campo `aplica` (quais skills usam) e, quando faz sentido, `segmento`/`tipo`/`categoria` para filtrar.

Como as skills consultam (3 passos, no início de cada execução):

1. Ler o manifest. WebFetch de `https://raw.githubusercontent.com/zaialsystem/zaia-marketplace/main/knowledge/manifest.json`.
2. Filtrar. Selecionar só os itens cujo `aplica` inclui a skill atual, refinando por `segmento`/`tipo`/`categoria` quando o manifest oferecer (ex.: ao criar um roteiro de recepção de advocacia, pegar o roteiro de exemplo com `segmento: advocacia` e `tipo: recepcao`).
3. WebFetch só do necessário. Montar a URL de cada arquivo escolhido com `rawBase + path` e ler apenas esses. Nada de baixar a base inteira.

Regra de conflito: em divergência entre o conteúdo embutido no plugin e o do GitHub, o GitHub prevalece. O embutido é só fallback mínimo. Se o WebFetch falhar, a skill segue com o fallback e avisa o assinante que está sem a base atualizada. As skills não inventam conteúdo de arquivo que não leram.

O que há hoje em `knowledge/` (indexado no manifest):
- `regras/`: roteiro-geral, recepcao, prospeccao, integracao-roteiro-faq, mecanica-plataforma, mineracao-faq.
- `causas-raiz/catalogo.md`: taxonomia das causas-raiz de erro, com sinais e correção (usada no diagnóstico).
- `roteiros/`: recepcao-advocacia.json e solicitacao-dados-procuracao.json (exemplos prontos).
- `faqs/`: exemplos-de-casos.json e precos-prazos.json (modelos de estilo e estrutura).
- `rotinas/`: analise-diaria.md e deteccao-pecas.md (heurísticas).
- `pecas/como-gerar-peca.md`: metodologia de geração de peça a partir do atendimento.
- `novidades.md`: o canal "o que dá para fazer agora" (ver seção 5).

## 5. Mecanismo de atualização (`contentVersion` vs `pluginVersion`)

O manifest carrega duas versões com papéis diferentes:

- `contentVersion`: muda quando só o conteúdo de `knowledge/` muda (uma regra ajustada, um exemplo novo, uma entrada em `novidades.md`). Como as skills leem o RAW via WebFetch, o efeito é imediato para todos os assinantes, sem reinstalar.
- `pluginVersion`: muda quando a estrutura muda (uma skill alterada ou uma skill nova). Aqui o assinante precisa atualizar o plugin no app.

Comparação com a versão instalada: no início, cada skill lê a versão instalada em `${CLAUDE_PLUGIN_ROOT}/.claude-plugin/plugin.json` e compara com `manifest.pluginVersion`. Se o manifest for maior, a skill prepara um aviso em linguagem simples ("Saiu uma atualização do plugin Za.ia. Veja as novidades e atualize em Customizar > Plugins."), citando o resumo de `novidades.md`. Se `manifest.skills` listar uma skill que o assinante ainda não tem, a skill avisa que há novidade a instalar pelo mesmo caminho. A `rotina-analise-diaria` inclui esse aviso no próprio painel diário.

`novidades.md`: é o canal semanal de novidades. A Za.ia Legal System edita esse arquivo a cada semana (o que mudou, o que passou a funcionar, o que vale experimentar). As rotinas leem o arquivo a cada run, então as novidades chegam ao assinante sem reinstalar nada.

## 6. As rotinas no lado do assinante (`rotina-analise-diaria` e `automacao-pecas`)

As duas rotinas rodam no Cowork do assinante (na conta dele, com o conector dele), leem os atendimentos via MCP (somente leitura) e mantêm o estado LOCAL (cursores de "até onde já processei", artefatos gerados), fora de qualquer repositório.

Regra de aprovação, inegociável: nada é criado na Za.ia sem o assinante aprovar, e nada vai ao cliente sem aprovação.
- `rotina-analise-diaria`: propõe o painel com mensagens rascunhadas e só dispara o que o assinante aprovar, item a item. Não cria tarefa, agendamento nem mensagem por conta própria.
- `automacao-pecas`: gera a peça para o assinante revisar e assinar. Nunca envia o documento ao cliente. A leitura dos atendimentos é somente leitura; nenhuma escrita na plataforma sem autorização.

Quando rodam como tarefa agendada (horários definidos no setup), o comportamento é o mesmo: produzem o resultado para o assinante, sem disparar nada sozinhas.

## 7. A skill `configurar-zaia` (setup que deixa o assinante decidir tudo)

É o "painel de controle" do plugin: o ponto de partida no primeiro uso e para onde as outras skills mandam o assinante quando não acham as preferências dele. Ela não altera nada na plataforma; só pergunta o que é do assinante e grava num config local. Conduz como conversa guiada, uma decisão de cada vez, sempre com uma sugestão pronta para quem não quer pensar muito.

O assinante decide:
- Paths: onde guardar config, estado e artefatos (peças geradas), sempre fora de Git.
- Análise diária: quais partes quer (tudo ou só algumas: pendências, contatos frios, follow-up/remarketing, lembretes), horários e limiares (ex.: quantos dias de silêncio contam como "contato frio").
- Peças: os modelos de branding do assinante, o modo (manual ou automático), e a cadência do automático.

Princípios que a skill deixa explícitos ao assinante: nada é gravado na Za.ia; o config e o estado nunca vão para a plataforma; nada de dado sensível versionado (config, estado e modelos de branding ficam numa pasta do assinante, fora de qualquer repositório).

## 8. Estrutura do repositório

```
zaia-marketplace/                      (raiz do repositório GitHub)
├── .claude-plugin/
│   └── marketplace.json               (manifesto do marketplace: zaia-loverly)
├── knowledge/                         (base de conhecimento viva, lida via WebFetch)
│   ├── manifest.json                  (índice central: paths, resumos, aplica, segmento/tipo/categoria, versões)
│   ├── novidades.md
│   ├── regras/                        (roteiro-geral, recepcao, prospeccao, integracao-roteiro-faq, mecanica-plataforma, mineracao-faq)
│   ├── causas-raiz/                   (catalogo.md)
│   ├── roteiros/                      (recepcao-advocacia.json, solicitacao-dados-procuracao.json)
│   ├── faqs/                          (exemplos-de-casos.json, precos-prazos.json)
│   ├── rotinas/                       (analise-diaria.md, deteccao-pecas.md)
│   └── pecas/                         (como-gerar-peca.md)
├── README.md
├── DOCS.md
└── plugins/
    └── zaia/                          (o plugin)
        ├── .claude-plugin/
        │   └── plugin.json            (manifesto do plugin: name, version, author)
        └── skills/
            ├── configurar-zaia/
            ├── diagnosticar-atendimento-ia/
            ├── criar-roteiro-atendimento/
            ├── criar-faq-atendimento/
            ├── recomendar-faq-da-semana/
            ├── rotina-analise-diaria/
            └── automacao-pecas/
```

Cada skill tem seu `SKILL.md` (com frontmatter `name` + `description`, onde `name` é igual ao nome da pasta). A pasta `.tools/` (validador) não vai para o GitHub: está no `.gitignore`.

## 9. Como o assinante instala (uma vez)

No app do Claude: Customizar > Plugins > botão "+" em "Personal plugins" > Add marketplace > from a repository > cole a URL `https://github.com/zaialsystem/zaia-marketplace` > instale o plugin `zaia`. As skills aparecem na aba de habilidades. Sendo o repo público, basta a URL; não é preciso dar acesso a cada assinante.

## 10. Como publicar e atualizar

Tudo é git push para `zaialsystem/zaia-marketplace`. Conforme o que muda:

- Conteúdo e exemplos (arquivos em `knowledge/`): edite o arquivo, suba `contentVersion` no `knowledge/manifest.json` e dê push. Efeito imediato via WebFetch, sem reinstalar. Inclui a edição semanal de `novidades.md`.
- Estrutura (skill alterada ou nova): edite os arquivos, suba `pluginVersion` no `knowledge/manifest.json` e a `version` em `plugins/zaia/.claude-plugin/plugin.json` e em `.claude-plugin/marketplace.json`, e dê push. O assinante recebe o aviso de atualização (seção 5).

Antes do push, vale rodar o validador local (`.tools/validate.py`), que confere o parse dos JSON, a consistência do manifest, a sincronia das versões, a ausência de placeholder de conta e a ausência de GitHub Actions.

Regra de segurança: sem GitHub Actions. O repositório não tem `.github/workflows/` e não deve ter. A conta da Za.ia Legal System fica sem Actions de propósito (evita scraping, cron de terceiros e qualquer automação que coloque a conta em risco). O projeto é 100% estático (markdown + JSON): tudo o que precisa acontecer, acontece na hora da leitura via WebFetch pelas skills.

## Observação de estilo

Padrão da Za.ia: sem travessão. Usar vírgula, dois-pontos ou parênteses no lugar.
