# Embeddings & vector search fundamentals

!!! success "Full lesson"
    Scheduled: **2026-08-07** · Week 5 · Phase 2. Your algorithms background is a real edge here — enjoy this one.

## Why this matters

To give an agent knowledge that doesn't fit in the context window (your docs, a codebase, a knowledge base), you retrieve only the *relevant* pieces on demand. Finding "relevant" pieces by meaning — not keyword matching — is done with **embeddings + vector search**. This is the machinery under RAG (next lesson) and under your Week 6 build.

## What an embedding is

An **embedding** is a fixed-length vector of floats (e.g. 1,536 dims) that represents the *meaning* of a piece of text. A model is trained so that **texts with similar meaning map to nearby vectors**. "dog" and "puppy" land close; "dog" and "tax law" land far apart. It turns semantics into geometry.

You get them from an embeddings endpoint (your provider has one — OpenAI-compatible `/v1/embeddings`, models like `v3small` → 1,536 dims, `v3large` → 3,072):

```python
resp = client.embeddings.create(model="v3small", input=["a dog", "a puppy", "corporate tax"])
vecs = [d.embedding for d in resp.data]   # three 1,536-dim vectors
```

## Measuring similarity: cosine similarity

Similarity = how close two vectors point in the same direction. **Cosine similarity** is the standard measure:

$$\cos(\mathbf{a},\mathbf{b}) = \frac{\mathbf{a}\cdot\mathbf{b}}{\lVert\mathbf{a}\rVert\,\lVert\mathbf{b}\rVert}$$

Range −1…1; higher = more similar. For unit-normalized vectors it's just the dot product. This will be very familiar territory for you:

```python
import numpy as np
def cosine(a, b):
    a, b = np.array(a), np.array(b)
    return float(a @ b / (np.linalg.norm(a) * np.linalg.norm(b)))
```

Why cosine and not Euclidean distance? Embeddings encode meaning mostly in *direction*, not magnitude; cosine ignores length and compares orientation. (For normalized vectors the two are monotonically related anyway.)

## Vector search = nearest neighbors

"Find the most relevant chunks to this query" becomes: **embed the query, then find the vectors nearest to it** (highest cosine). That's k-nearest-neighbors in a high-dimensional space.

- **Brute force:** compare the query to every stored vector. O(N·d). Perfectly fine for thousands of chunks — and worth writing once yourself to demystify it.
- **Approximate NN (ANN):** for millions of vectors, structures like HNSW graphs or IVF indexes find near-neighbors in ~sublinear time by trading a little accuracy for speed. This is what vector DBs (FAISS, Chroma — Week 6) implement so you don't have to.

An algorithms note you'll appreciate: high dimensions make exact NN structures (kd-trees) degrade (the "curse of dimensionality"), which is *why* ANN methods like HNSW exist.

## Brute-force retrieval, end to end

```python
docs = ["Paris is the capital of France.", "The mitochondria is the powerhouse of the cell.",
        "France's capital city is Paris."]
doc_vecs = [d.embedding for d in client.embeddings.create(model="v3small", input=docs).data]

def search(query, k=2):
    q = client.embeddings.create(model="v3small", input=[query]).data[0].embedding
    scored = sorted(((cosine(q, v), doc) for v, doc in zip(doc_vecs, docs)), reverse=True)
    return scored[:k]

print(search("What is the capital of France?"))
# -> the two France sentences rank top, by MEANING, even without the word "capital" matching exactly
```

That semantic match — retrieving "France's capital city is Paris" for a query about "capital of France" without exact keyword overlap — is the whole point.

## Resources

- **OpenAI — "Embeddings" guide** (what they are, how to call them, use cases).
- **"Cosine similarity"** (any linear-algebra reference — you know this).
- Optional: FAISS wiki intro on ANN indexes (HNSW/IVF) for the "how do vector DBs scale" question.

## Exercise (~45–60 min)

1. Embed 6–8 short sentences on 2–3 topics with your provider's embeddings endpoint.
2. Implement `cosine` and a brute-force `search(query, k)`.
3. Query with paraphrases that share *meaning* but not keywords; confirm the right sentences rank top.
4. (Stretch) Time brute-force search as you scale to a few thousand random vectors — feel where you'd want ANN.

## Checkpoint — answer these

1. What does an embedding represent, and what's true about the vectors of two similar texts?
2. Why cosine similarity rather than Euclidean distance for embeddings?
3. What does "vector search" compute, and when do you need ANN instead of brute force?
4. Why does semantic search find relevant text that keyword search would miss?
