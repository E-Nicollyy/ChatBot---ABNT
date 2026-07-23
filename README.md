# Assistente Acadêmico (Node.js + Azure AI Foundry)

Migração do projeto original (Python + Colab + Gradio + Gemini) para Node.js/Express,
com frontend em `index.html` puro e chamadas à IA via Azure AI Foundry.

## Estrutura

```
assistente-academico-js/
├── index.html       # Frontend (abas: TCC, Artigo, Relatório, Trabalho, Correção)
├── server.js        # Backend Express: chamada à IA + geração de .docx com formatação ABNT
├── package.json
├── .env.example      # Modelo das variáveis de ambiente
└── README.md
```

## Configuração no VSCode

1. Abra a pasta no VSCode.
2. No terminal integrado, instale as dependências:
   ```bash
   npm install
   ```
3. Copie `.env.example` para `.env`:
   ```bash
   cp .env.example .env
   ```
4. Edite o `.env` e preencha:
   - `AZURE_AI_ENDPOINT` — já vem preenchido com `https://foundry0807.services.ai.azure.com/openai/v1`
   - `AZURE_AI_KEY` — a chave do recurso no Azure AI Foundry
   - `AZURE_AI_MODEL` — o **nome do deployment** publicado no Foundry (não é o nome genérico do modelo, é o nome que você deu ao deployment)
5. Rode o servidor:
   ```bash
   npm start
   ```
6. Acesse `http://localhost:3000` no navegador.

## Sobre o endpoint da Azure AI Foundry

O caminho `/openai/v1` é o endpoint unificado (compatível com a API da OpenAI) da Azure AI Foundry.
O `server.js` chama `POST {AZURE_AI_ENDPOINT}/chat/completions` enviando `model` (o deployment) e
`messages` no formato padrão OpenAI, autenticando com o header `api-key`.

Se o seu recurso estiver configurado para autenticação via Microsoft Entra ID (token OAuth) em vez
de chave de API, troque no `server.js`, dentro da função `chamarIA`:
```js
"api-key": AZURE_AI_KEY,
```
por:
```js
"Authorization": `Bearer ${AZURE_AI_KEY}`,
```

## O que foi portado do script Python

| Python (Colab)                          | Node.js (este projeto)                          |
|------------------------------------------|--------------------------------------------------|
| `google.generativeai` / REST Gemini      | `fetch` para `chat/completions` da Azure Foundry |
| `python-docx`                            | pacote `docx` (npm)                              |
| `pypdf`, leitura de `.docx`/`.txt`        | `pdf-parse`, `mammoth`, `fs`                      |
| Interface Gradio (`gr.Blocks`, abas)     | `index.html` com abas em HTML/CSS/JS puro         |
| `userdata.get('APYKEY')` (Colab secrets) | variáveis de ambiente via `.env` (`dotenv`)        |

A lógica de formatação ABNT (capa, folha de rosto, cabeçalho de artigo, numeração de página,
margens 3-3-2-2 cm, fonte Arial 12, espaçamento 1,5, recuo de 1,25 cm, parágrafo de resumo e
referências com estilo próprio) foi replicada função a função no `server.js`.

## Observações e pontos para revisar

- **Nomes de campo do formulário** (`tipoDoc`, `prompt`, `nomeAluno`, etc.) já batem entre
  `index.html` e `server.js` — se você alterar um lado, alinhe o outro.
- O download do `.docx` gerado é servido por uma rota temporária (`/download/:id`) que mantém o
  arquivo em memória por 30 minutos. Para uso local isso é suficiente; para produção, considere
  salvar em disco ou storage externo.
- Não há framework de build (sem React/Vite) — é HTML/JS puro carregado direto pelo Express
  (`express.static`), como pedido na estrutura do projeto.
- O limite de upload está em 15 MB (`multer`); ajuste em `server.js` se precisar de mais.
