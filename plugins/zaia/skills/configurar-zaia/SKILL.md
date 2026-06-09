---
name: configurar-zaia
description: Configura o plugin Za.ia no primeiro uso e quando o assinante quiser reajustar. Apresenta o que cada skill faz e deixa o assinante decidir tudo que e dele: onde guardar config/estado/artefatos, quais partes da analise diaria quer (tudo ou algumas), horarios, limiares, e a parte de pecas (modelos de branding, modo manual/automatico, cadencia). Use SEMPRE que o assinante disser "configurar zaia", "ajustar minhas preferencias", "comecar", "setup", "primeira vez", "ativar as rotinas", "configurar a analise diaria", "configurar as pecas", ou quando outra skill nao encontrar o config local. Grava as escolhas num config local; as demais skills leem esse config.
---

# Configurar o plugin Za.ia (setup e ajustes do assinante)

Esta skill faz o **primeiro acerto** do plugin Za.ia e serve também para **reajustar** quando o assinante quiser. Ela não muda nada na plataforma Za.ia: só pergunta o que é do assinante (onde guardar as coisas, quais rotinas quer, horários, limiares e como tratar as peças) e grava essas escolhas num arquivo local que as outras skills leem.

O público é o **próprio assinante**, em geral um advogado que não é técnico. Fale de forma clara, didática e tranquila. Nada de jargão sem explicar. Conduza como uma conversa guiada, uma decisão de cada vez, sempre com uma sugestão pronta para quem não quiser pensar muito. No fim, o assinante deve entender o que ficou configurado e onde isso vive.

Pense nesta skill como o "painel de controle" do plugin: ela é o ponto de partida no primeiro uso, e é para onde as outras skills mandam o assinante quando não acham as preferências dele.

## Base de conhecimento e atualizacao (consultar SEMPRE no inicio)

Antes de agir, leia o indice central da Za.ia Legal System. Use WebFetch para ler:

`https://raw.githubusercontent.com/zaialsystem/zaia-marketplace/main/knowledge/manifest.json`

Do manifest: leia `novidades.md` (via `rawBase` + path) para mostrar ao assinante o que ha de novo. Em conflito, o GitHub prevalece sobre o embutido; se o fetch falhar, siga com o fallback e avise. Aviso de atualizacao: compare `manifest.pluginVersion` com a versao instalada em `${CLAUDE_PLUGIN_ROOT}/.claude-plugin/plugin.json`; se o manifest for maior, avise o assinante para atualizar em Customizar > Plugins.

## Princípios (deixe explícito para o assinante)

Diga isto com suas palavras no começo, para o assinante ficar tranquilo:

- **Nada é gravado na Za.ia.** Toda escolha de configuração fica no computador do assinante (arquivos locais). A skill não cria nem altera nada na plataforma sem pedir.
- **O config e o estado nunca vão para a Za.ia.** São preferências e cursores de uso interno do plugin, não dados da plataforma.
- **Nada de dado sensível versionado.** Os arquivos de config, de estado e os modelos de branding ficam numa pasta do assinante, fora de qualquer repositório ou controle de versão. Nunca suba isso para Git.

## Workflow do setup

### Passo 1: Apresentar as 7 skills (uma linha cada)

Mostre ao assinante o que ele tem em mãos, em frases curtas, para ele saber o que pode pedir depois:

1. **configurar-zaia** (esta): faz o setup e reajusta as preferências do plugin; grava tudo num config local que as outras skills leem.
2. **diagnosticar-atendimento-ia**: explica por que a IA respondeu de um jeito e entrega o ajuste pronto no roteiro ou na FAQ para o erro não se repetir.
3. **criar-roteiro-atendimento**: entrevista o assinante sobre o negócio e gera um roteiro de atendimento no formato da plataforma, pronto para importar.
4. **criar-faq-atendimento**: monta entradas de FAQ (preços, prazos, exemplos de casos) no formato da plataforma, para a IA responder com informação curada.
5. **recomendar-faq-da-semana**: garimpa as conversas da semana e sugere FAQs novas a partir do que os clientes mais perguntaram.
6. **rotina-analise-diaria**: todo dia no horário escolhido, varre os atendimentos e entrega um painel com pendências, contatos frios, oportunidades de follow-up e lembretes.
7. **automacao-pecas**: a partir dos atendimentos que coletaram os dados certos, gera peças (ex.: procuração, contrato) já no padrão visual da banca.

Pergunte se ele quer configurar tudo agora ou só uma parte (ex.: só a análise diária). Siga o ritmo dele.

### Passo 2: Escolher onde guardar config, estado, artefatos e modelos

Explique, simples: o plugin precisa de **uma pasta no computador** do assinante para guardar as preferências dele, o controle do que as rotinas já trataram, os relatórios e peças geradas, e os modelos de documento da banca.

Sugira um padrão e deixe ele aceitar ou trocar: uma pasta **"Za.ia" dentro dos Documentos** dele. A partir dessa pasta-base, organize assim:

- **config**: o arquivo `zaia-config.json` (as preferências).
- **estado**: o arquivo `zaia-estado.json` (o que já foi tratado, para a rotina não repetir).
- **artefatos**: uma subpasta para os relatórios da análise diária e as peças geradas.
- **modelos**: uma subpasta para os modelos de branding (procuração, contrato, etc.).

Crie a pasta-base e as subpastas escolhidas. Em seguida grave um **ponteiro mínimo** num caminho fixo e convencional, para as outras skills **sempre acharem o config** mesmo que o assinante tenha escolhido uma pasta diferente:

- Caminho fixo do ponteiro: `${HOME}/.zaia-skills/config-pointer.json`
- Conteúdo do ponteiro:

```json
{ "configPath": "<caminho absoluto do zaia-config.json>" }
```

Explique o porquê em uma frase: "esse pequeno atalho faz com que qualquer skill encontre suas preferências automaticamente, sem você ter que dizer o caminho toda vez". O `zaia-config.json` de verdade fica na pasta escolhida; o ponteiro só aponta para ele.

### Passo 3: Conferir o conector MCP da Za.ia ligado

O plugin lê os atendimentos do assinante pela conexão (MCP) da Za.ia. Confira se está ligada fazendo **uma leitura simples**, por exemplo chamar `listar_roteiros` (ou outra leitura leve que retorne sem erro).

- Se responder, ótimo: avise que a conexão está ativa e siga.
- Se não responder, oriente em linguagem simples a ligar o conector da Za.ia nas conexões/MCP do Claude e tente de novo. Não prossiga para as rotinas que dependem de leitura enquanto a conexão não estiver de pé.

### Passo 4: Configurar a rotina de análise diária

Explique o que ela entrega: um resumo diário do que precisa de atenção, montado a partir dos atendimentos. Deixe o assinante **ligar ou desligar cada parte** (ele pode querer tudo ou só algumas):

- **pendencias**: atendimentos/tarefas em aberto que precisam de uma ação.
- **contatos_frios**: contatos que ficaram sem resposta há muitos dias.
- **follow_up_remarketing**: oportunidades de retomar contato (quem demonstrou interesse e esfriou).
- **lembretes**: compromissos e prazos a lembrar.

Depois pergunte:

- **Horário** em que ele quer receber o painel (ex.: 08:00). Use o fuso do assinante.
- **Limiar de dias para "contato frio"**: a partir de quantos dias sem resposta um contato entra na lista (ex.: 3 dias).

Crie a **tarefa agendada diária** no app do assinante usando o recurso de agendamento/rotina disponível, para a skill `rotina-analise-diaria` rodar sozinha no horário escolhido. Se não houver esse recurso disponível, **imprima instruções claras** para o assinante agendar manualmente (o que rodar, em que horário) e avise que, sem o agendamento, a análise só roda quando ele pedir.

### Passo 5: Configurar a automação de peças

Explique o que ela faz: quando um atendimento já coletou os dados necessários, ela **gera a peça** (ex.: procuração, contrato) no padrão visual da banca. Então configure:

1. **Coletar os modelos de branding.** Peça ao assinante **cópias dos documentos dele**, no padrão da banca, por exemplo uma **procuração** e um **contrato** que ele já usa. Salve essas cópias na **pasta de modelos** escolhida no Passo 2. Explique que esses modelos são a referência visual/textual que a peça gerada vai seguir.
2. **Escolher o modo** de geração:
   - **manual**: a peça só é gerada quando o assinante pedir.
   - **automatico**: o plugin vigia os atendimentos e gera a peça assim que os dados estiverem prontos.
   - **ambos**: aceita os dois (pode pedir na hora e também deixar rodar sozinho).
3. **Se for automático (ou ambos):**
   - Pergunte **qual slug de roteiro vigiar** (qual fluxo, quando concluído, dispara a peça). Sugira `solicitacao-dados-procuracao` como padrão.
   - Pergunte a **cadência**: de quanto em quanto tempo o plugin verifica atendimentos prontos, **30 ou 60 minutos**. Crie a **tarefa agendada** para a skill `automacao-pecas` rodar nessa cadência (se não houver recurso de agendamento, imprima instruções para agendar manualmente).
4. **Escolher o formato de saída**: **docx** (só o Word) ou **docx+pdf** (Word e PDF).

### Passo 6: Gravar tudo no config

Grave as escolhas no `zaia-config.json`, na pasta escolhida, seguindo o schema da próxima seção. Confirme com o assinante, em linguagem simples, um resumo do que ficou (quais rotinas ligadas, horário, limiar, modo das peças, formato).

**Reexecutar esta skill ajusta o config:** ao rodar de novo, **leia o config existente** primeiro e **mude só o que o assinante pedir**, preservando o resto. Reconfigurar não recomeça do zero, só atualiza o que ele quiser trocar.

## Schema do config (as outras skills leem isto)

O arquivo `zaia-config.json` tem exatamente esta forma. Os caminhos são absolutos, na pasta escolhida pelo assinante:

```json
{
  "versao": "1",
  "paths": { "config": "<abs>", "estado": "<pasta/arquivo de estado e cursores>", "artefatos": "<pasta de relatorios e pecas>", "modelos": "<pasta dos modelos de branding>" },
  "analiseDiaria": { "ativa": true, "partes": ["pendencias","contatos_frios","follow_up_remarketing","lembretes"], "horario": "08:00", "limiarContatoFrioDias": 3 },
  "pecas": { "ativa": true, "modo": "manual", "slugVigiado": "solicitacao-dados-procuracao", "cadenciaMin": 60, "formato": "docx" }
}
```

Notas de cada campo:

- **versao**: versão do schema do config (mantenha `"1"`).
- **paths.config**: caminho absoluto do próprio `zaia-config.json`.
- **paths.estado**: pasta (ou arquivo) onde vive o estado/cursor das rotinas (ver abaixo).
- **paths.artefatos**: pasta onde a análise diária e as peças salvam saídas.
- **paths.modelos**: pasta dos modelos de branding coletados no Passo 5.
- **analiseDiaria.ativa**: liga/desliga a análise diária inteira.
- **analiseDiaria.partes**: lista só com as partes que o assinante quis (qualquer subconjunto de `pendencias`, `contatos_frios`, `follow_up_remarketing`, `lembretes`).
- **analiseDiaria.horario**: hora do dia (formato `HH:MM`) no fuso do assinante.
- **analiseDiaria.limiarContatoFrioDias**: número de dias sem resposta para um contato virar "frio".
- **pecas.ativa**: liga/desliga a automação de peças.
- **pecas.modo**: `manual`, `automatico` ou `ambos`.
- **pecas.slugVigiado**: slug do roteiro que, ao concluir, dispara a peça (usado no modo automático).
- **pecas.cadenciaMin**: `30` ou `60` (minutos entre verificações no modo automático).
- **pecas.formato**: `docx` ou `docx+pdf`.

## Estado e cursores (arquivo local separado)

O estado/cursor das rotinas **não vive no config**. Ele fica num arquivo **local separado**, `zaia-estado.json`, na **pasta de estado** (`paths.estado`). Serve para as rotinas saberem o que já trataram e não repetir. A forma é:

```json
{ "analiseDiaria": { "ultimoRun": "ISO", "itensTratados": [] }, "pecas": { "conversasGeradas": [] } }
```

- **analiseDiaria.ultimoRun**: data/hora ISO da última execução da análise.
- **analiseDiaria.itensTratados**: ids do que já entrou em painel, para não repetir.
- **pecas.conversasGeradas**: ids das conversas/atendimentos para os quais a peça já foi gerada.

Este arquivo é manejado pelas rotinas no dia a dia; aqui no setup, garanta apenas que a pasta de estado existe e que `paths.estado` aponta para ela. Como o config, o estado nunca vai para a Za.ia nem para Git.

## Observação sobre estilo de escrita

A Za.ia não usa travessão na escrita. Ao conversar com o assinante e ao escrever qualquer texto, use vírgula, dois-pontos ou parênteses no lugar.
