# Tool / function calling concepts

!!! success "Full lesson"
    Scheduled: **2026-07-15** · Week 2 · Phase 1. This is *the* atom of agents — the most important mechanism after the loop itself.

## Why this matters

An LLM alone can only produce text. It can't read a file, hit an API, run code, or query a database. **Tool calling** is the mechanism that gives it hands. Every agent you will ever build is: a loop (Week 1) + tools (this lesson). Master this and the rest is composition.

## The mechanic (say it out loud until it's automatic)

> The model does not run tools. It **requests** a tool call as structured output. **Your code** executes it and feeds the result back. The model then continues with that result in context.

Four steps:

1. **You declare tools** to the model: each has a `name`, a `description` (when to use it), and a **parameter schema** (JSON Schema describing the arguments).
2. **The model decides** whether to answer directly or emit a **tool call** — a structured object like `{"name": "get_weather", "arguments": {"city": "Chicago"}}`. It does *not* execute anything.
3. **Your code executes** the real function `get_weather("Chicago")` and captures the result.
4. **You send the result back** as a `tool` / `tool_result` message. The model reads it and produces its next turn (a final answer, or another tool call).

Steps 2–4 repeating *is* the agent loop from Week 1. Tool calling is what the loop loops on.

## What a tool declaration looks like (OpenAI-compatible shape)

```python
tools = [
    {
        "type": "function",
        "function": {
            "name": "get_weather",
            "description": "Get the current weather for a city. Use when the user asks about weather.",
            "parameters": {
                "type": "object",
                "properties": {
                    "city": {"type": "string", "description": "City name, e.g. 'Chicago'"}
                },
                "required": ["city"],
            },
        },
    }
]
```

The `description` fields are **prompt engineering, not documentation** — the model reads them to decide *whether* and *how* to call the tool. Vague descriptions → wrong or missed calls. Treat them like the system prompts of last lesson: specific, unambiguous.

## What comes back, and what you do with it

When the model wants the tool, the response contains a `tool_calls` array instead of (or alongside) content. You:

```python
# 1. model returns tool_calls
tool_call = response.choices[0].message.tool_calls[0]
name = tool_call.function.name                     # "get_weather"
args = json.loads(tool_call.function.arguments)    # {"city": "Chicago"}

# 2. YOU execute the real function
result = get_weather(**args)                        # e.g. "Chicago: 12°C, cloudy"

# 3. append the assistant's tool_call message AND a tool result message, then call again
messages.append(response.choices[0].message)        # the assistant turn with the tool_call
messages.append({
    "role": "tool",
    "tool_call_id": tool_call.id,
    "content": result,
})
# 4. call the model again with the updated messages -> it answers using the result
```

That send-back-and-call-again is one turn of the loop. Do it until the model returns a normal message with no `tool_calls` — which, from Week 1, is exactly how the loop knows it's **done**.

## Key ideas that trip people up

- **Arguments are strings you must parse.** `tool_call.function.arguments` is a JSON *string* — `json.loads` it. And the model can hallucinate arguments or wrong types; validate before executing.
- **IDs must match.** The `tool_call_id` on your result must equal the `id` of the call the model made, or the provider rejects the follow-up.
- **The model can call multiple tools at once**, or the same tool several times. Handle a list, not just one.
- **Schemas differ slightly across vendors** (OpenAI vs Anthropic vs Google). Frameworks (Week 7) paper over this; doing it once by hand now means you'll understand what they're hiding. (Note: your course's gateway supports tool calling on its non-streaming chat path for OpenAI/Google/Anthropic models.)

## Resources

- **OpenAI — "Function calling" guide** (the request/response shapes above).
- **Anthropic — "Tool use" guide** (same concept, slightly different JSON; good for contrast).
- Re-read Week 1's `agent-loop.ts` walkthrough with this lesson in mind — `executeToolCalls` is steps 2–4 in production code.

## Exercise (~45–60 min)

Don't build the loop yet (that's the project) — just do **one** cycle by hand:

1. Write a trivial Python function, e.g. `def add(a, b): return a + b`.
2. Declare it as a tool (schema above as a template).
3. Send a user message like *"What is 21 + 21?"* with the tool declared.
4. Print the model's `tool_calls`. Confirm it asked for `add` with the right args (and did **not** compute the answer itself).
5. Execute `add`, send the result back as a `tool` message, call again, and print the final answer.

If your terminal prints "42" *after* a round-trip through your own `add` function — you understand tool calling.

## Checkpoint — answer these

1. Who executes a tool — the model or your code? What does the model actually produce?
2. What three things must a tool declaration contain, and why is the `description` really prompt engineering?
3. Walk through the 4 steps of one tool cycle for "What's 21+21?".
4. How does this connect to the Week 1 loop's stop condition?
