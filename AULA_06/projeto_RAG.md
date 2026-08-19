Aula 06 – Projeto e Arquitetura RAG
Residência em Trilhas de Tecnologias em IA Generativa – PUC-Rio Ecoa

------------------------------------------------------------
Cenário 1 – Agente de Reembolso
------------------------------------------------------------

Parte 1 – Limitações
- Documentos normativos complexos e em constante atualização.
- Recibos médicos escaneados com baixa qualidade.
- Necessidade de respeitar LGPD (dados sensíveis).
- Custo elevado de processamento em grande volume.

Parte 2 – Organização dos documentos
Tipos de arquivos: PDFs, planilhas, imagens escaneadas, DOCX/Markdown.
Volume: dezenas de regulamentos, centenas de circulares, milhares de recibos.
Tamanho típico: regulamentos 50–100 páginas, circulares 5–20 páginas, recibos 1 página.
Frequência: regulamentos anuais, circulares mensais/trimestrais, recibos diários.

Estrutura de pastas:
```text
documentos/
├── regulamentos/
├── circulares/
├── tabelas/
├── recibos/
└── faq/
```



Parte 3 – Pipeline de ingestão
Documentos → Extração (PDF parser, OCR, Pandas) → Limpeza → Metadados → Chunking → Embeddings → Banco Vetorial

Parte 4 – Metadados
Documento:
```json
{
  "document_id": "reg_2026",
  "title": "Regulamento Geral 2026",
  "author": "Operadora Saúde",
  "source": "regulamentos/reg_2026.pdf",
  "document_type": "regulamento",
  "created_at": "2026-01-01",
  "updated_at": "2026-07-01",
  "category": "normativo"
}
```


Chunk:
{
  "document_id": "reg_2026",
  "chunk_id": "reg_2026-12",
  "page": 12,
  "section": "Reembolso Terapias",
  "document_type": "regulamento",
  "valid_from": "2026-01-01",
  "valid_to": "2026-12-31",
  "text": "Artigo 12: Limite de R$200 por sessão"
}

Parte 5 – Chunking / Splitting
- Estratégia: dividir por seções semânticas.
- Tamanho: 800–1000 caracteres.
- Overlap: 200 caracteres.
- Tabelas: manter inteiras ou por linha.
- Imagens: OCR para texto.
- Evidência: testes de recuperação e cobertura.

Parte 6 – Embeddings
Modelo: OpenAI text-embedding-3-large
Dimensão: 3072
Suporta português: Sim
Multilíngue: Sim
Tamanho máximo: 8192 tokens
Open source: Não
Local: Não
API: Sim
Custo: US$0,13 por 1M tokens
Fonte: OpenAI Docs

Justificativa: alta precisão semântica em português.
Alternativas descartadas: modelos menores ou open-source.
Sigilo: anonimizar dados antes de enviar.
Chunking: limite de tokens suporta chunks definidos.

Arquitetura final:
Documentos → Extração → Limpeza → Metadados → Chunking → Embeddings → Banco Vetorial → RAG Agent → Decision Agent → Resposta

Riscos e limitações:
- RAG não resolve cálculos agregados.
- OCR pode falhar.
- API externa exige anonimização.
- Custo elevado.

------------------------------------------------------------
Cenário 2 – Regulamento Acadêmico
------------------------------------------------------------

Parte 1 – Limitações
- Documentos menos sigilosos.
- Atualização semestral/anual.
- Menor criticidade financeira.

Parte 2 – Organização dos documentos
Tipos: PDFs de regulamentos, DOCX de ementas, planilhas de calendário.
Volume: dezenas de regulamentos, centenas de ementas.
Tamanho: regulamentos 30–50 páginas, ementas 2–5 páginas.
Frequência: semestral/anual.

Estrutura:
documentos/
├── regulamentos/
├── ementas/
├── calendario/
└── comunicados/

Parte 3 – Pipeline
Mesma estrutura, menos OCR.

Parte 4 – Metadados
Adicional: semester, academic_year.

Parte 5 – Chunking
- Divisão por artigos e parágrafos.
- Chunks menores (~600–800).
- Overlap 150.
- Tabelas mantidas inteiras.

Parte 6 – Embeddings
Modelo: Instructor-xl (open-source)
Dimensão: 768
Suporta português: Sim
Multilíngue: Sim
Tamanho máximo: ~512 tokens
Open source: Sim
Local: Sim
API: HuggingFace
Custo: Gratuito
Fonte: HuggingFace Models

Justificativa: uso local reduz custo e risco.
Alternativas descartadas: OpenAI embeddings.
Chunking: limite menor exige chunks menores.

Arquitetura final:
Documentos → Extração → Limpeza → Metadados → Chunking → Embeddings (local) → Banco Vetorial → RAG Agent → Resposta

Riscos e limitações:
- Modelo local menos preciso.
- Chunking restrito.
- Risco de confusão em versões antigas.

------------------------------------------------------------
Comparação entre os dois cenários
------------------------------------------------------------
Diferentes:
- Tipo de documento.
- Frequência de atualização.
- Criticidade.
- Modelo de embeddings.

Iguais:
- Pipeline de ingestão.
- Uso de metadados.
- Chunking com overlap.
Boa prática geral.

Escolha prioritária:
Agente de Reembolso, por maior impacto e criticidade.

------------------------------------------------------------
Referências
------------------------------------------------------------
- OpenAI Embeddings Documentation
- HuggingFace Instructor-xl
- Vídeos no YouTube sobre RAG e chunking
- Materiais técnicos sobre LGPD

------------------------------------------------------------
Uso de IA
------------------------------------------------------------
Utilizei a IA (Microsoft Copilot) como apoio na estruturação das etapas e na redação das propostas.
Busquei vídeos no YouTube e materiais técnicos sobre RAG para compreender melhor os conceitos.
Avaliei criticamente cada resposta frente às fontes externas.
A decisão sobre cada técnica levou em conta tempo, formatos de dados, criticidade quanto ao sigilo (LGPD) e custo.
