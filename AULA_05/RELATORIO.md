# Aula 05 — Documents, Metadados e Busca Vetorial com LangChain

## Exercício 1 — Criando Documents na mão

### Lista de Documents

```python
from langchain_core.documents import Document

documentos = [
    Document(
        page_content="Embeddings são representações vetoriais densas de texto.",
        metadata={"fonte": "arquivo_01.md", "pagina": 1, "tipo": "teoria", "tema": "embeddings", "autor": "Marcio"}
    ),
    Document(
        page_content="Chunking é o processo de dividir documentos longos em pedaços menores.",
        metadata={"fonte": "arquivo_01.md", "pagina": 2, "tipo": "teoria", "tema": "chunking", "autor": "Marcio"}
    ),
    Document(
        page_content="RAG combina busca de informação com geração de texto por LLMs.",
        metadata={"fonte": "arquivo_02.md", "pagina": 1, "tipo": "teoria", "tema": "rag", "autor": "Marcio"}
    ),
    Document(
        page_content="Tokenização é o processo de dividir texto em unidades menores chamadas tokens.",
        metadata={"fonte": "arquivo_02.md", "pagina": 3, "tipo": "teoria", "tema": "tokenizacao", "autor": "Marcio"}
    ),
    Document(
        page_content="O Recursive Character Text Splitter tenta preservar a estrutura natural do texto.",
        metadata={"fonte": "arquivo_01.md", "pagina": 4, "tipo": "pratica", "tema": "chunking", "autor": "Marcio"}
    ),
]
```

### Print de cada documento (page_content + metadata)

```
Documento 1:
  page_content: Embeddings são representações vetoriais densas de texto.
  metadata: {'fonte': 'arquivo_01.md', 'pagina': 1, 'tipo': 'teoria', 'tema': 'embeddings', 'autor': 'Marcio'}

Documento 2:
  page_content: Chunking é o processo de dividir documentos longos em pedaços menores.
  metadata: {'fonte': 'arquivo_01.md', 'pagina': 2, 'tipo': 'teoria', 'tema': 'chunking', 'autor': 'Marcio'}

Documento 3:
  page_content: RAG combina busca de informação com geração de texto por LLMs.
  metadata: {'fonte': 'arquivo_02.md', 'pagina': 1, 'tipo': 'teoria', 'tema': 'rag', 'autor': 'Marcio'}

Documento 4:
  page_content: Tokenização é o processo de dividir texto em unidades menores chamadas tokens.
  metadata: {'fonte': 'arquivo_02.md', 'pagina': 3, 'tipo': 'teoria', 'tema': 'tokenizacao', 'autor': 'Marcio'}

Documento 5:
  page_content: O Recursive Character Text Splitter tenta preservar a estrutura natural do texto.
  metadata: {'fonte': 'arquivo_01.md', 'pagina': 4, 'tipo': 'pratica', 'tema': 'chunking', 'autor': 'Marcio'}
```

### len(documentos)

```
5
```

### Respostas

**Que tipos de dado são aceitos dentro de `metadata`? Teste colocar uma lista ou um dicionário aninhado e relate o que acontece.**

Testamos os dois casos e ambos foram aceitos sem nenhum erro:

```python
doc_lista = Document(
    page_content="Teste com lista nos metadados.",
    metadata={"tags": ["embeddings", "rag", "chunking"]}
)
# Resultado: {'tags': ['embeddings', 'rag', 'chunking']}

doc_dict = Document(
    page_content="Teste com dicionário aninhado nos metadados.",
    metadata={"info": {"autor": "Marcio", "ano": 2026}}
)
# Resultado: {'info': {'autor': 'Marcio', 'ano': 2026}}
```

O campo `metadata` é tipado simplesmente como `dict` no LangChain, sem validação de schema — aceita listas, dicionários aninhados ou qualquer outra estrutura Python válida dentro dele. A responsabilidade de manter uma estrutura consistente é do desenvolvedor, não da biblioteca.

**Observação importante**: embora o `Document` em si aceite qualquer estrutura, isso não significa que toda vector store aceite. Muitas vector stores (como o Chroma, por exemplo) exigem que os valores de metadata sejam tipos primitivos (string, int, float, bool), não listas nem dicionários aninhados. Esse é um ponto de atenção para quando o schema for de fato indexado em uma vector store.

**O que acontece se você criar um `Document` sem passar `metadata`?**

Não ocorre nenhum erro. O parâmetro `metadata` é opcional, e quando não informado assume o valor padrão de um dicionário vazio:

```python
doc_sem_metadata = Document(page_content="Documento sem metadados definidos.")
print(doc_sem_metadata.metadata)  # {}
print(type(doc_sem_metadata.metadata))  # <class 'dict'>
```

---

## Exercício 2 — Projetando o schema de metadados

### Schema final

| Campo | Descrição | Origem |
|---|---|---|
| `fonte` | Nome do arquivo `.md` de origem | Obrigatório |
| `documento_id` | Identificador do documento | Obrigatório |
| `chunk_index` | Posição do chunk dentro do documento | Obrigatório |
| `estrategia` | Qual das 10 estratégias da Aula 04 gerou este chunk | Obrigatório |
| `chunk_size` | Configuração usada | Obrigatório |
| `chunk_overlap` | Configuração usada | Obrigatório |
| `n_caracteres` | Tamanho real do chunk | Obrigatório |
| `pagina` | Número da página no PDF original de onde o chunk foi extraído | Campo próprio 1 |
| `heading_secao` | Título da seção/heading mais próximo (quando disponível, ex: no Markdown Splitter) | Campo próprio 2 |
| `contem_tabela` | Booleano indicando se o chunk contém sintaxe de tabela (`\|`) | Campo próprio 3 |

### Justificativa dos campos próprios

- **`pagina`**: permite citar exatamente em qual página do PDF original a informação apareceu. É essencial para um sistema de RAG que precise responder ao usuário "isso está na página X do documento Y", oferecendo rastreabilidade precisa até a fonte original.

- **`heading_secao`**: responde à pergunta "de qual seção do documento veio essa informação?". Saber se um trecho recuperado vem do "Resumo", da "Metodologia" ou das "Referências" ajuda tanto o LLM quanto o usuário final a avaliar a relevância e o contexto da resposta antes de confiar nela.

- **`contem_tabela`**: como identificado na análise da Aula 04, tabelas perdem parte da formatação padrão durante a conversão PDF → Markdown. Esse campo permite sinalizar e filtrar chunks que podem conter dados tabulares mal-formatados, possibilitando um tratamento especial (ou uma checagem manual) antes de usá-los em produção num sistema de RAG.

### Exemplos preenchidos com chunks reais

Foram utilizados dois chunks reais, de documentos diferentes, ambos gerados na Aula 04 com a estratégia Recursive Character Text Splitter (Teste 9), para demonstrar que o schema funciona de forma consistente independentemente do documento de origem.

**Exemplo 1 — documento `bioetica_e_ia.md`**

```json
{
  "fonte": "bioetica_e_ia.md",
  "documento_id": "bioetica_e_ia",
  "chunk_index": 0,
  "estrategia": "recursive",
  "chunk_size": 500,
  "chunk_overlap": 50,
  "n_caracteres": 405,
  "pagina": null,
  "heading_secao": null,
  "contem_tabela": true
}
```

**Exemplo 2 — documento `twitter_algoritmo.md`**

```json
{
  "fonte": "twitter_algoritmo.md",
  "documento_id": "twitter_algoritmo",
  "chunk_index": 0,
  "estrategia": "recursive",
  "chunk_size": 500,
  "chunk_overlap": 50,
  "n_caracteres": 465,
  "pagina": null,
  "heading_secao": null,
  "contem_tabela": false
}
```

**Observação sobre os campos `pagina` e `heading_secao`**: ambos aparecem como `null` nos exemplos, porque a extração de PDF→Markdown realizada na Aula 04 (via `pymupdf4llm`) não preservou marcação de número de página no texto, e a estratégia utilizada aqui (Recursive) não carrega metadados de heading — diferente do Markdown Header Text Splitter (Teste 10), que preserva essa informação. Essa é uma limitação já documentada no relatório da Aula 04.

**Observação sobre `contem_tabela`**: o campo é calculado de forma heurística e simples (`"|" in texto`), o que pode gerar falsos positivos — no Exemplo 1, o valor `true` não indica uma tabela de verdade, e sim a presença de um caractere `|` isolado no cabeçalho da revista (usado como separador visual, não como sintaxe de tabela). Isso é uma limitação conhecida do critério de detecção usado, não do schema em si.

### Respostas

**Qual campo você incluiria se precisasse citar a fonte na resposta final do RAG, informando ao usuário exatamente de onde veio a informação?**

A combinação de `fonte` (nome do arquivo) com `chunk_index` (posição exata dentro do documento) já permite uma citação básica, do tipo "segundo o documento `bioetica_e_ia.md` (trecho 3)...". Para uma citação mais precisa e amigável, o campo próprio `pagina` seria o ideal (ex: "página 5 do documento X") — ainda que, como observado acima, esse campo não esteja disponível na extração atual, sendo uma melhoria futura pendente do pipeline de extração.

**Por que `chunk_index` é útil? Pense no caso em que o trecho recuperado está cortado no meio de uma explicação.**

Porque ele permite reconstruir o contexto ao redor de um chunk recuperado. Se o sistema de RAG recupera o chunk de índice 5 e a resposta parece cortada ou incompleta, é possível usar `chunk_index` para buscar automaticamente os chunks vizinhos (índices 4 e 6, por exemplo) do mesmo `documento_id`, recuperando mais contexto antes de gerar a resposta final. Essa técnica é conhecida como expansão de contexto (ou "sentence-window retrieval") e é comum em implementações mais robustas de RAG.
