# Start building a minimal agent from scratch

!!! success "Full lesson (start the build)"
    Scheduled: **2026-07-26** · Week 3 · Phase 2. Implement the design from the last lesson. No framework.

## Goal

Turn your design doc into a working — if bare — agent loop. By the end of this session you should have an agent that can take a user request, make **one or more** tool-using turns, and return a final answer. It won't be robust yet (that's Week 4); the goal is a running skeleton.

## Build it in layers (don't write it all at once)

**Layer 1 — one turn, no loop** (you basically have this from Week 2). Confirm: user message in, model either answers or requests a tool, you execute one tool, feed back, get an answer.

**Layer 2 — wrap it in the loop.** Replace "do it once" with "repeat until no tool call or MAX_STEPS." This is the whole leap from Week 2 to a real agent.

**Layer 3 — add tracing.** Print each iteration so you can see the agent think.

## A minimal reference implementation

```python
import os, json
from openai import OpenAI

client = OpenAI(base_url=os.environ["LLM_BASE_URL"], api_key=os.environ["LLM_API_KEY"])
MODEL = "gpt4o"
MAX_STEPS = 10

# --- tools: name -> (python fn, schema) ---
def add(a, b): return a + b
def multiply(a, b): return a * b

TOOL_FNS = {"add": add, "multiply": multiply}
TOOLS = [
    {"type": "function", "function": {
        "name": "add", "description": "Add two numbers.",
        "parameters": {"type": "object",
            "properties": {"a": {"type": "number"}, "b": {"type": "number"}},
            "required": ["a", "b"]}}},
    {"type": "function", "function": {
        "name": "multiply", "description": "Multiply two numbers.",
        "parameters": {"type": "object",
            "properties": {"a": {"type": "number"}, "b": {"type": "number"}},
            "required": ["a", "b"]}}},
]

def run(user_input):
    messages = [
        {"role": "system", "content": "You are a careful assistant. Use tools for arithmetic. Think before acting."},
        {"role": "user", "content": user_input},
    ]
    for step in range(1, MAX_STEPS + 1):
        resp = client.chat.completions.create(model=MODEL, messages=messages, tools=TOOLS)
        msg = resp.choices[0].message
        messages.append(msg)

        if not msg.tool_calls:
            print(f"[step {step}] final answer")
            return msg.content

        for call in msg.tool_calls:
            args = json.loads(call.function.arguments)
            print(f"[step {step}] action: {call.function.name}({args})")
            fn = TOOL_FNS[call.function.name]          # (Week 4: handle unknown tool)
            result = fn(**args)                          # (Week 4: handle exceptions)
            print(f"[step {step}] observation: {result}")
            messages.append({"role": "tool", "tool_call_id": call.id, "content": str(result)})

    return "Stopped: hit MAX_STEPS."

if __name__ == "__main__":
    print(run("What is (21 + 21) times 3?"))
```

Run it. You should see it call `add`, observe 42, call `multiply`, observe 126, then give a final answer — **a genuine multi-step agent you wrote.**

## What's deliberately missing (Week 4 fixes these)

- **Unknown tool** — `TOOL_FNS[name]` will `KeyError` if the model hallucinates a tool name.
- **Tool exceptions** — if a tool raises, the whole program crashes instead of letting the agent recover.
- **Bad arguments** — no validation of the parsed args.
- **Transient API errors** — no retry.

Leave these for now; just *notice* them. Naming your skeleton's weaknesses is the point.

## Exercise (~60 min)

1. Implement the reference above against your provider (fix model name / params as needed).
2. Confirm the two-step arithmetic works and you can *see* the trace.
3. Add a **third** tool of your own (e.g. `power(base, exp)`), and ask a three-step question.
4. Break it on purpose: ask for something with no matching tool and watch it either answer directly (fine) or misbehave (note it for Week 4).

## Checkpoint — answer these

1. What single change turned your Week 2 one-shot tool call into an agent?
2. Point to the exact line that implements the stop condition.
3. Name three ways this skeleton can currently crash or misbehave.
4. Why build in layers (one turn → loop → tracing) instead of all at once?
