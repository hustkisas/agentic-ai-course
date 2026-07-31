# Multi-agent patterns

!!! success "Full lesson"
    Scheduled: **2026-08-24** · Week 8 · Phase 3. When one agent isn't the right shape for the problem.

## Why this matters

You've built a capable single agent. Some problems, though, are better served by **several agents** — each with a focused role, a smaller tool set, and its own context — coordinating. This lesson maps the standard multi-agent patterns and, crucially, **when multi-agent actually helps vs when it's just complexity**.

## First: when is multi-agent worth it?

Multi-agent is *not* automatically better. It adds coordination overhead, cost (more LLM calls), and failure surface. Reach for it when:

- **The task decomposes into distinct specialties** (research vs write vs review) where a focused prompt + tool set per role beats one over-loaded agent.
- **Context would otherwise overflow** — each sub-agent keeps its own smaller context (ties to Week 5).
- **You want parallelism** — independent subtasks run concurrently.
- **You want separation of concerns / guardrails** — e.g. a "critic" agent that only reviews, never acts.

Recall "Building Effective Agents" (Week 2): **start simple.** One agent with good tools solves most problems. Add agents only when a single one is genuinely the wrong shape.

## The core patterns

### 1. Supervisor / workers (a.k.a. orchestrator–workers)
A **supervisor** agent receives the task, breaks it into subtasks, and **delegates** each to a specialized **worker** agent; it then integrates the results. The supervisor is the "manager"; workers are focused specialists.

```
          ┌────────────┐
  task →  │ supervisor │  ── delegates ──► worker A (researcher)
          │            │  ── delegates ──► worker B (writer)
          └─────┬──────┘  ── delegates ──► worker C (checker)
                └── integrates results → final answer
```
Most common, most flexible. This is your Week 8 build, and what Pi's orchestrator does at the process level (next lesson).

### 2. Planner / executor
One agent (**planner**) produces a step-by-step plan; another (**executor**) carries it out step by step. Separates "figure out what to do" from "do it" — useful when planning benefits from a different prompt/model than execution.

### 3. Fan-out / parallelization
Split a task into independent pieces, run agents on each **concurrently** (your Week 1 `asyncio`), then aggregate. Two flavors:
- **Sectioning** — different agents handle different *parts* (e.g. summarize 10 documents in parallel).
- **Voting** — several agents attempt the *same* task; you take a majority/best answer for reliability.

### 4. Evaluator / optimizer (generate → critique → revise)
A **generator** produces output; an **evaluator** critiques it against criteria; loop until good enough. Great for quality-sensitive output (foreshadows Week 10 evals, and "adversarial verification").

### 5. Handoffs / network
Agents pass control to each other peer-to-peer (the OpenAI SDK "handoff" from Week 7). Flexible but harder to reason about — a supervisor is usually clearer.

## How agents actually communicate

Under the hood, "delegation" is mundane: **a sub-agent is a tool the supervisor can call.** The supervisor's tool list includes things like `ask_researcher(question)` → which runs the researcher agent and returns its answer as the tool result. So multi-agent = your Week-4 tool mechanic, where some "tools" are *themselves agents*. That's the key insight that makes this un-magical.

- **In LangGraph:** sub-agents are **subgraphs**/nodes; the supervisor routes via conditional edges.
- **In SDKs:** **handoffs** (OpenAI) or agents-as-tools.
- **In Pi's orchestrator:** each agent is a separate **process**, coordinated over IPC (next lesson).

## Pitfalls

- **Over-decomposition:** three agents where one would do — pure overhead.
- **Lost context:** workers don't see what the supervisor knows unless you pass it; be explicit about what each sub-agent needs.
- **Cost/latency blowup:** every agent is more LLM calls; parallelize independent work, and cap depth.
- **Error propagation:** a worker's wrong answer poisons the supervisor's synthesis — hence evaluator/critic patterns.

## Resources

- Anthropic — "Building Effective Agents" (the orchestrator–workers and evaluator–optimizer sections).
- LangGraph — "Multi-agent" docs (supervisor pattern, subgraphs).
- OpenAI Agents SDK — "Handoffs".

## Exercise (~30–45 min, mostly design)

1. For each of these, pick the best pattern and justify: (a) summarize 20 files then write one report; (b) write code, then have it reviewed and fixed; (c) answer a question three ways and take the consensus.
2. Sketch a supervisor/workers design for a task you care about: name the workers, their specialties, their tools, and what the supervisor passes to each.
3. Write one sentence on how "a sub-agent is just a tool" changes how hard this seems.

## Checkpoint — answer these

1. Give three concrete signals that a task genuinely warrants multi-agent (not just one agent).
2. Describe supervisor/workers, planner/executor, and evaluator/optimizer in one line each.
3. Mechanically, how does one agent "delegate" to another — what is a sub-agent, really?
4. Name two pitfalls of multi-agent and how you'd mitigate each.
