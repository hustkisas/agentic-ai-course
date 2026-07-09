# Weekly review + plan next week

!!! success "Weekly ritual (every Sunday)"
    Scheduled: **2026-08-16** · Week 6 · Phase 2. 30 minutes — and the end of Phase 2, so reflect a bit more.

## The 5-question review

1. **What did I finish vs plan?** (Vector DB set up? Ingestion pipeline? `search_docs` tool wired in? RAG milestone?)
2. **What slipped, and why?**
3. **Reschedule the slippage** into next week's slots.
4. **Did I build the milestone?** A working RAG agent over your own docs is the Phase-2 capstone deliverable.
5. **Are next week's blocks set?** Week 7 begins Phase 3 (frameworks: LangGraph, MCP).

## Week 6 self-check

You've completed Phase 2 when you can:

- [ ] Stand up a persistent vector DB and query it with your provider's embeddings.
- [ ] Ingest real documents with a sensible chunking pipeline.
- [ ] Expose retrieval as a tool so the agent decides when/what to search.
- [ ] **Run a RAG agent** over your own docs that searches (sometimes more than once), cites sources, and honestly declines unknowns — passing a small eval set at `temperature=0`.

## Phase 2 retrospective (you just did the hard part)

Look back over Weeks 3–6. From scratch, no framework, you built: the agent loop, a tool registry, robust error handling + retries + guards, context compaction, token/cost tracking, embeddings + vector search, a vector DB, an ingestion pipeline, and agentic retrieval. **That is the entire core of applied agentic AI.** Most practitioners only ever wire these together through a framework; you understand what the framework is doing. Everything remaining builds on this foundation.

Write 3 sentences: what clicked, what's still fuzzy, and what you're most proud of.

## Looking ahead: Phase 3 (Weeks 7–8)

Now that you've built it all by hand, you'll learn the **frameworks** (LangGraph, the Agents SDKs, MCP) — and *appreciate* them, because you know what they abstract. Then **multi-agent** patterns: supervisors and workers coordinating (studying Pi's orchestrator).

## Coaching note

Bring your RAG agent to the next session. Ask to be quizzed on the full Phase-2 stack (loop → tools → robustness → context → RAG) — it's the foundation Phase 3 assumes. Then get Phase 3 written into the site.
