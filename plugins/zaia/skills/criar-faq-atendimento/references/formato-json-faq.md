# Formato JSON da FAQ (especificação da plataforma)

Siga este formato à risca.

## Estrutura do arquivo

O JSON começa com um objeto com a chave `"faqs"`, contendo um array de entradas.

```json
{
  "faqs": [
    {
      "questionText": "Quanto custa a consulta inicial?",
      "responseText": "A consulta inicial é gratuita. Nela analisamos seu caso e explicamos as opções, sem nenhum compromisso.",
      "category": "Financeiro"
    },
    {
      "questionText": "Quais documentos preciso levar?",
      "responseText": "Depende do tipo de caso. Em geral, leve RG, CPF e comprovante de residência. Para casos trabalhistas, inclua a carteira de trabalho e os últimos contracheques. Se faltar algum documento, podemos orientar você na consulta.",
      "category": "Processo"
    }
  ]
}
```

## Campos

| Campo | Obrigatório | Descrição |
|---|---|---|
| `questionText` | Sim | A pergunta que o cliente faria, em linguagem natural. |
| `responseText` | Não (mas recomendado) | A resposta que o agente deve dar: clara, acolhedora, sem juridiquês. |
| `category` | Não (padrão "Geral") | Categoria temática. |

## Categorias sugeridas

- **Geral**: funcionamento do escritório, localização, horários.
- **Financeiro**: valores, formas de pagamento, consulta gratuita.
- **[Nome de cada serviço]**: dúvidas específicas do serviço (ex.: "Rescisão Trabalhista"). Crie uma categoria por serviço oferecido.
- **Objeções**: hesitações comuns ("é caro", "já tenho advogado", "vou pensar", "meu caso não dá em nada").
- **Exemplos de casos**: uma entrada por caso (perfil + desfecho), para os roteiros de prospecção consultarem e escolherem o mais parecido com o cliente. Não prometer o mesmo resultado.
- **Processo**: andamento, prazos, documentos necessários.
- Áreas específicas conforme o escritório (Trabalhista, Família, Previdenciário, Consumidor, etc.).

## Boas práticas

1. Pergunta específica e natural (melhor que genérica).
2. Resposta completa mas enxuta (1 a 3 parágrafos).
3. Linguagem acessível, sem termos técnicos sem explicação.
4. Tom acolhedor.
5. Inclua o próximo passo quando possível.
6. Cubra pelo menos 5 objeções.
7. Organize por categoria, criando uma categoria por serviço.

## Entrada de "Exemplo de caso" (modelo)

```json
{
  "questionText": "Exemplo de caso: trabalhador com LER/DORT readmitido",
  "responseText": "Atendemos um cliente que desenvolveu LER por esforço repetitivo após anos na mesma função. O caso foi conduzido com laudo e perícia, resultando em reconhecimento do nexo e indenização. Cada caso é único e não há promessa de resultado, mas mostra como tratamos situações parecidas.",
  "category": "Exemplos de casos"
}
```

A descrição precisa deixar claro o perfil (área, dor) para a IA casar com o relato do cliente. Não inclua valores nem promessa de resultado.

## Instruções de geração

1. Gere entre 20 e 50 FAQs cobrindo as categorias relevantes ao escritório.
2. Adapte ao contexto (áreas, localização, diferenciais).
3. Pelo menos 5 objeções.
4. Respostas entre 1 e 3 parágrafos.
5. Crie as categorias que os roteiros mandam consultar.
6. Retorne apenas o JSON, sem texto adicional, quando o assinante pedir o arquivo final.
