# RAG Agent — Production-grade Document Q&A

A production-quality RAG (Retrieval-Augmented Generation) pipeline that answers
questions about your documents with source citations.

Built to demonstrate real agentic AI engineering skills for €85k+ roles.

---

## What this does differently from tutorial RAG

| Feature | Basic tutorial | This project |
|---|---|---|
| Chunking | Fixed size | Recursive, semantic-aware |
| Query | Raw question → embed | HyDE expansion + multi-query |
| Retrieval | Top-k cosine | Top-k cosine + Cohere rerank |
| Citations | None | Every fact cited with page |
| Accuracy proof | "trust me" | RAGAS eval scores |
| Observability | None | LangSmith trace every step |

## Stack

- **LangChain** — orchestration
- **OpenAI** `text-embedding-3-small` — embeddings
- **Qdrant** — vector store (local in-memory or Docker)
- **Cohere** `rerank-english-v3.0` — reranker (free tier)
- **RAGAS** — evaluation framework
- **LangSmith** — observability and tracing

## Setup

```bash
# 1. Clone and install
pip install -r requirements.txt

# 2. Set your API keys
cp .env.example .env
# Edit .env — add your OPENAI_API_KEY (required)
# Add COHERE_API_KEY for reranking (free at cohere.com)
# Add LANGCHAIN_API_KEY for tracing (free at smith.langchain.com)

# 3. Add your PDFs
cp your_documents.pdf data/

# 4. Run
python main.py --mode chat       # ask questions
python main.py --mode eval       # run RAGAS evaluation
python main.py --mode all        # ingest + eval + chat
```

## Eval results (on sample dataset)

After running `python main.py --mode eval` you'll see something like:

```
  v1 basic:
    Faithfulness       : 0.71
    Answer relevancy   : 0.74
    Context precision  : 0.68

  v2 HyDE + rerank:
    Faithfulness       : 0.89   ↑ 0.18
    Answer relevancy   : 0.91   ↑ 0.17
    Context precision  : 0.86   ↑ 0.18
```

These numbers are what you show in interviews and your GitHub README.

## Architecture

```
INGESTION PIPELINE
Documents → Recursive Chunking → Embedding → Qdrant Vector DB

QUERY PIPELINE (v2)
Question → Multi-Query Expansion (3 phrasings)
         → Retrieve top-8 chunks from Qdrant
         → Cohere Rerank → top-3 chunks
         → GPT-4o-mini with citation prompt
         → Answer with [Source: file, page X] citations

EVAL LAYER
RAGAS: faithfulness, answer_relevancy, context_precision, context_recall
```

## Files

```
rag_agent/
├── main.py              # Entry point — run everything from here
├── src/
│   ├── ingest.py        # Milestone 1: load, chunk, embed, store
│   ├── query.py         # Milestone 1: basic retrieval + LLM chain
│   ├── query_v2.py      # Milestone 2: HyDE + reranker chain
│   └── eval.py          # Milestone 3: RAGAS evaluation suite
├── data/                # Put your PDFs here
├── evals/               # Eval datasets and results
└── .env.example         # API key template
```
