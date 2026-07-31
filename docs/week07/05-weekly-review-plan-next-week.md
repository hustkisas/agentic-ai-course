# Weekly review + plan next week

!!! success "Weekly ritual (every Sunday)"
    Scheduled: **2026-08-23** · Week 7 · Phase 3. 30 minutes.

## The 5-question review

1. **What did I finish vs plan?** (LangGraph fundamentals? Ported your agent? Surveyed the Agents SDKs? Connected an MCP tool?)
2. **What slipped, and why?**
3. **Reschedule the slippage** into next week's slots.
4. **Did I build, or just read?** You should have your Phase-2 agent running in LangGraph, and at least one MCP tool connected.
5. **Are next week's blocks set?** Week 8 is multi-agent + the Phase-3 milestone.

## Week 7 self-check

Ready for Week 8 when you can:

- [ ] Explain LangGraph in terms of state, nodes, and (conditional) edges — and draw your agent loop as a graph.
- [ ] Run your Phase-2 agent (tools + RAG) rebuilt in LangGraph, and articulate what the framework added vs hid.
- [ ] Compare hand-rolled vs LangGraph vs a vendor Agents SDK, and pick sensibly per scenario.
- [ ] Explain MCP (client/server, tools/resources/prompts, transports) and connect one MCP tool to your agent.

## Looking ahead: Week 8

You move from one agent to **many**: multi-agent patterns (supervisor/worker, planner/executor, fan-out+verify), a study of Pi's real multi-process orchestrator (`supervisor.ts`), then building a **supervisor that delegates to specialized sub-agents** — the Phase-3 milestone. LangGraph subgraphs / SDK handoffs from this week are the tools you'll use.

## Coaching note

Bring your LangGraph port and your MCP experiment to the next session. Ask to be quizzed on the framework trade-offs and MCP's client/server model — Week 8's multi-agent work builds on graphs and handoffs. Then get Week 8 written into the site.
