# Multi-step task test + debug

!!! success "Full lesson"
    Scheduled: **2026-07-31** · Week 4 · Phase 2. Push your agent on a real multi-step task and learn to debug agent behavior.

## Why this matters

A single tool call working doesn't prove your agent works. Real tasks need **several tool calls in sequence, where each depends on the last**. This is where subtle bugs surface — and where you learn the debugging skills that make you effective with agents. Debugging an agent is different from debugging normal code: the "logic" is partly in a stochastic model, so you debug by **reading the trace**.

## Design a genuine multi-step task

Give your agent tools that force a *chain*, where step N needs step N-1's output. Example tool set:

- `get_price(item)` → returns a number
- `apply_discount(price, percent)` → returns a number
- `add_tax(price, rate)` → returns a number

Then ask: *"What's the final cost of a 'widget' after a 20% discount and 8% tax?"* A correct run must: get_price → apply_discount → add_tax → answer. Three dependent steps.

## Debugging by reading the trace

Your per-step prints (Week 3) are your debugger. For each iteration you want to see: the **action** (tool + args), the **observation** (result), and ultimately the **final answer**. When something's wrong, the trace tells you *which* of these broke.

Make the trace richer while debugging:

```python
print(f"[step {step}] THOUGHT: {msg.content!r}")          # model's reasoning, if any
for call in msg.tool_calls or []:
    print(f"[step {step}] ACTION: {call.function.name}({call.function.arguments})")
# ... after dispatch:
print(f"[step {step}] OBSERVATION: {content!r}")
```

## The common agent bugs and how to spot them

| Symptom in the trace | Likely cause | Fix |
|---|---|---|
| Wrong/missing tool chosen | Weak tool `description` | Sharpen the description (it's prompt engineering) |
| Right tool, wrong args | Ambiguous param schema/description | Add descriptions & types to params; add an example |
| Agent answers without using tools | System prompt not directive enough | "You MUST use tools for X; do not compute yourself." |
| Answer ignores the tool result | Result formatting unclear | Return clearer strings; label units |
| Infinite-ish loop until MAX_STEPS | Model can't tell it's done, or a tool keeps erroring | Clearer results; consider a `finish` tool; check the error |
| Crash mid-run | Unhandled path in dispatch | Widen `safe_dispatch` (last lesson) |
| Non-deterministic pass/fail | Temperature too high | Set `temperature=0` (or remove it if the model rejects it) |

## The debugging workflow

1. **Reproduce** with a fixed prompt and `temperature=0` so runs are comparable.
2. **Read the trace top-down.** Find the *first* step that goes wrong — everything after it is downstream noise.
3. **Localize:** is the bad step a *reasoning* problem (wrong tool/args → fix the prompt/schema) or a *plumbing* problem (crash, wrong id, unparsed args → fix your code)?
4. **Change one thing, re-run.** Same discipline as any debugging.
5. **Add a regression note:** keep a list of tasks your agent should handle; re-run them after changes (this is the seed of Week 10 evals).

## Exercise (~60 min)

1. Implement the three chained tools (`get_price`, `apply_discount`, `add_tax`) and run the widget task.
2. Deliberately introduce two bugs and practice finding them via the trace:
   - Make one tool's `description` vague and watch the model mis-select or mis-call it.
   - Return a result without units and see if the final answer gets confused.
3. Fix both by reading the trace and adjusting *only* the thing the trace implicates.
4. Write down 3 tasks your agent now handles correctly — your first mini eval set.

## Checkpoint — answer these

1. Why does a multi-step *dependent* task expose bugs a single call can't?
2. What's your primary debugging tool for an agent, and how do you use it?
3. Given a bad step, how do you decide if it's a reasoning bug vs a plumbing bug — and does the fix differ?
4. Why set `temperature=0` while debugging?
