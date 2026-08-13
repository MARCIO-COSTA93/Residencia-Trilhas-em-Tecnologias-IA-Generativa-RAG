# Introducao a IA - Aula 04

Este projeto avalia e compara 10 estrategias diferentes de divisao de documentos (chunking) para uso em sistemas de RAG, utilizando os splitters do LangChain sobre um conjunto de 12 documentos PDF (papers tecnicos de IA e os documentos das aulas anteriores).

## Estrutura do Projeto

AULA_04/

├── README.md # Este arquivo

├── RELATORIO.md # Relatorio completo de analise (configuracoes, estatisticas, desafios e conclusoes)

├── Aula_04_a.ipynb # Notebook com o pipeline completo

└── results/ # Chunks, embeddings e summaries gerados (adicionado em etapa posterior)

## Objetivo

Implementar e comparar 10 estrategias de chunking (LangChain), gerar embeddings para cada chunk e avaliar como cada estrategia influencia: quantidade de chunks, tamanho, preservacao de contexto, sobreposicao entre chunks e qualidade da estrutura resultante para uso em RAG.

## Pipeline

```
PDF
 |
 v
Extracao para Markdown (pymupdf4llm)
 |
 v
10 estrategias de chunking (LangChain splitters)
 |
 v
Embeddings (OpenRouter / nvidia-nemotron-3-embed-1b)
 |
 v
JSON estruturado (chunks + embeddings + metadados)
```

## O que foi feito

1. **Extracao de 12 PDFs para Markdown**, incluindo papers tecnicos de referencia (Attention Is All You Need, BERT, GPT-3, GPT-4, InstructGPT, LLaMA, LoRA, Retrieval-Augmented Generation, Scaling Laws) e os 3 documentos das aulas anteriores.

2. **10 estrategias de chunking testadas** nos 3 documentos-base: fixo (200/500/1000/2000 caracteres), fixo com overlap (leve e pesado), por paragrafo, por sentenca agrupada, recursivo e por estrutura Markdown (headings).

3. **Geracao de embeddings** para todos os chunks gerados, mantendo o mesmo modelo em todos os testes para permitir comparacao justa.

4. **Analise comparativa completa**, respondendo as 15 perguntas obrigatorias sobre as estrategias testadas (ver `RELATORIO.md`).

5. **Selecao das duas melhores estrategias** (Recursive Character Text Splitter e Markdown Header Text Splitter) e aplicacao nas mesmas nos 12 documentos completos.

6. **Estrutura de dados organizada em JSON**, com chunk_id, estrategia, tamanho, embedding e metadados, salva em pastas por documento e por teste.

## Principais descobertas

- O **Markdown Splitter** preserva melhor a estrutura semantica do documento, mas gera chunks muito maiores e em menor quantidade.
- O **Recursive Splitter** oferece o melhor equilibrio entre tamanho controlado e respeito a estrutura natural do texto, sendo recomendado como estrategia padrao.
- A extracao de PDF para Markdown tem limitacoes relevantes: imagens sao descartadas e tabelas perdem parte da formatacao padrao Markdown.
- Tres desafios tecnicos reais foram encontrados e resolvidos durante o processamento: comportamento de remontagem automatica do splitter de paragrafo, chunks excessivamente grandes gerados pelo Markdown Splitter em documentos tecnicos longos, e um chunk pontual com tokenizacao anomala (secao de referencias bibliograficas). Detalhes completos na secao "Desafios encontrados e solucoes aplicadas" do `RELATORIO.md`.

## Relatorio completo

A analise detalhada, com todas as tabelas de resultados, as 15 respostas obrigatorias e a secao de desafios/solucoes, esta disponivel em [`RELATORIO.md`](RELATORIO.md).

## Conceitos aprendidos

- Estrategias de chunking (fixo, com overlap, por paragrafo, por sentenca, recursivo, por estrutura semantica)
- Uso dos splitters do LangChain (`CharacterTextSplitter`, `RecursiveCharacterTextSplitter`, `MarkdownHeaderTextSplitter`)
- Extracao de PDF para Markdown e suas limitacoes (imagens, tabelas, necessidade de OCR)
- Tratamento de erros de API em pipelines de embeddings em lote (limites de tokens, processamento robusto com log de falhas)
- Trade-offs entre granularidade de chunk e qualidade de recuperacao em sistemas de RAG
