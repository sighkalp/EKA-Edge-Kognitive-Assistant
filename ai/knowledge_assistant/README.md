# Knowledge Assistant Module

Architecture Version: 1.1 — Architecture Standardized
Last Updated: 2026-08-29

**Responsibility**: Answer grounded operational questions from approved mission documents using a custom Python RAG pipeline.

---

## Overview

The Knowledge Assistant is the repository’s grounded retrieval layer. It searches approved mission documents, converts them into embeddings, retrieves relevant passages, and uses a language model to answer questions while citing the source material.

This module is part of the Python service layer behind the FastAPI backend. The frontend never talks to the module directly; it asks the backend, and the backend orchestrates the assistant.

**Technology stack**:
- Python RAG pipeline
- PostgreSQL + pgvector
- sentence-transformers/all-MiniLM-L6-v2
- Gemini API as primary LLM
- Ollama + Qwen3-4B-Instruct-2507 as local/offline fallback
- PyMuPDF for PDF extraction
- Pytest for validation

---

## Inputs

### Query
- Natural-language question in English
- Example: "How do I secure the reaction chamber?"
- Optional mission context, experiment step, or current procedure state

### Sources
- Approved mission documents and SOP PDFs
- PDF content extracted with PyMuPDF

### Optional Context
- mission_id
- experiment_id
- current_step
- user role or scope

---

## Outputs

The Knowledge Assistant returns structured JSON compatible with the FastAPI layer and frontend-facing contract.

```json
{
  "query": "How do I secure the reaction chamber?",
  "answer": "To secure the chamber, confirm the seals are seated and tighten all fasteners before starting the reaction.",
  "confidence": 0.92,
  "sources": [
    {
      "document_id": "SOP_2024_v1",
      "section": "2.3.1",
      "page": 7,
      "snippet": "Secure the chamber before initiating the reaction."
    }
  ],
  "status": "success"
}
```

### Error shape

```json
{
  "query": "Can you tell me a joke?",
  "status": "error",
  "error_code": "QUERY_OUT_OF_SCOPE",
  "message": "This query is not related to approved mission documents."
}
```

---

## Retrieval and Generation

### Embeddings
- Model: sentence-transformers/all-MiniLM-L6-v2
- Embeddings are stored in PostgreSQL + pgvector
- Retrieval is semantic and grounded in approved mission content

### Generator
- Primary: Gemini API
- Local/offline fallback: Ollama + Qwen3-4B-Instruct-2507

### Document extraction
- PDF processing is handled with PyMuPDF
- The system should only answer using indexed approved documents

### Constraints
- The assistant is designed to answer only from approved mission content
- No untrusted or unapproved document should be indexed
- The repository standard uses PostgreSQL + pgvector, not ChromaDB or alternative vector stores
- Gemini is the primary online model; Ollama is the offline fallback

---

## Installation and Setup

```bash
python -m venv .venv
source .venv/bin/activate  # Windows: .venv\Scripts\activate
pip install -r ai/knowledge_assistant/requirements.txt
```

Create a local database and populate the vector store from approved PDFs.

```bash
python ai/knowledge_assistant/build_vectorstore.py \
  --documents data/mission_docs/ \
  --output data/vectorstore/
```

---

## Configuration

Example environment values:

```env
VECTOR_STORE_TYPE=postgresql
VECTOR_STORE_PATH=postgresql://localhost:5432/eka_db
EMBEDDINGS_MODEL=sentence-transformers/all-MiniLM-L6-v2
GEMINI_API_KEY=your-gemini-api-key
LLM_MODEL=gemini
OLLAMA_BASE_URL=http://localhost:11434
OLLAMA_MODEL=qwen3-4b-instruct-2507
```

The canonical deployment database remains PostgreSQL + pgvector. Local PostgreSQL + pgvector is the default development setup; Supabase PostgreSQL + pgvector is optional for cloud hosting.

---

## Usage

### Python usage

```python
from ai.knowledge_assistant import KnowledgeAssistant

assistant = KnowledgeAssistant(
    vector_store_path="postgresql://localhost:5432/eka_db",
    llm_model="gemini",
    llm_fallback="qwen3-4b-instruct-2507",
    embeddings_model="all-MiniLM-L6-v2"
)

response = assistant.query(
    question="How do I secure the chamber?",
    context={"mission_id": "mission_001", "step": 5}
)

print(response)
```

### Backend API usage

```bash
curl -X POST http://localhost:8000/api/knowledge/query \
  -H "Content-Type: application/json" \
  -d '{
    "query": "How do I secure the chamber?",
    "context": {"step": 5}
  }'
```

---

## Integration with EKA

- The frontend calls FastAPI.
- FastAPI calls the Knowledge Assistant module when a user asks a grounded question.
- The assistant retrieves passages from PostgreSQL + pgvector and generates a grounded answer.
- The answer is returned with source citations for user trust and auditability.

This module does not replace the backend. It is an internal Python service orchestrated by FastAPI.

---

## Testing

- Pytest for retrieval pipeline validation
- contract validation for answer payloads
- citation and source-grounding checks
- out-of-scope rejection checks for non-approved questions

Example checks:
- query returns citations
- confidence stays in a valid range
- answer does not include unsupported claims
- fallback path remains usable when Gemini is unavailable

---

## Troubleshooting

### Vector store not found
```bash
python ai/knowledge_assistant/build_vectorstore.py --documents data/mission_docs/ --output data/vectorstore/
```

### Low answer quality
- verify the document set is approved and indexed
- increase retrieval depth
- check that the correct embedding model is configured

### Fallback not working
- confirm Ollama is running locally
- verify `OLLAMA_BASE_URL` and `OLLAMA_MODEL`

---

## Dependencies

Core packages should be pinned in `ai/knowledge_assistant/requirements.txt` and should cover:
- sentence-transformers
- torch
- psycopg2 / SQLAlchemy / pgvector
- google-generativeai or equivalent Gemini SDK
- ollama
- PyMuPDF

---

## Acceptance Criteria

- Querying approved documents returns grounded answers with citations
- Source snippets match retrieved passages from the approved corpus
- Gemini is used as the primary generation path when available
- Ollama + Qwen3-4B-Instruct-2507 is the local fallback
- PostgreSQL + pgvector remains the canonical vector store
- The module never answers outside mission-approved content

---

## Notes

- The repository standard keeps FastAPI as the only application backend.
- The frontend remains a Next.js + React + TypeScript client only.
- The Knowledge Assistant is a backend internal service, not a frontend feature.

---

**Status**: Ready for implementation