# Build ingestion + chunking pipeline

!!! success "Full lesson"
    Scheduled: **2026-08-12** · Week 6 · Phase 2. Turn real documents into a searchable index.

## Why this matters

So far you've indexed a handful of hand-typed sentences. Real RAG ingests *documents*: load files → chunk them well (Week 5's highest-leverage decision) → embed → store with metadata. This lesson builds that pipeline once, cleanly, so your agent can answer over an actual corpus next.

## The pipeline (indexing phase)

```
files → load text → chunk (with overlap, on boundaries) → embed (batched) → store (vector + text + source metadata)
```

## Step 1 — load

Start with plain text / Markdown (simplest). Read each file into a string, keep its path as the source.

```python
from pathlib import Path

def load_docs(folder):
    for path in Path(folder).rglob("*.md"):        # add *.txt etc. as needed
        yield str(path), path.read_text(encoding="utf-8")
```

(PDFs/HTML need extra parsing libraries — add later; don't let format wrangling distract from RAG.)

## Step 2 — chunk (the part that decides quality)

Apply Week 5's theory: ~200–500 tokens per chunk, 10–20% overlap, split on structure where possible. A simple, decent chunker splits on paragraphs and packs them up to a size budget with overlap:

```python
def chunk_text(text, max_chars=1500, overlap=200):
    paras = [p.strip() for p in text.split("\n\n") if p.strip()]
    chunks, cur = [], ""
    for p in paras:
        if len(cur) + len(p) + 2 <= max_chars:
            cur = f"{cur}\n\n{p}" if cur else p
        else:
            if cur:
                chunks.append(cur)
            # start next chunk with an overlap tail of the previous one
            cur = (cur[-overlap:] + "\n\n" + p) if cur else p
    if cur:
        chunks.append(cur)
    return chunks
```

(`max_chars≈1500` ≈ ~375 tokens at 4 chars/token. Tune it. This is intentionally simple — structure-aware chunkers exist in LangChain/LlamaIndex, but write one yourself first to understand the trade-offs.)

## Step 3 — embed (batched)

Embeddings endpoints accept a **list** — batch to cut round-trips (your provider allows up to 16 inputs per call). Respect that cap.

```python
def embed_batch(texts, model="v3small", batch=16):
    out = []
    for i in range(0, len(texts), batch):
        resp = client.embeddings.create(model=model, input=texts[i:i+batch])
        out.extend(d.embedding for d in resp.data)
    return out
```

## Step 4 — store with metadata

Give each chunk a stable id and record its `source` (and position) so you can cite it later.

```python
def ingest(folder, collection):
    ids, docs, metas = [], [], []
    for source, text in load_docs(folder):
        for j, ch in enumerate(chunk_text(text)):
            ids.append(f"{source}#{j}")
            docs.append(ch)
            metas.append({"source": source, "chunk": j})
    vecs = embed_batch(docs)
    collection.add(ids=ids, embeddings=vecs, documents=docs, metadatas=metas)
    print(f"Ingested {len(docs)} chunks from {folder}")
```

## Operational concerns (real-world)

- **Idempotency / re-ingest:** re-running with the same ids updates rather than duplicates (Chroma upserts by id) — good. If you change the chunker, clear the collection and re-ingest (new chunk boundaries = new ids/vectors).
- **Cost:** embedding a big corpus costs tokens. Batch, and on your course's provider, clear large runs with the maintainer.
- **Model consistency:** the query side (next lesson) must embed with the *same* model you ingest with.

## Exercise (~60 min)

1. Point `ingest` at a real folder of your own `.md`/`.txt` files (your notes, a project's docs, a few downloaded articles).
2. Inspect a few chunks by printing them — do they stand alone and stay on-topic? Adjust `max_chars`/`overlap` if not.
3. Ingest, then run 3 real questions against the collection (query embedding + `collection.query`). Do the retrieved chunks contain the answers, with correct `source`?
4. Change the chunk size deliberately (very small, then very large) and observe retrieval quality degrade — feel the trade-off from Week 5.

## Checkpoint — answer these

1. What are the four stages of the ingestion pipeline?
2. Why batch embeddings, and what cap must you respect on your provider?
3. Why store `source`/position metadata with each chunk?
4. If you change your chunking strategy, what must you do to the existing index, and why?
