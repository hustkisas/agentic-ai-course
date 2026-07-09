# Wire retrieval into your agent as a tool

!!! success "Full lesson"
    Scheduled: **2026-08-14** · Week 6 · Phase 2. The moment RAG becomes *agentic*.

## Why this matters

There are two ways to use retrieval, and the difference is the whole point of this course:

- **Static RAG (a pipeline):** always retrieve, always stuff chunks in the prompt, always answer. Simple; a *workflow* (Week 2's "Building Effective Agents" language).
- **Agentic RAG (retrieval as a tool):** give your Week 4 agent a `search_docs` tool and let *it decide* when to search, what to search for, and whether to search again. That's an *agent*.

Agentic RAG is more powerful: the agent can reformulate a bad query, search multiple times for a multi-part question, or skip retrieval entirely when it already knows the answer. You've built the agent; now you give it a library card.

## The move: retrieval becomes a tool

Everything you built in Weeks 5–6 (embed query → vector search → return chunks) becomes the body of one tool function. Register it exactly like `add` from Week 4.

```python
def search_docs(query: str, k: int = 4) -> str:
    q_vec = client.embeddings.create(model="v3small", input=[query]).data[0].embedding
    res = collection.query(query_embeddings=[q_vec], n_results=k)
    docs = res["documents"][0]
    metas = res["metadatas"][0]
    if not docs:
        return "No relevant documents found."
    # return chunks WITH their sources so the model can cite
    return "\n\n".join(f"[source: {m['source']}]\n{d}" for d, m in zip(docs, metas))

registry.register(Tool(
    name="search_docs",
    description="Search the user's document collection for passages relevant to a query. "
                "Use this whenever the question might be answered by the user's documents. "
                "Returns passages with their source. Call again with a refined query if results are weak.",
    parameters={"type": "object",
        "properties": {
            "query": {"type": "string", "description": "What to search for, in natural language."},
            "k": {"type": "integer", "description": "How many passages to return (default 4)."}},
        "required": ["query"]},
    fn=search_docs,
))
```

That `description` is doing heavy lifting (Week 2): it tells the model *when* to reach for the tool and hints that it can *re-query*. Vague description → the agent won't use it well.

## Make the agent cite and stay grounded

Add a system-prompt instruction so the agent uses retrieved text faithfully:

```python
SYSTEM = {"role": "system", "content":
    "You answer questions using the user's documents. Use the search_docs tool to find "
    "relevant passages. Answer ONLY from retrieved passages; if they don't contain the "
    "answer, say you couldn't find it. Cite the source of each fact you use."}
```

This directly targets the RAG failure modes from Week 5: "right chunks, wrong answer" (grounding) and "no-answer case" (don't hallucinate).

## Why this is better than static RAG — see it in the trace

Ask a two-part question ("What does the onboarding doc say about VPN, and who is the IT contact?"). Watch the agent (from your Week 3 tracing):

```
[step 1] ACTION: search_docs({"query": "VPN onboarding setup"})
[step 1] OBSERVATION: [source: onboarding.md] ...VPN instructions...
[step 2] ACTION: search_docs({"query": "IT contact / help desk"})
[step 2] OBSERVATION: [source: contacts.md] ...IT contact...
[step 3] final answer (cites onboarding.md and contacts.md)
```

Two targeted searches, driven by the agent's own reasoning — something static "retrieve once" RAG can't do. This is your Week 1 loop + Week 4 tools + Week 5–6 retrieval, composed.

## Exercise (~45–60 min)

1. Add `search_docs` (backed by your Week 6 collection) to your Week 4 agent's registry.
2. Add the grounding/citation system prompt.
3. Ask a single-fact question — confirm one search and a cited answer.
4. Ask a two-part question — confirm the agent searches twice (read the trace) and cites both sources.
5. Ask something **not** in your docs — confirm it says "couldn't find it" rather than inventing an answer.

## Checkpoint — answer these

1. What's the difference between static RAG and agentic RAG, in "workflow vs agent" terms?
2. What does making retrieval a *tool* let the agent do that a fixed pipeline can't?
3. Which parts of the tool `description` and system prompt target specific RAG failure modes from Week 5?
4. Trace how your loop + tools + retrieval compose to answer a two-part question.
