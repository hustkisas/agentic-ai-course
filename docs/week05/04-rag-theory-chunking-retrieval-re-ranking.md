# RAG theory: chunking, retrieval, re-ranking

!!! success "Full lesson"
    Scheduled: **2026-08-09** · Week 5 · Phase 2. The theory you'll implement next week.

## Why this matters

**RAG (Retrieval-Augmented Generation)** is how you give an agent knowledge it wasn't trained on and can't fit in context: your documents, a codebase, a wiki. Instead of stuffing everything into the prompt, you **retrieve only the relevant pieces** (via last lesson's embeddings + vector search) and put *those* in the prompt. It's the most common real-world LLM pattern, and your Week 6 project.

## The pipeline

RAG has two phases.

**Indexing (offline, once):**
```
documents → chunk → embed each chunk → store vectors (+ the text) in a vector DB
```

**Querying (per request):**
```
user question → embed → search DB for top-k similar chunks → put chunks in the prompt → model answers grounded in them
```

The model then answers *from the retrieved text*, not from memory — which reduces hallucination and lets you cite sources.

## Chunking: the step that quietly decides quality

You can't embed a whole 50-page doc as one vector — it'd be too coarse to match anything well, and too big for the model's context when retrieved. So you split documents into **chunks**. This is the highest-leverage decision in RAG:

- **Too big:** each chunk mixes many topics → fuzzy embeddings → irrelevant retrievals, and you waste context.
- **Too small:** chunks lose the surrounding context needed to be meaningful → the model gets fragments.
- **Rule-of-thumb start:** ~200–500 tokens per chunk, with **10–20% overlap** between consecutive chunks so a sentence split across a boundary isn't lost.
- **Respect structure** where you can: split on paragraphs/headings/sentences, not mid-word. "Semantic" or structure-aware chunking beats blind fixed-size slicing.

Store each chunk's **source metadata** (file name, position) alongside its vector — you'll need it for citations.

## Retrieval: top-k and its knobs

- **k** = how many chunks to retrieve (start with 3–5). Too few → miss the answer; too many → dilute the prompt and raise cost.
- **Similarity threshold:** optionally drop chunks below a cosine score so a query with *no* good match returns nothing rather than garbage.
- **The context you build:** concatenate the retrieved chunks (with their sources) into the prompt, then instruct the model to answer *only* from them and cite which chunk it used.

## Re-ranking: a second, sharper pass

Vector search is fast but approximate about *relevance* — the top-k by cosine aren't always the *best* k. **Re-ranking** improves precision:

1. Retrieve a larger candidate set (say top-20) cheaply with vector search.
2. Score each candidate against the query with a more accurate but heavier model — a **cross-encoder / re-ranker** that reads (query, chunk) *together* (vs embeddings, which encode them separately).
3. Keep the top-5 after re-scoring.

This "retrieve broad, then re-rank narrow" pattern noticeably improves answer quality. (Your course's provider mentions a re-ranker endpoint in the works — a BGE model — exactly for this.)

## Failure modes (know them before you build)

- **Bad chunking** → the answer's text exists but got split so no chunk matches well. (Most common; fix chunking first.)
- **Retrieval miss** → k too small, or query phrased very differently from the source.
- **Right chunks, wrong answer** → model ignored them; fix with a firmer "answer only from the provided context" instruction.
- **No-answer case** → the docs don't contain it; the model should say "not found," not invent. Instruct + threshold for this.
- **Stale index** → docs changed but you didn't re-index.

## The RAG mindset

> RAG turns "what does the model know?" into "what can the model *look up*?" You're building a librarian: chunk well, retrieve the right pages, and make the model answer from them — with citations.

## Resources

- Anthropic / OpenAI RAG guides (retrieval + grounding).
- "Retrieval-Augmented Generation" (Lewis et al.) — the origin paper (skim the abstract/figure).
- Any practical "chunking strategies for RAG" write-up (LlamaIndex/LangChain docs have good ones).

## Exercise (~30–45 min, on paper + a little code)

1. Take one real document (a README, a paper, your own notes). Decide a chunking strategy: size, overlap, split boundary. Justify each choice.
2. Manually chunk the first page and eyeball whether each chunk stands alone / stays on one topic.
3. Extend last lesson's brute-force `search` to (a) chunk a document, (b) embed chunks, (c) retrieve top-k for a question. Print the retrieved chunks — do they contain the answer?
4. Write down which failure mode you'd expect to hit first with your document, and why.

## Checkpoint — answer these

1. Draw the two RAG phases (indexing and querying) as pipelines.
2. Why is chunking the highest-leverage decision, and what are the symptoms of too-big vs too-small chunks?
3. What does re-ranking add on top of vector search, and why is "retrieve broad, re-rank narrow" effective?
4. Name two RAG failure modes and how you'd mitigate each.
