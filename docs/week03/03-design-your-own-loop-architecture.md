# Design your own loop architecture

!!! success "Full lesson (design, on paper)"
    Scheduled: **2026-07-24** · Week 3 · Phase 2. This is a design lesson — you'll sketch the architecture before coding it (Week 3 day 4 → Week 4).

## Why design first

You're a systems person: you know that ten minutes of design saves hours of debugging. Before writing the agent, decide its **state**, its **control flow**, and its **failure behavior**. This lesson gives you the checklist; the output is a diagram/notes you'll implement next.

## The core state

At any moment your agent holds:

- **`messages`** — the running transcript (system + user + assistant + tool results). This grows every turn. It *is* the agent's memory.
- **`tools`** — the registry: a map from tool name → (schema, Python function).
- **`model` / config** — which model, and per-model param rules (temperature vs max_completion_tokens, etc.).
- **loop bookkeeping** — iteration count, a max-iteration cap, and a "done" flag.

Keep this in one small object/dataclass. Everything the loop does is read/update this state.

## The control flow (pseudocode)

```text
function run(user_input):
    messages = [system_prompt, user_input]
    for step in 1..MAX_STEPS:
        response = call_model(messages, tools)          # one LLM call
        messages.append(response.assistant_message)

        if response has no tool_calls:
            return response.content                      # DONE (the Week 1 stop condition)

        for each tool_call in response.tool_calls:
            result = dispatch(tool_call)                 # execute, with error handling
            messages.append(tool_result(tool_call.id, result))
        # loop again: model sees results, decides next step
    return "Stopped: hit MAX_STEPS"                      # guard against runaways
```

This is exactly the shape of Pi's `agent-loop.ts` you studied — reproduce it yourself and you own the concept.

## The decisions to make now (write down your answer for each)

1. **Stop condition.** Primary: no tool calls in the response. Do you also want an explicit `finish` tool the model can call to end deliberately? (Optional; "no tool call" is enough to start.)
2. **Iteration guard.** What's `MAX_STEPS`? (Start ~10.) What do you return if you hit it? A runaway loop burns tokens/money — this cap is non-negotiable.
3. **Tool dispatch.** How do you map a tool name to a function? (A dict `{"add": add_fn}` is perfect.) What if the model names a tool that doesn't exist?
4. **Argument validation.** Arguments arrive as a JSON string of *unvalidated* data. Where do you `json.loads` and check types before calling the real function?
5. **Error paths.** When a tool raises, do you crash, or **feed the error back to the model as the observation** so it can recover? (Almost always the latter — that's next lesson.)
6. **Multiple tool calls.** The model may request several at once. Do you run them in order (simple) or concurrently (faster, Week 1's `asyncio`)? Start sequential.
7. **Observability.** How will you *see* each step? At minimum, print `Thought/Action/Observation` per iteration. You'll thank yourself when debugging.

## A note on scope

Resist adding features. For now: one model, sequential tools, print-based tracing, a hard step cap. No persistence, no streaming, no framework. You're building the *engine*; trimmings come later. (Recall "Building Effective Agents": start simple.)

## Resources

- Re-read your **Week 1 Agent Loop lesson** and Pi's `packages/agent/src/agent-loop.ts` with this checklist in hand — map each checklist item to where it lives in that real code.

## Exercise (~30–45 min) — deliverable: a design doc

Write a one-page design (text or diagram) that answers all 7 decisions above for *your* agent. Include:
- the state object's fields,
- the loop pseudocode adapted to your choices,
- your `MAX_STEPS` value and hit-cap behavior,
- how a tool error becomes an observation.

You'll implement this exact design in the next two lessons.

## Checkpoint — answer these

1. What are the four parts of your agent's state?
2. What is the primary stop condition, and why do you *also* need an iteration guard?
3. When a tool raises an exception, what's the better behavior — crash or feed the error back? Why?
4. Why keep the first version deliberately minimal (one model, sequential, no persistence)?
