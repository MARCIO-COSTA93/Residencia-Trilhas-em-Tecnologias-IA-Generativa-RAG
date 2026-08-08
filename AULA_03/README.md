\# Introducao a IA - Aula 03



Este projeto contem exercicios praticos sobre Embeddings - representacoes vetoriais de texto usadas em busca semantica e sistemas de Retrieval-Augmented Generation (RAG).



\## Estrutura do Projeto



AULA\_03/

├── README.md              # Este arquivo

└── Aula\_03\_a.ipynb        # Tarefa A: Geracao e visualizacao de embeddings



\## Passo a Passo para Configuracao e Execucao



\### 1. Ativar o Ambiente Virtual



\# No Linux/macOS

source venv/bin/activate



\# No Windows

venv\\Scripts\\activate



\### 2. Instalar as Dependencias



pip install openai scikit-learn matplotlib numpy



\## Tarefa A - Geracao e Visualizacao de Embeddings



Arquivo: Aula\_03\_a.ipynb



\### Arquitetura do Codigo



┌─────────────────────────────────────────────┐

│              Aula\_03\_a.ipynb                 │

├─────────────────────────────────────────────┤

│                                               │

│  1. Configuracao                             │

│     - client (OpenAI -> OpenRouter)          │

│                                               │

│  2. Funcoes                                  │

│     - gerar\_embeddings\_em\_lote()             │

│       entrada: lista de textos               │

│       saida: vetores + tokens gastos         │

│                                               │

│     - reduzir\_dimensoes()                    │

│       entrada: vetores (2048D)               │

│       saida: coordenadas (3D) via PCA        │

│                                               │

│     - plotar\_grafico\_3d()                    │

│       entrada: coordenadas + categorias      │

│       saida: grafico 3D com legenda          │

│                                               │

│  3. Execucao                                 │

│     palavras -> embeddings -> PCA -> grafico │

│                                               │

└─────────────────────────────────────────────┘



\### Fluxo de dados



"gato", "carro", "banana"...

&#x20;       |

&#x20;       v

+--------------------+

|  OpenRouter API     |  (nvidia/nemotron-3-embed-1b:free)

+--------------------+

&#x20;       |

&#x20;       v

&#x20;  Vetores 2048D

&#x20;       |

&#x20;       v

+--------------------+

|       PCA           |  (reduz para 3 dimensoes)

+--------------------+

&#x20;       |

&#x20;       v

&#x20;  Coordenadas 3D

&#x20;       |

&#x20;       v

+--------------------+

|   Matplotlib 3D      |  (grafico com legenda por categoria)

+--------------------+



\### O que foi feito



1\. Geracao de embeddings: utilizando o modelo gratuito nvidia/nemotron-3-embed-1b:free via OpenRouter, foram gerados vetores de 2048 dimensoes para 9 palavras de 3 categorias diferentes:

&#x20;  - Animais: gato, felino, cachorro

&#x20;  - Veiculos: carro, caminhao, moto

&#x20;  - Frutas: banana, maca, goiaba



2\. Reducao de dimensionalidade: como nao e possivel visualizar 2048 dimensoes diretamente, foi utilizado o algoritmo PCA (Principal Component Analysis) para reduzir os vetores a 3 dimensoes, preservando ao maximo as relacoes de distancia entre eles.



3\. Visualizacao 3D: os vetores reduzidos foram plotados em um grafico 3D, coloridos por categoria, permitindo observar visualmente como palavras de significado semelhante tendem a ficar mais proximas no espaco vetorial.



\### Otimizacoes aplicadas



\- Chamada unica a API: em vez de gerar um embedding por palavra (9 chamadas), todas as palavras foram enviadas em uma unica requisicao, reduzindo overhead de rede.

\- Codigo modularizado em funcoes, com docstrings documentando parametros e retornos.



\### Conceitos aprendidos



\- O que e um embedding e por que ele e util

\- Como gerar embeddings via API (OpenRouter)

\- Consumo de tokens em chamadas de embedding

\- Reducao de dimensionalidade com PCA

\- Visualizacao de dados de alta dimensao



\## Como sair do Ambiente Virtual?



deactivate



