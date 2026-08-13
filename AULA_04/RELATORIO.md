# Relatório — Avaliação de Estratégias de Chunking com LangChain

## 1. Objetivo

Implementar e comparar 10 estratégias diferentes de divisão de documentos (chunking) usando os splitters do LangChain, gerar embeddings para cada chunk e avaliar como cada estratégia influencia a quantidade de chunks, o tamanho, a preservação de contexto e a qualidade da estrutura resultante para uso em sistemas de RAG.

## 2. Base de documentos

Foram utilizados 12 documentos PDF: 3 documentos das aulas anteriores (`bioetica_e_ia`, `escrita_academica_ia`, `twitter_algoritmo`) e 9 papers técnicos de referência em IA (Attention Is All You Need, BERT, GPT-3, GPT-4, InstructGPT, LLaMA, LoRA, Retrieval-Augmented Generation, Scaling Laws).

Seguindo a orientação do exercício, as 10 estratégias de chunking foram testadas primeiro apenas nos 3 documentos-base, para viabilizar a comparação. Depois de selecionadas as estratégias mais adequadas, elas foram aplicadas ao conjunto completo dos 12 documentos.

## 3. Extração PDF → Markdown

A conversão de PDF para Markdown foi feita com a biblioteca `pymupdf4llm`. A extração preservou bem títulos e a hierarquia de seções (`#`, `##`, `###`), além de formatação de superscript (útil para notas de rodapé e afiliações de autores).

**Observações relevantes sobre a extração:**

- Vários documentos (incluindo `retrieval_augmented_generation.pdf` e `gpt3_language_models.pdf`) precisaram de OCR em boa parte das páginas, indicando que os PDFs de origem não continham texto nativo selecionável em todas as páginas. OCR introduz risco de erros de reconhecimento, especialmente em fórmulas matemáticas e símbolos.
- **Imagens** não ficaram representadas de nenhuma forma no Markdown resultante — foram efetivamente descartadas na conversão.
- **Tabelas** tiveram o conteúdo preservado, mas não a formatação Markdown padrão de tabela (`| Coluna A | Coluna B |` com linha separadora `|---|---|`). Os valores aparecem como uma sequência de campos separados por `|`, sem uma linha de cabeçalho claramente distinta.

## 4. Estratégias testadas nos 3 documentos-base

| Teste | Estratégia | Configuração |
|---|---|---|
| 1 | Fixo | 200 caracteres, sem overlap |
| 2 | Fixo | 500 caracteres, sem overlap |
| 3 | Fixo | 1000 caracteres, sem overlap |
| 4 | Fixo | 2000 caracteres, sem overlap |
| 5 | Fixo + overlap | 500 caracteres, overlap 50 |
| 6 | Fixo + overlap | 500 caracteres, overlap 200 |
| 7 | Por parágrafo | Separação por `\n\n` |
| 8 | Por sentença | Sentenças agrupadas em 3 |
| 9 | Recursivo | Separadores hierárquicos (parágrafo → linha → espaço → caractere) |
| 10 | Markdown | Separação por headings (`#`, `##`, `###`) |

### 4.1 Resultados consolidados (média dos 3 documentos-base)

| Teste | Estratégia | Chunks (média) | Tam. médio | Tam. mín | Tam. máx | Tokens (média) |
|---|---|---|---|---|---|---|
| 1 | Fixo 200 | 266,3 | 199,2 | 92,3 | 200,0 | 15426,7 |
| 2 | Fixo 500 | 106,7 | 498,0 | 325,7 | 500,0 | 14947,7 |
| 3 | Fixo 1000 | 53,3 | 996,4 | 825,3 | 1000,0 | 14796,7 |
| 4 | Fixo 2000 | 27,0 | 1968,1 | 1158,7 | 2000,0 | 14722,0 |
| 5 | Fixo+overlap leve (50) | 118,7 | 497,0 | 209,0 | 500,0 | 16595,7 |
| 6 | Fixo+overlap pesado (200) | 177,0 | 498,8 | 358,7 | 500,0 | 24897,3 |
| 7 | Parágrafo | 190,0 | 281,5 | 2,7 | 1587,7 | 14655,3 |
| 8 | Sentença agrupada (3) | 145,0 | 384,6 | 29,3 | 1661,0 | 14753,3 |
| 9 | Recursivo | 154,7 | 355,8 | 25,3 | 499,0 | 15329,3 |
| 10 | Markdown (headings) | 19,3 | 2706,0 | 95,7 | 8530,7 | 14374,3 |

## 5. Análise comparativa das estratégias

**Qual gerou mais chunks?** O Teste 1 (fixo, 200 caracteres), com 266,3 chunks em média — esperado, já que quanto menor o chunk, mais fragmentos são necessários.

**Qual gerou menos chunks?** O Teste 10 (Markdown, por headings), com apenas 19,3 chunks em média, por cortar somente nas grandes seções do documento.

**Como o tamanho variou?** Nos testes de tamanho fixo, a relação é praticamente inversa: dobrar o `chunk_size` reduz o número de chunks quase pela metade. Nas estratégias estruturais (parágrafo, sentença, Markdown), a variação é irregular, refletindo o tamanho real dos elementos do documento.

**Qual preservou melhor a estrutura?** O Markdown Splitter (Teste 10), por respeitar explicitamente a hierarquia de títulos e seções, incluindo essa hierarquia nos metadados de cada chunk. O Recursive (Teste 9) também demonstrou boa preservação indireta de estrutura: em um dos testes, parou um chunk exatamente antes do início de uma nova seção, mesmo sem ter atingido o limite de tamanho configurado.

**Como tabelas e imagens foram tratadas?** Ver seção 3.

**O chunking por caracteres fragmentou conceitos importantes?** Sim, principalmente no Teste 1 (200 caracteres) — frases e ideias podem ser cortadas no meio, já que o corte não tem nenhum conhecimento da estrutura do texto.

**O chunking por parágrafo produziu chunks muito grandes?** Não — produziu chunks pequenos em média (281,5 caracteres), mas com grande variação (de 2,7 a 1587,7 caracteres). O menor chunk observado tinha apenas 3 caracteres, revelando que muitos "parágrafos" identificados pela extração são, na prática, fragmentos curtos de metadado (título, ISSN, afiliação), não blocos de texto substancial.

**O chunking por sentença preservou melhor o contexto?** Parcialmente. Em texto corrido funcionou bem, mas a lógica de pontuação simples (`.!?`) não distingue fim de frase real de outros usos do ponto — elementos como numeração de afiliação ("1.") foram tratados como sentenças completas, misturando metadado estrutural com conteúdo.

**O Recursive Splitter apresentou vantagens?** Sim, claramente, na comparação direta com o Teste 5 (mesmo `chunk_size` e `overlap`, splitter fixo): o Recursive gerou mais chunks e menores em média (355,8 vs. 497,0 caracteres), por priorizar cortes em pontos estruturais naturais antes de recorrer ao corte bruto por caractere.

**O Markdown Splitter preservou a estrutura semântica?** Sim, foi a estratégia com melhor preservação entre as 10 testadas. A limitação observada é que conteúdo situado antes do primeiro heading (cabeçalho de revista, ISSN, etc.) fica sem nenhum metadado de seção associado.

**Qual estratégia parece mais adequada para RAG?** Não há uma resposta única — depende do tipo de consulta. Para perguntas pontuais, um chunk pequeno e preciso favorece a recuperação. Para perguntas que exigem contexto amplo (ex: resumir uma metodologia), uma estratégia estrutural entrega um bloco mais coerente. Em termos de equilíbrio, o **Recursive Splitter** parece a escolha mais robusta como padrão, por combinar tamanho controlado com respeito à estrutura natural do texto, sem depender de o documento ter formatação Markdown rica em headings.

**Quais estratégias devem ser descartadas?** O Teste 1 (200 caracteres, tamanho pequeno demais e sem ganho de precisão sobre o Teste 2), o chunking por sentença com regex simples (frágil em documentos com muita estrutura/metadado) e o chunking por parágrafo sem filtro de tamanho mínimo (gera muitos chunks "lixo").

**Quais estratégias usar em próximos experimentos?** Recursive Splitter como padrão; Markdown Splitter como alternativa quando o documento tiver headings bem definidos; e uma variação do chunking por parágrafo com filtro de tamanho mínimo, para eliminar o ruído observado.

## 6. Estratégias selecionadas e aplicação nos 12 documentos completos

Com base na análise acima, foram selecionadas duas estratégias complementares para aplicação no conjunto completo de 12 documentos:

- **Recursive Character Text Splitter** (Teste 9) — estratégia padrão, pelo equilíbrio entre tamanho controlado e respeito à estrutura.
- **Markdown Header Text Splitter** (Teste 10) — estratégia complementar, para casos que exigem mais contexto por chunk.

### 6.1 Resultados nos 12 documentos completos

| Documento | Recursive (chunks) | Markdown (chunks) | Tam. médio Recursive | Tam. médio Markdown |
|---|---|---|---|---|
| attention_is_all_you_need | 120 | 26 | 360,4 | 1590,8 |
| bert_pretraining | 193 | 31 | 351,4 | 2090,3 |
| bioetica_e_ia | 152 | 19 | 361,8 | 2770,7 |
| escrita_academica_ia | 140 | 20 | 349,5 | 2294,8 |
| gpt3_language_models | 750 | 87 | 362,7 | 2998,9 |
| gpt4_technical_report | 847 | 136 | 359,0 | 2142,8 |
| instruct_gpt | 557 | 67 | 354,5 | 2836,3 |
| llama_foundation_models | 264 | 58 | 356,1 | 1547,5 |
| lora_low_rank_adaptation | 269 | 42 | 344,4 | 2111,1 |
| retrieval_augmented_generation | 221 | 33 | 339,0 | 2185,5 |
| scaling_laws_llm | 284 | 22 | 343,9 | 3820,7 |
| twitter_algoritmo | 172 | 19 | 356,1 | 3052,5 |

O padrão observado nos 3 documentos-base se confirma em escala real: o Markdown Splitter gera consistentemente muito menos chunks, com tamanho médio várias vezes maior. O caso mais extremo é o GPT-4 Technical Report (documento mais longo da base): 847 chunks pelo Recursive contra apenas 136 pelo Markdown.

## 7. Desafios encontrados e soluções aplicadas

Durante a implementação, três problemas reais surgiram e foram resolvidos:

**a) Chunking por parágrafo retornando apenas 1 chunk por documento.** Ao configurar o `CharacterTextSplitter` com um `chunk_size` intencionalmente alto (para isolar o efeito do separador `\n\n`), o splitter identificou corretamente os pontos de quebra, mas em seguida remontou os fragmentos automaticamente até caberem no limite configurado — um comportamento padrão da biblioteca que não era o esperado. A correção foi implementar a divisão diretamente em Python, sem o passo de remontagem.

**b) Chunks excessivamente grandes gerados pelo Markdown Splitter em documentos técnicos longos.** Em papers como GPT-3, GPT-4 e LLaMA, algumas seções (ex: "Related Work", "Experiments") não têm sub-headings suficientes para uma quebra natural, gerando chunks de até 28 mil caracteres — acima do limite de entrada aceito pelo modelo de embedding utilizado. A solução foi aplicar um "splitting em cascata": chunks do Markdown Splitter que excedessem um limite de segurança foram reprocessados com o Recursive Splitter como segundo passo, preservando a lógica estrutural para a maioria dos chunks e garantindo compatibilidade para os casos extremos.

**c) Erro pontual de limite de tokens em um chunk aparentemente normal.** Um chunk de aproximadamente 10.600 caracteres, referente à seção de Referências Bibliográficas de um dos papers, foi tokenizado de forma anormalmente densa, excedendo o limite de tokens da API mesmo sendo processado individualmente. A causa provável é a alta concentração de caracteres especiais e notação de citação típica de listas de referências. A solução aplicada foi um tratamento de erro robusto: qualquer chunk que falhasse na geração de embedding era automaticamente ignorado e registrado em um log, sem interromper o processamento dos demais — de um total de mais de 4.400 chunks processados no conjunto completo, apenas 1 precisou ser descartado dessa forma.

## 8. Conclusão

Não existe uma estratégia de chunking universalmente superior — a escolha depende do tipo de aplicação e do formato do documento. Entre as 10 estratégias avaliadas, o **Recursive Character Text Splitter** se destacou como a opção mais equilibrada para uso geral em sistemas de RAG, por respeitar a estrutura natural do texto sem depender de formatação específica. O **Markdown Header Text Splitter** se mostrou mais adequado quando o objetivo é preservar contexto semântico amplo (por exemplo, para tarefas de sumarização), desde que combinado com um mecanismo de segurança para limitar o tamanho máximo de chunk.

A etapa de extração de PDF para Markdown mostrou-se uma fonte relevante de limitações — em especial a perda de imagens e a formatação incompleta de tabelas — que merecem atenção em qualquer pipeline de RAG que dependa de documentos técnicos ricos em elementos visuais e tabulares.
