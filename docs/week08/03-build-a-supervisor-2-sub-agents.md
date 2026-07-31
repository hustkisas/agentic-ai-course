# Build a supervisor + 2 sub-agents

!!! success "Full lesson (hands-on)"
    Scheduled: **2026-08-28** · Week 8 · Phase 3. Build the multi-agent pattern in-process, the simple way.

## Goal

Build a working **supervisor + 2 specialist sub-agents** in one Python process, using the key insight from the patterns lesson: **a sub-agent is just a tool the supervisor can call.** You already have an agent (Phase 2) and know tools cold — this is composition, not new machinery.

## The design

- **Sub-agent A — Researcher:** an agent whose job is to gather facts (tools: `search_docs` from Week 6, or a simple lookup). Focused prompt: "find and return relevant facts, with sources."
- **Sub-agent B — Writer:** an agent that turns facts into a clean answer. Focused prompt: "write a concise, well-structured answer from the provided facts; don't invent."
- **Supervisor:** an agent whose *tools are the two sub-agents*. It decides: research first, then write, then return.

```
user → supervisor
          ├─ tool: ask_researcher(question) → [Researcher agent runs] → facts
          └─ tool: ask_writer(facts, question) → [Writer agent runs] → answer
       supervisor returns the answer
```

## Implementation: sub-agents as tools

Wrap each sub-agent's `run()` (your Phase-2 agent function) as a tool function the supervisor can call:

```python
# You have run_agent(system_prompt, user_input, tools) from Phase 2 (your loop).

def ask_researcher(question: str) -> str:
    """Delegate a fact-finding question to the research specialist. Returns facts with sources."""
    return run_agent(
        system_prompt="You are a researcher. Find relevant facts and return them with sources. Do not write prose.",
        user_input=question,
        tools=[search_docs],            # the researcher gets the RAG tool
    )

def ask_writer(question: str, facts: str) -> str:
    """Delegate writing a final answer, given facts. Returns polished prose."""
    return run_agent(
        system_prompt="You are a writer. Using ONLY the provided facts, write a concise, cited answer.",
        user_input=f"Question: {question}\n\nFacts:\n{facts}",
        tools=[],                        # the writer needs no tools
    )

# The supervisor is just another agent whose TOOLS are the sub-agents:
supervisor_tools = [
    Tool("ask_researcher", "Get facts for a question from the research specialist.",
         {"type":"object","properties":{"question":{"type":"string"}},"required":["question"]},
         fn=ask_researcher),
    Tool("ask_writer", "Get a polished, cited final answer given a question and facts.",
         {"type":"object","properties":{"question":{"type":"string"},"facts":{"type":"string"}},
          "required":["question","facts"]},
         fn=ask_writer),
]

def supervise(question):
    return run_agent(
        system_prompt=("You are a supervisor. For a user question, first call ask_researcher to get "
                       "facts, then call ask_writer with those facts to produce the final answer. "
                       "Return the writer's answer."),
        user_input=question,
        tools=supervisor_tools,
    )
```

That's the whole pattern. **Nothing new** — the supervisor runs your Phase-2 loop; its "tools" happen to run more loops. When you read the trace, you'll see the supervisor call `ask_researcher`, get facts back as an observation, then call `ask_writer`.

## Make each agent's context its own

A benefit of this structure (Week 5 tie-in): each sub-agent has its **own messages/context**. The researcher's noisy tool output doesn't bloat the writer's context — the supervisor passes only the distilled `facts` string. This is context isolation for free, and a real reason to split agents.

## Trace it (your debugging skill from Week 4)

```
[supervisor step 1] ACTION: ask_researcher({"question": "..."})
    [researcher] ACTION: search_docs(...) → OBSERVATION: [source:...] ...
    [researcher] final: facts...
[supervisor step 1] OBSERVATION: facts...
[supervisor step 2] ACTION: ask_writer({"question":"...","facts":"..."})
    [writer] final: polished answer
[supervisor step 2] OBSERVATION: polished answer
[supervisor step 3] final answer
```

Nested traces = nested loops. Seeing this makes multi-agent concrete.

## Framework alternative (optional)

The same thing in LangGraph is a **supervisor graph with two subgraphs**; in the OpenAI Agents SDK it's **handoffs**. Build the plain-Python version first (you'll understand exactly what those abstract), then optionally redo it with LangGraph's supervisor helper to compare.

## Watch out for

- **Depth/cost:** each sub-agent is a full loop = many LLM calls. Keep it to 2 sub-agents and cap iterations (your Week-4 guard, per agent).
- **Context passing:** be explicit about what the supervisor hands each worker; they don't share memory.
- **Error propagation:** if the researcher returns junk, the writer writes junk — consider a quick check (seed of the evaluator pattern).

## Exercise (~60 min)

1. Refactor your Phase-2 agent so its loop is a reusable `run_agent(system_prompt, user_input, tools)` function.
2. Build `ask_researcher` and `ask_writer` as above (researcher gets `search_docs`; writer gets none).
3. Build the supervisor whose tools are those two, and run a question end to end.
4. Read the nested trace and confirm the flow: research → write → return.
5. (Stretch) Add a third sub-agent (a "critic" that checks the writer's answer against the facts) — the evaluator pattern.

## Checkpoint — answer these

1. Mechanically, what makes the supervisor able to "delegate"? What are its tools?
2. Why does giving each sub-agent its own context help (tie to Week 5)?
3. What does the nested trace show, and why is that reassuring about how un-magical multi-agent is?
4. Name a cost and a correctness risk of this design, and one mitigation each.
