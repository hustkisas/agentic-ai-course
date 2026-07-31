# Rebuild your Phase-2 agent in LangGraph

!!! success "Full lesson (hands-on port)"
    Scheduled: **2026-08-19** · Week 7 · Phase 3. Port your own agent to the framework — the best way to learn it.

## Goal

Take the agent you built by hand in Phase 2 — including its tools and, ideally, `search_docs` (RAG) — and rebuild it in LangGraph. You'll end with the *same behavior* on the *same tasks*, which makes the comparison concrete: **what did the framework add, and what did it hide?**

## Port it piece by piece

Map your hand-built parts to LangGraph equivalents. You already know every concept; this is translation.

| Your Phase-2 code | LangGraph equivalent |
|---|---|
| `messages` list (state) | `MessagesState` (or a custom `TypedDict` state) |
| `client.chat.completions.create(...)` | a `call_model` node using a LangChain chat model |
| `Tool` + `ToolRegistry` | plain Python functions passed as `tools=[...]` (docstring = description) |
| `safe_dispatch` (run tool, catch errors) | `ToolNode` (handles execution + tool messages) |
| `if not msg.tool_calls: return` | `tools_condition` conditional edge |
| the `for step in range(MAX_STEPS)` loop | the `tools → call_model` cycle in the graph |
| `search_docs` RAG tool | the same function, registered as a tool |

## Tools become functions with docstrings

LangGraph/LangChain read the **docstring and type hints** to build the schema you wrote by hand:

```python
def search_docs(query: str, k: int = 4) -> str:
    """Search the user's documents for passages relevant to a query.
    Use when the question might be answered by the user's documents.
    Returns passages with their source."""
    # ... your Week 6 retrieval code, unchanged ...
    return results
```

Notice: the JSON schema you wrote explicitly in Phase 2 is now *inferred* from the signature + docstring. Convenient — and a good reminder that the description still matters (it's the docstring now).

## Point the model at your provider

LangChain's OpenAI-compatible chat model accepts a custom base URL and key, so your existing endpoint works:

```python
from langchain_openai import ChatOpenAI
model = ChatOpenAI(model="gpt4o", base_url="<your OpenAI-compatible base_url>", api_key="<your username>")
```

(If a model rejects `temperature`, pass `temperature=None` or omit it — the Phase-2 provider quirk still applies.)

## Assemble and run

Use either the prebuilt (`create_react_agent(model, tools=[add, search_docs])`) or the explicit `StateGraph` from last lesson. Run your **Phase-2 eval tasks** through it and confirm identical outcomes.

## The comparison — the actual learning

After it works, write down honestly:

**What LangGraph added (less code for you):**
- Tool execution + error handling + the loop — gone from your code (it's in `ToolNode` + the graph).
- Streaming, checkpointing, human-in-the-loop available with small additions.
- Schema inference from function signatures.

**What LangGraph hid (harder to see/control):**
- The exact messages/prompt sent to the model.
- The precise retry/error behavior (is it what you'd have chosen?).
- More dependencies and version sensitivity.

**When you'd choose which:**
- Framework: complex graphs, need persistence/streaming/HITL, team standardization.
- Hand-rolled: simple loops, maximum control/transparency, minimal dependencies, learning.

There's no universally right answer — the point is you can now make the call *knowingly*.

## Exercise (~60 min)

1. Port your Phase-2 agent (tools + `search_docs`) to LangGraph. Get it running against your provider.
2. Run your Phase-2 eval set through both versions; confirm same answers.
3. Write the three-part comparison above (added / hid / when-to-use) from your own experience.
4. (Stretch) Add one framework-only feature — e.g. stream intermediate steps, or add a checkpointer and resume a run.

## Checkpoint — answer these

1. For three of your Phase-2 components, name the LangGraph equivalent.
2. Where did your explicit JSON tool schema go, and what replaced it?
3. From *your* port: one thing the framework clearly improved, one thing it obscured.
4. Give a concrete scenario where you'd keep the hand-rolled agent instead.
