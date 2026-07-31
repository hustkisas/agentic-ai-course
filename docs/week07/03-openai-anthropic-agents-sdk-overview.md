# OpenAI / Anthropic Agents SDK overview

!!! success "Full lesson (survey + compare)"
    Scheduled: **2026-08-21** · Week 7 · Phase 3. A shorter, comparative lesson — the goal is a map, not mastery.

## Why this matters

LangGraph isn't the only game. The model vendors ship their own agent SDKs, and they make different trade-offs. You don't need to master each — you need a **map**: what exists, how they differ, and how to choose. Because you built an agent by hand, you can evaluate any SDK by one question: *how does it handle the loop, tools, and stop condition you already understand?*

## The landscape

- **OpenAI Agents SDK** (`openai-agents`) — a lightweight Python framework from OpenAI. Core primitives: **Agents** (an LLM + instructions + tools), **handoffs** (one agent delegates to another — built-in multi-agent, relevant to Week 8), **guardrails** (validate inputs/outputs), and **sessions** (conversation memory). It runs the loop for you and is deliberately minimal.
- **Anthropic** — provides the **Claude Agent SDK** / the Messages API with tool use, plus the reference patterns from "Building Effective Agents" (Week 2). Historically Anthropic leaned on "here are the primitives and patterns, compose them" rather than a heavy framework, though SDK tooling has grown.
- **Others you'll hear about:** LlamaIndex (RAG-first agents), CrewAI / AutoGen (multi-agent-first), Pydantic AI (type-safe, validation-first). Same concepts, different ergonomics.

## The pattern behind all of them

Every one of these is a wrapper around what you built:

```
loop { model call with tools → if tool_calls: run them, feed back → else: done }
```

They differ in **ergonomics and extras**, not in the fundamental mechanism:

| Dimension | What to look at |
|---|---|
| **Tool definition** | Decorator? Function + docstring? Explicit schema? |
| **The loop** | Hidden ("run this agent") vs explicit graph (LangGraph) |
| **Multi-agent** | Handoffs (OpenAI), graphs/subgraphs (LangGraph), crews (CrewAI) |
| **Guardrails** | Built-in input/output validation? |
| **Memory/sessions** | Automatic conversation persistence? |
| **Provider lock-in** | Tied to one vendor, or model-agnostic? |
| **Observability** | Native tracing? |

## OpenAI Agents SDK — a taste

```python
from agents import Agent, Runner, function_tool

@function_tool
def add(a: float, b: float) -> float:
    """Add two numbers."""
    return a + b

agent = Agent(name="Calc", instructions="Use tools for arithmetic.", tools=[add])
result = Runner.run_sync(agent, "What is 21 + 21?")
print(result.final_output)
```

Compare to your hand-built agent and to LangGraph's `create_react_agent`: three spellings of the *same loop*. The `@function_tool` decorator infers the schema (like LangGraph's docstring approach); `Runner.run` hides the loop (like the prebuilt). Its distinctive feature is **handoffs** for multi-agent — you'll revisit that next week.

## How to choose (a practical heuristic)

- **Simplest task / max control / learning →** hand-rolled (you have this).
- **Complex control flow, persistence, streaming, HITL →** LangGraph.
- **Staying in one vendor's ecosystem, want handoffs/guardrails with minimal setup →** that vendor's Agents SDK.
- **RAG-centric →** LlamaIndex. **Multi-agent-centric →** CrewAI/AutoGen (or LangGraph subgraphs).
- **Provider-agnostic gateway (like your course's) →** favor model-agnostic frameworks (LangGraph, or plain SDKs pointed at the OpenAI-compatible endpoint).

Don't over-invest in one framework. The **concepts** (loop, tools, context, RAG, evals) transfer; frameworks are swappable ergonomics on top.

## Resources

- **OpenAI Agents SDK docs** — read the "Agents", "Tools", and "Handoffs" pages.
- **Anthropic docs** — tool use + the Agent SDK / "Building Effective Agents" (re-skim).
- Skim one *other* framework's quickstart (CrewAI or LlamaIndex) just to see the range.

## Exercise (~30–45 min)

1. Read the OpenAI Agents SDK "Agents" + "Handoffs" pages. (Optionally run the `add` example against your provider's OpenAI-compatible endpoint if the SDK accepts a custom base URL.)
2. Fill in the comparison table above for: your hand-rolled agent, LangGraph, and the OpenAI Agents SDK.
3. For three scenarios (a simple internal tool; a complex multi-step research assistant; a vendor-locked enterprise app), pick which approach you'd use and justify it in one line each.

## Checkpoint — answer these

1. What single loop underlies all these SDKs, and how do they mainly differ?
2. What is a "handoff" in the OpenAI Agents SDK, and which Week-8 topic does it relate to?
3. Given a provider-agnostic gateway, which frameworks fit best and why?
4. Why is it unwise to over-invest in one framework this early?
