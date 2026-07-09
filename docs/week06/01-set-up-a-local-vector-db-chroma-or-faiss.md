# Set up a local vector DB (Chroma or FAISS)

!!! success "Full lesson"
    Scheduled: **2026-08-10** · Week 6 · Phase 2. Trade your brute-force search for a real vector store.

## Why this matters

Last week you wrote brute-force cosine search — perfect for learning, fine for a few thousand vectors. A **vector database** does the same job but adds: persistence (index once, reuse), fast approximate nearest-neighbor search at scale, and metadata storage/filtering (for citations). This week you build a real RAG system on top of one.

## Which one?

- **Chroma** — easiest to start: `pip install chromadb`, runs embedded (in-process), stores to disk, handles embeddings + metadata for you. **Recommended for this course.**
- **FAISS** — a fast ANN *library* (from Meta), not a full DB: you manage the vectors and metadata yourself. More control, more moving parts. Great to know exists; heavier to wire up.

We'll use **Chroma**. The concepts transfer directly to any vector store.

## The mental model (same as your brute-force version, productized)

A vector DB stores records of `(id, vector, document_text, metadata)` and answers "give me the k records whose vectors are nearest this query vector." That's exactly your `search()` from Week 5 — the DB just makes it persistent, fast, and filterable.

## Chroma quick start

```python
import chromadb

client_db = chromadb.PersistentClient(path="./chroma_store")   # persists to disk
collection = client_db.get_or_create_collection("my_docs")

# add documents (Chroma can embed for you, but we'll pass our own vectors
# so you keep using your provider's embeddings endpoint)
collection.add(
    ids=["doc1", "doc2", "doc3"],
    embeddings=[vec1, vec2, vec3],                  # from your provider's /embeddings
    documents=["Paris is the capital of France.",
               "Mitochondria are the powerhouse of the cell.",
               "France's capital city is Paris."],
    metadatas=[{"source": "geo.txt"}, {"source": "bio.txt"}, {"source": "geo.txt"}],
)

# query
q_vec = client.embeddings.create(model="v3small", input=["capital of France?"]).data[0].embedding
res = collection.query(query_embeddings=[q_vec], n_results=2)
print(res["documents"])   # the top-2 matching chunks
print(res["metadatas"])   # their sources -> citations
```

Why pass your **own** embeddings (`embeddings=[...]`) rather than let Chroma embed? Because you want to use *your provider's* embeddings endpoint (the one your agent already uses), keeping the whole system on one model family and one credential. Chroma is then purely the store + search.

!!! note "Consistency rule"
    Index and query **must use the same embedding model**. Vectors from different models aren't comparable. If you re-index with a new model, re-embed everything.

## What you're getting over brute force

- **Persistence:** `PersistentClient(path=...)` writes to disk — embed once, restart, still there.
- **Speed at scale:** Chroma uses ANN under the hood (HNSW), so it stays fast as your collection grows past what brute force can handle.
- **Metadata + filtering:** store `source`, `page`, etc., and even filter queries (`where={"source": "geo.txt"}`) — essential for citations and scoping.

## Resources

- **Chroma docs** — "Getting Started" (collections, add, query, persistence).
- Optional: **FAISS wiki** — read the "Getting started" + an ANN index page to understand what Chroma does internally.

## Exercise (~45–60 min)

1. `pip install chromadb`. Create a `PersistentClient` collection.
2. Take your 6–8 sentences from Week 5, embed them with your provider, and `add` them (with `source` metadata).
3. Query with a paraphrased question; confirm you get the right documents *and* their sources back.
4. Restart your script (don't re-add) and query again — confirm persistence works (the index survived).
5. (Stretch) Add a `where={"source": ...}` filter and see scoped retrieval.

## Checkpoint — answer these

1. What does a vector DB add over your Week 5 brute-force search (name three things)?
2. Why pass your own embeddings to Chroma instead of letting it embed?
3. What's the consistency rule between indexing and querying, and what breaks if you violate it?
4. Why does storing metadata (source) matter for a RAG agent?
