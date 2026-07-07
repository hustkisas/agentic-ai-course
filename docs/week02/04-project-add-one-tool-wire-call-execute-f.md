# PROJECT: add one tool + wire call → execute → feedback

!!! success "Milestone project"
    Scheduled: **2026-07-19** · Week 2 · Phase 1 · **MILESTONE**. Proves you can do a full tool cycle end to end — the core mechanic of every agent.

## Goal

Extend your Week 1 script so the model can use **one real tool**, and you wire the complete cycle by hand: declare the tool → let the model request it → **your code executes it** → feed the result back → get a final answer that uses the result.

This is deliberately *not* a loop yet (that's Weeks 3–4). You're proving you can execute **one** turn of tool use correctly. Get this right and the loop is just "do this repeatedly."

## Setup

- Reuse your Week 1 client (OpenAI-compatible endpoint; your username where the key goes; on-network if required).
- Use a **non-streaming** call for this — simpler to inspect `tool_calls`.
- Pick a model that supports tool calling (a general chat model; avoid the restricted reasoning-only ones for now).

## Specification (definition of done)

1. Define one real Python function. Good choices: `add(a, b)`, `get_current_time()`, or `word_count(text)`. Keep it trivial — the point is the *plumbing*, not the tool.
2. Declare it to the model with a name, a clear description, and a JSON-Schema `parameters` block.
3. Send a user message that *requires* the tool (e.g. "What is 4211 + 899?").
4. Detect the model's `tool_calls`, parse the arguments (`json.loads`), and **execute your real function**.
5. Append the assistant's tool-call message and a `tool` result message (with the matching `tool_call_id`), then call the model again.
6. Print the model's final answer, which should incorporate your function's result.
7. Handle the "no tool call" case gracefully (if the model answers directly, just print that).

## Skeleton (fill in the blanks)

```python
import os, json
from openai import OpenAI

client = OpenAI(
    base_url=os.environ["LLM_BASE_URL"],   # your OpenAI-compatible endpoint
    api_key=os.environ["LLM_API_KEY"],     # your username where the key goes
)
MODEL = "gpt4o"   # a tool-calling-capable model

def add(a: float, b: float) -> float:
    return a + b

TOOLS = [{
    "type": "function",
    "function": {
        "name": "add",
        "description": "Add two numbers. Use for any arithmetic addition.",
        "parameters": {
            "type": "object",
            "properties": {
                "a": {"type": "number"},
                "b": {"type": "number"},
            },
            "required": ["a", "b"],
        },
    },
}]

messages = [{"role": "user", "content": "What is 4211 + 899?"}]

# --- turn 1: does the model ask for the tool? ---
r1 = client.chat.completions.create(model=MODEL, messages=messages, tools=TOOLS)
msg = r1.choices[0].message

if not msg.tool_calls:
    print("No tool needed:", msg.content)
else:
    messages.append(msg)                      # the assistant turn (contains the tool_call)
    for call in msg.tool_calls:
        args = json.loads(call.function.arguments)
        result = add(**args)                  # <-- YOUR code runs the real function
        messages.append({
            "role": "tool",
            "tool_call_id": call.id,
            "content": str(result),
        })
    # --- turn 2: model answers using the tool result ---
    r2 = client.chat.completions.create(model=MODEL, messages=messages, tools=TOOLS)
    print("Final answer:", r2.choices[0].message.content)
```

## Stretch goals (optional)

- Add a **second** tool (e.g. `multiply`) and ask a question that needs both — notice the model may call them in sequence, which is the loop wanting to exist.
- Print the raw `tool_calls` object so you *see* the structured request the model emitted.
- Deliberately ask something the tool can't do and confirm the model answers directly (no tool call).

## Checkpoint / reflection

1. At the moment the model returned `tool_calls`, had it computed the answer? Who did?
2. Why must `tool_call_id` on your result match the call's `id`?
3. What would you have to add to turn this single cycle into an open-ended agent? (You're describing Week 3.)
4. Where did prompting (last lesson) show up — what made the model choose the tool?

When your script prints the correct sum *after* routing through your own `add()` — Phase 1 is done. You now have both halves of an agent (loop + tools) in your head and hands. Weeks 3–4 combine them.
