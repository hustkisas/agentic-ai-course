# LangGraph fundamentals

!!! success "Full lesson"
    Scheduled: **2026-08-17** · Week 7 · Phase 3. Now that you've built an agent by hand, meet the framework — and appreciate it.

## Why this matters (and why now)

You spent Phase 2 building the loop, tool dispatch, error handling, and retries yourself. That was the right order: **now you'll understand exactly what a framework does, because you did it by hand.** LangGraph is the most popular framework for building agents as **graphs**, and its model will feel natural to a systems/algorithms person — it's a state machine, not magic.

## The core idea: an agent is a graph

LangGraph models your application as a **directed graph**:

- **State** — a shared, typed object that flows through the graph (e.g. `{"messages": [...]}`). Every node reads it and returns updates to it.
- **Nodes** — functions that do work (call the model, run a tool) and return a partial state update.
- **Edges** — connections that decide what runs next. **Normal edges** are fixed ("after A, go to B"). **Conditional edges** branch based on the state ("if the model asked for a tool, go to the tool node; else finish").
- **Compile + invoke** — you build the graph once, then run it on an input; LangGraph drives the state through nodes/edges until it reaches the end.

Your Phase-2 loop *is* a graph with two nodes and a conditional edge:

```
        ┌─────────────┐
  START→│  call_model │←─────────┐
        └──────┬──────┘          │
               │ conditional     │
        tool_calls? ──yes──→ ┌───┴───┐
               │             │ tools │
               no            └───────┘
               ↓
              END
```

"Should I loop again?" (your `if not tool_calls: return`) becomes a **conditional edge**. The loop becomes the cycle `call_model → tools → call_model`. Same logic you wrote — expressed declaratively.

## What it looks like (prebuilt ReAct agent)

LangGraph ships a prebuilt that *is* your Phase-2 agent:

```python
from langgraph.prebuilt import create_react_agent

def add(a: float, b: float) -> float:
    """Add two numbers."""
    return a + b

agent = create_react_agent(model="openai:gpt4o", tools=[add])   # your loop, in one line
result = agent.invoke({"messages": [{"role": "user", "content": "What is 21 + 21?"}]})
print(result["messages"][-1].content)
```

That one call gives you the loop + tool dispatch + the stop condition you hand-wrote. But the *value* of LangGraph is the layer below — building custom graphs when the prebuilt isn't enough.

## Building a graph explicitly (the mental model that matters)

```python
from langgraph.graph import StateGraph, START, END, MessagesState
from langgraph.prebuilt import ToolNode, tools_condition

def call_model(state: MessagesState):
    response = model_with_tools.invoke(state["messages"])
    return {"messages": [response]}          # nodes return STATE UPDATES

graph = StateGraph(MessagesState)
graph.add_node("call_model", call_model)
graph.add_node("tools", ToolNode([add]))
graph.add_edge(START, "call_model")
graph.add_conditional_edges("call_model", tools_condition)  # tool_calls? -> "tools" else END
graph.add_edge("tools", "call_model")                       # the cycle
app = graph.compile()
```

Map each line to your hand-built agent: `call_model` = your model call; `ToolNode` = your `safe_dispatch`; `tools_condition` = your `if not msg.tool_calls`; the `tools → call_model` edge = your loop. **Nothing here is new to you** — that's the point of doing Phase 2 first.

## What the framework gives you (why it's worth adopting)

- **Persistence / checkpointing** — save state between steps, resume, time-travel (your Week 9 topic, for free).
- **Streaming** — stream tokens and intermediate steps out of the box.
- **Human-in-the-loop** — pause the graph for approval, then continue.
- **Branching / parallelism / subgraphs** — the building blocks for multi-agent (Week 8).
- **Observability integration** (LangSmith — Week 10).

## What it hides (be aware)

- The exact prompt/messages sent to the model (harder to see than your hand-rolled version).
- Control flow is declarative — great for complex graphs, but a simple loop can be *clearer* hand-written. (Recall "Building Effective Agents": use the simplest thing that works.)
- Version churn — LangGraph/LangChain APIs move fast; pin versions.

## Resources

- **LangGraph docs — "Quickstart" and "Low-level concepts" (state, nodes, edges)**.
- **`create_react_agent` reference** — the prebuilt agent.
- Keep your Phase-2 agent code open beside the docs and map concept-to-concept.

## Exercise (~45–60 min)

1. `pip install langgraph langchain`. Point the model at your OpenAI-compatible endpoint (base_url + your username as key; LangChain's OpenAI integration accepts these).
2. Run `create_react_agent` with your `add` tool. Confirm it matches your hand-built behavior.
3. Rebuild the same thing **explicitly** with `StateGraph` (the code above). Draw the graph on paper and label which node/edge corresponds to which part of your Phase-2 code.
4. Note one thing the framework made trivial, and one thing it made harder to see.

## Checkpoint — answer these

1. Define state, node, and edge in LangGraph — and what makes an edge *conditional*.
2. Draw your Phase-2 agent loop as a LangGraph graph. What plays the role of your `if not tool_calls`?
3. Name two things LangGraph gives you for free that you'd have had to build yourself.
4. Why was building the agent by hand first (Phase 2) the right order before learning this?
