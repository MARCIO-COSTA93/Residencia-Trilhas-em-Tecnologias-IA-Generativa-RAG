\# Introdução à IA - Aula 02



Este projeto contém o código para conversão de documentos PDF em Markdown utilizando a biblioteca Docling, além da extração de metadados estruturados (título, autores e ano) utilizando Structured Outputs via API.



\## Passo a Passo para Configuração e Execução



\### 1. Ativar o Ambiente Virtual



```bash

\# No Linux/macOS

source venv/bin/activate



\# No Windows

venv\\Scripts\\activate

```



\### 2. Instalar o Docling



```bash

pip install docling

```



Documentação oficial: https://docling-project.github.io/docling/getting\_started/installation/



\### 3. Converter os PDFs para Markdown



Os arquivos PDF originais (bioética e IA, escrita acadêmica, algoritmo do Twitter) foram convertidos para Markdown utilizando o Docling, preservando títulos, parágrafos e estrutura do documento.



\### 4. Extrair Metadados com Structured Outputs



Para cada arquivo `.md` gerado, foi utilizada uma chamada de API (via OpenRouter, modelo `openai/gpt-oss-20b:free`) com Structured Outputs para extrair:



\- \*\*Título\*\* do trabalho

\- \*\*Autores\*\* (lista)

\- \*\*Ano\*\* de publicação



O resultado é salvo em um arquivo `.json` correspondente, no formato:



```json

{

&#x20; "titulo": "Título do trabalho",

&#x20; "autores": \["Autor 1", "Autor 2"],

&#x20; "ano": 2024

}

```



\## Arquivos desta pasta



| Arquivo | Descrição |

|---|---|

| `Aula\_02.ipynb` | Notebook completo com todo o processo (conversão + extração de metadados) |

| `bioetica\_e\_ia.md` / `.json` | Conversão e metadados do artigo sobre bioética e IA |

| `escrita\_academica\_ia.md` / `.json` | Conversão e metadados do artigo sobre escrita acadêmica |

| `twitter\_algoritmo.md` / `.json` | Conversão e metadados do artigo sobre algoritmo do Twitter |



\## Como sair do Ambiente Virtual?



```bash

deactivate

```

