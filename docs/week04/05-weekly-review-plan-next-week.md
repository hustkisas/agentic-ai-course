# Weekly review + plan next week

!!! success "Weekly ritual (every Sunday)"
    Scheduled: **2026-08-02** · Week 4 · Phase 2. 30 minutes. This is a big milestone week — take stock properly.

## The 5-question review

1. **What did I finish vs plan?** (Registry? Error handling + retries + guard? Multi-step debugging? The milestone agent?)
2. **What slipped, and why?**
3. **Reschedule the slippage** into next week's slots.
4. **Did I build the milestone?** The from-scratch multi-tool agent is *the* deliverable of the course so far — did you complete it?
5. **Are next week's blocks set?** Week 5 shifts to context/memory and RAG theory.

## Week 4 self-check — the big one

You've hit the core milestone when you can:

- [ ] Explain and use a tool registry (one source of truth per tool).
- [ ] Turn tool errors into observations so the agent recovers instead of crashing.
- [ ] Retry transient provider errors with backoff (and NOT retry permanent ones).
- [ ] Enforce an iteration guard.
- [ ] Debug agent behavior by reading the trace and localizing reasoning vs plumbing bugs.
- [ ] **Run your from-scratch multi-tool agent** on a real multi-step task, with a small eval set passing at `temperature=0`.

If the milestone agent isn't done, make it your only task next week before moving on — everything ahead builds on it.

## Celebrate (seriously)

If your agent runs: you have built, from scratch and with no framework, the thing most people only ever consume through a library. You now understand agents from the inside. That's the hardest conceptual hill in the whole plan, and it's behind you.

## Looking ahead: Weeks 5–6

Phase 2 continues into **context/memory** (why long conversations break, and compaction — studying Pi's `compaction.ts`) and **RAG** (embeddings, vector search, then building a retrieval tool for *your* agent). Your milestone agent is the thing you'll bolt RAG onto in Week 6.

## Coaching note

Bring your milestone agent to the next session. Ask to be quizzed on the robustness patterns (error-as-observation, retry classification, iteration guard) and to review your code against Pi's loop. Then get Weeks 5–6 written into the site.
