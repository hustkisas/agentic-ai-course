# Weekly review + plan next week

!!! success "Weekly ritual (every Sunday)"
    Scheduled: **2026-08-09** · Week 5 · Phase 2. 30 minutes.

## The 5-question review

1. **What did I finish vs plan?** (Context/compaction? Token & cost tracking? Embeddings + brute-force search? RAG theory?)
2. **What slipped, and why?**
3. **Reschedule the slippage** into next week's slots.
4. **Did I build, or just read?** You should have added compaction + token/cost tracking to your agent, and a working brute-force semantic search.
5. **Are next week's blocks set?** Week 6 is the RAG build + milestone — protect the time.

## Week 5 self-check

Ready for Week 6 when you can:

- [ ] Explain why long agents break and how compaction fixes it (trigger + turn-boundary cut + preserved state).
- [ ] Estimate context tokens before a call and track $ cost from `usage`.
- [ ] Explain embeddings + cosine similarity, and implement a brute-force semantic search.
- [ ] Explain the RAG pipeline, why chunking matters most, and what re-ranking adds.

If your brute-force semantic search isn't working, fix that first next week — Week 6 builds directly on it.

## Looking ahead: Week 6

You'll turn the theory into a real RAG system: a proper vector DB (Chroma/FAISS), an ingestion + chunking pipeline, and — the payoff — a **`search_docs` tool wired into your Week 4 agent**, so it can answer questions grounded in *your* documents, with citations. That's the Week 6 milestone.

## Coaching note

Bring your compaction-enabled agent and your semantic-search code to the next session. Ask to be quizzed on the RAG pipeline and chunking trade-offs — Week 6 assumes them. Then get Week 6 (the RAG build) written into the site.
