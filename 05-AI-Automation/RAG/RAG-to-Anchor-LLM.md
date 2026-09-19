**Use RAG to anchor LLM outputs to real documents: ingest authoritative sources, index them as vectors, retrieve relevant passages at query time, and force the generator to cite and verify — this reduces hallucinations and makes factual pages auditable.**

### Overview

Retrieval‑Augmented Generation (RAG) combines a **retriever** (searching a document store) with a **generator** (LLM) so answers are built from retrieved evidence rather than only model memory. RAG is the standard enterprise approach to keep outputs current and verifiable.

---

### Step 1 — Ingest and preprocess source content

- **Collect**: gather PDFs, HTML pages, CSVs, calendar feeds, diocesan registries, and official church pages.
- **Normalize**: convert to plain text, remove boilerplate, and extract metadata (title, date, URL, language).
- **Chunk**: split long documents into coherent chunks (200–1,000 tokens) with overlap to preserve context.  
    **Why:** chunking keeps retrieval precise and fits model context windows.

---

### Step 2 — Vectorize and index

- **Embed**: use a stable embedding model to convert each chunk into a vector.
- **Index**: store vectors in a vector DB (e.g., OpenSearch, Pinecone, Milvus) with metadata for provenance.
- **Versioning**: tag indexes with source timestamps and content hashes so you can audit which snapshot produced an answer.  
    **Key point:** consistent embeddings and metadata are essential for traceability.

---

### Step 3 — Retriever design

- **First pass**: use approximate nearest neighbor (ANN) search to fetch top‑k candidate chunks.
- **Hybrid signals**: combine dense vectors with sparse signals (BM25) for keyword precision on names, dates, and addresses.
- **K selection**: tune k (commonly 5–20) by measuring downstream answer quality.  
    **Result:** a small, relevant evidence set that the generator can consume efficiently.

---

### Step 4 — Rerank and filter

- **Reranker**: apply a lightweight model or heuristic to reorder candidates by relevance and recency.
- **Safety filters**: remove content flagged for privacy, PII, or policy violations before generation.  
    **Important:** filtering prevents leaking sensitive records and reduces hallucination triggers.

---

### Step 5 — Prompting the generator

- **Context window**: prepend retrieved chunks with clear instructions and provenance (source, URL, date).
- **Constrain**: ask the LLM to _only_ use the provided chunks and to cite the source for each factual claim.
- **Fallback**: if evidence is insufficient, instruct the model to respond with “insufficient evidence” rather than guessing.  
    **Outcome:** generated text that is evidence‑backed and auditable.

---

### Step 6 — Post‑processing, verification, and human review

- **Automated checks**: run fact‑consistency tests, date/place normalization, and citation presence checks.
- **Human sign‑off**: route high‑risk pages (legal, liturgical, cemetery records) to editors for verification.
- **Logging**: store prompt, retrieved chunks, model output, and reviewer decisions for audits.

---

### Step 7 — Deploy, monitor, and iterate

- **Metrics**: track hallucination rate, citation coverage, retrieval latency, and editor override frequency.
- **Retraining**: refresh index on schedule and after major source updates.
- **A/B test**: compare different retrievers, k values, and prompt templates to optimize accuracy.

---

### Risks and mitigations

- **Stale data** → frequent reindexing and source timestamps.
- **PII leakage** → strict filters and redaction before indexing.
- **Over‑reliance on single source** → diversify sources and surface provenance.

---

### Quick checklist (student‑friendly)

- Ingest → Clean → Chunk → Embed → Index → Retrieve → Rerank → Prompt → Verify → Log.

**Sources:** .