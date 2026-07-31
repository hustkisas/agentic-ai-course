# PROJECT: multi-agent delegation system

!!! success "Milestone project"
    Scheduled: **2026-08-30** · Week 8 · Phase 3 · **MILESTONE**. Completes Phase 3.

## Goal

Build a working **multi-agent system** where a supervisor coordinates ≥2 specialized sub-agents to complete a task that's cleaner to solve with division of labor than with one agent. This proves you can compose agents — the top of the single-agent → multi-agent progression.

## Choose a task that genuinely benefits from roles

Pick one where specialization clearly helps:

- **Research report:** Researcher (gathers facts via `search_docs`/lookup) → Writer (drafts) → Critic (checks claims against sources). Output: a short cited report.
- **Code helper:** Planner (breaks a request into steps) → Coder (writes code per step) → Reviewer (checks/refines). Output: a small working script.
- **Data Q&A:** Retriever (pulls relevant rows/passages) → Analyst (computes/answers) → Verifier (sanity-checks the number). Output: an answer with its evidence.

Reuse your Phase-2 agent and your Week-6 RAG tool wherever they fit.

## Requirements (definition of done)

1. **A reusable agent runner** (`run_agent(system_prompt, user_input, tools)`) — your Phase-2 loop, parameterized.
2. **≥2 specialist sub-agents**, each with a focused system prompt and its *own* tool set + context.
3. **A supervisor** whose tools are the sub-agents; it decomposes the task, delegates, and integrates results.
4. **Robustness carried over from Phase 2:** per-agent iteration guard, tool-errors-as-observations, retries. A sub-agent failing shouldn't crash the system.
5. **Context discipline (Week 5):** the supervisor passes each worker only what it needs; workers don't share raw context.
6. **Nested tracing:** you can see the supervisor's calls and each sub-agent's inner loop.
7. **It works end to end** on your task, and produces a better/cleaner result than a single overloaded agent would (compare!).
8. **A small eval set:** 3–5 inputs where the system behaves correctly at `temperature=0`.

## Structure

```
multiagent/
├── agent_core.py      # run_agent(...) — the reusable Phase-2 loop + registry + robustness
├── subagents.py       # ask_researcher / ask_writer / ask_critic (each wraps run_agent)
├── supervisor.py      # supervisor agent whose tools are the sub-agents
└── evals.py           # 3-5 test inputs + expected behavior
```

## The comparison that proves the point

Run your task **two ways**: (a) one single agent with all tools and a big prompt, and (b) your multi-agent system. Note where multi-agent wins (focus, context isolation, cleaner output) — and honestly, where it costs more (latency, tokens, complexity). This judgment — *is multi-agent worth it here?* — is the real skill (Week 2's "don't build an agent when a workflow will do," one level up).

## Stretch goals (optional)

- **Parallel workers:** if two sub-tasks are independent, run them concurrently with `asyncio.gather` (Week 1) and integrate — the fan-out pattern.
- **Evaluator loop:** have the critic send work back to the writer until it passes (generate→critique→revise).
- **Framework version:** rebuild it as a LangGraph supervisor with subgraphs, or with OpenAI SDK handoffs; compare to your hand-built version.
- **Process isolation (advanced):** run one sub-agent as a separate process and talk to it over stdin/stdout JSON — a tiny taste of what Pi's orchestrator does.

## Reflection (short written answers)

1. Trace one full run: supervisor's delegations, each sub-agent's inner loop, and the integrated result.
2. From your A/B comparison: did multi-agent actually beat the single agent here? On what axes, and at what cost?
3. Where did context isolation help (what noise stayed out of which agent)?
4. How does your in-process design relate to Pi's process-based orchestrator — what would you change to make yours distributed?

## Done means done — Phase 3 complete

When your supervisor coordinates specialists to finish a real task, recovers from a sub-agent error, keeps each agent's context clean, and you can justify *why* multi-agent was (or wasn't) worth it — **Phase 3 is complete.** You now command the full toolkit: single agents (Phase 2), frameworks + MCP (Week 7), and multi-agent coordination (Week 8). Next: Phase 4 makes it production-grade (persistence, streaming, providers — studying Pi's harness).
