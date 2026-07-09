# PROJECT: RAG agent over your own docs

!!! success "Milestone project"
    Scheduled: **2026-08-16** · Week 6 · Phase 2 · **MILESTONE**. Completes Phase 2 — a genuinely useful agent.

## Goal

Combine everything from Phase 2 into a **RAG agent**: your from-scratch agent (Week 4) + a persistent vector index of *your own documents* (Week 6) exposed as a `search_docs` tool (Week 6 day 3). It answers real questions grounded in your corpus, decides for itself when to search, searches multiple times when needed, and **cites its sources**. This is the first thing you've built that you'd actually use.

## Choose your corpus

Pick a document set you care about — this makes the project real:

- your own notes / a personal knowledge base,
- a project's documentation or a codebase's `.md` files,
- a small set of papers or articles you've downloaded,
- (careful with anything sensitive — this is a learning project; keep it to non-confidential material).

20–200 chunks is a good size: big enough to be non-trivial, small enough to iterate fast.

## Requirements (definition of done)

1. **Ingestion (Week 6 days 1–2):** a script that loads your docs, chunks them (sensible size + overlap), embeds them, and stores them in a persistent Chroma collection with `source` metadata. Re-runnable without duplicating.
2. **Retrieval tool (Week 6 day 3):** `search_docs(query, k)` registered in your agent's tool registry, returning passages **with sources**.
3. **The agent (Week 4):** your robust loop — registry, error-as-observation, retry, iteration guard, tracing.
4. **Grounding:** system prompt instructs "answer only from retrieved passages; say so if not found; cite sources."
5. **Agentic behavior:** on a multi-part question, the agent issues **more than one** search (visible in the trace).
6. **Citations:** answers name the source file(s) used.
7. **Honest no-answer:** for a question not covered by your docs, it declines instead of hallucinating.
8. **A small eval set:** 5 questions with known answers from your docs (+ 1 that isn't), all behaving correctly at `temperature=0`.

## Structure

```
rag_agent/
├── ingest.py         # load -> chunk -> embed -> Chroma  (run once / on doc changes)
├── agent.py          # your Week 4 agent + search_docs tool + grounding prompt
└── evals.py          # 6 test questions; assert correct/declined behavior
```

## Stretch goals (optional)

- **Re-ranking (Week 5):** retrieve top-15, then re-score with a stronger pass (or your provider's re-ranker when available), keep top-4. Compare answer quality.
- **Query reformulation:** prompt the agent to rephrase and re-search when the first results are weak — and watch it do so in the trace.
- **Show retrieved context:** print the chunks the agent retrieved alongside its answer, so you can audit grounding.
- **PDF/HTML ingestion:** add a parser so it works on more than plain text/Markdown.
- **Chat mode:** loop on user input for a real Q&A session over your docs.

## Reflection (short written answers)

1. Trace a two-part question end to end: each search query, the sources retrieved, and the final cited answer.
2. Which RAG failure mode did you hit first (bad chunking? retrieval miss? grounding?), and how did you fix it?
3. Where does each Phase-2 piece show up: the loop, tools, compaction/token-budget, embeddings, vector DB, retrieval-as-tool?
4. What would you improve with re-ranking or better chunking — and how would you *measure* that improvement? (You're describing Week 10 evals.)

## Done means done — and Phase 2 is complete

When your agent answers real questions from *your* documents, searches more than once when needed, cites sources, and honestly declines what it can't find — **you've completed Phase 2, the heart of the course.** You now have, built by hand: the agent loop, tools, robustness, context management, embeddings, a vector store, and agentic retrieval. From here the course *extends* this: frameworks & multi-agent (Phase 3), harness engineering (Phase 4), evaluation (Phase 5), and deployment (Phase 6).
