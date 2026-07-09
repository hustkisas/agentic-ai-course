# Add error handling, retries, iteration guard

!!! success "Full lesson"
    Scheduled: **2026-07-29** · Week 4 · Phase 2. This is what separates a demo from something that actually works.

## The mindset (from Week 1, now made concrete)

You're engineering **reliability around an unreliable component**. Three failure sources, three defenses:

1. The **tool** fails (bad input, missing file, network error) → **turn the error into an observation** so the agent can recover.
2. The **provider** fails transiently (429 rate limit, 5xx, timeout) → **retry with backoff**.
3. The **agent** loops forever (keeps calling tools, never finishes) → **iteration guard**.

## 1. Tool errors become observations (the key move)

The instinct is to `try/except` and crash. The *agentic* move is to **feed the error back to the model** as the tool result — then it can try a different approach, exactly like a human seeing an error message.

```python
def safe_dispatch(registry, call):
    try:
        result = registry.dispatch(call.function.name, call.function.arguments)
        return str(result), False
    except KeyError as e:
        return f"Error: {e}. Available tools: {list(registry._tools)}", True
    except json.JSONDecodeError as e:
        return f"Error: arguments were not valid JSON ({e}).", True
    except Exception as e:
        return f"Error running {call.function.name}: {e}", True
```

Then in the loop, append the error *as the tool result*:

```python
for call in msg.tool_calls:
    content, is_error = safe_dispatch(registry, call)
    messages.append({"role": "tool", "tool_call_id": call.id, "content": content})
```

Now if the model calls a nonexistent tool or passes bad args, it *sees* the error and self-corrects on the next turn instead of your program dying. This mirrors Pi exactly: tools throw, the loop catches, and the failure is reported back to the model as an error tool result (`isError: true`).

!!! warning "Every tool call must still get a result"
    Even on error, you must append a `tool` message for that `tool_call_id`. A tool call with no matching result makes the next provider request invalid. (Pi even inserts a synthetic "No result provided" for orphaned calls — same principle.)

## 2. Retry transient provider errors

Network blips, rate limits (429), and server errors (5xx) are *transient* — retry them with **exponential backoff**. Do NOT retry *your* bugs (400 bad request, auth errors) — those won't fix themselves.

```python
import time

def call_model_with_retry(client, **kwargs):
    delay = 1.0
    for attempt in range(5):
        try:
            return client.chat.completions.create(**kwargs)
        except Exception as e:
            transient = any(s in str(e).lower() for s in
                            ["429", "rate", "timeout", "500", "502", "503", "overloaded", "connection"])
            if not transient or attempt == 4:
                raise
            time.sleep(delay)
            delay *= 2      # 1s, 2s, 4s, 8s
```

This is a hand-rolled version of Pi's `isRetryableAssistantError` classifier (`packages/ai/src/utils/retry.ts`) — it *excludes* quota/billing errors (permanent) and retries overload/rate/network/5xx (transient). Same logic, smaller.

## 3. The iteration guard (never optional)

An agent with a bug — or a genuinely stuck task — can loop forever, burning tokens and money. Cap it:

```python
for step in range(1, MAX_STEPS + 1):
    ...
return "Stopped after MAX_STEPS without finishing."
```

Pick `MAX_STEPS` (~10–15 to start). Consider logging a warning when you hit it so silent truncation doesn't masquerade as success.

## Putting it together — the robust loop

```python
def run(user_input):
    messages = [SYSTEM, {"role": "user", "content": user_input}]
    for step in range(1, MAX_STEPS + 1):
        resp = call_model_with_retry(client, model=MODEL, messages=messages, tools=registry.schemas())
        msg = resp.choices[0].message
        messages.append(msg)
        if not msg.tool_calls:
            return msg.content
        for call in msg.tool_calls:
            content, is_error = safe_dispatch(registry, call)
            print(f"[step {step}] {call.function.name} -> {'ERROR ' if is_error else ''}{content[:80]}")
            messages.append({"role": "tool", "tool_call_id": call.id, "content": content})
    return "Stopped: hit MAX_STEPS."
```

## Exercise (~60 min)

1. Add `safe_dispatch` and confirm a tool that raises no longer crashes the agent — instead the agent *recovers*. (Make a tool that raises on some inputs, e.g. `divide` that rejects divide-by-zero, and ask a question that triggers it.)
2. Force an unknown-tool call (prompt the model to use a tool you didn't register) and watch it read the error and adapt.
3. Add `call_model_with_retry`. (You can simulate a transient error by temporarily raising in a wrapper to see the backoff.)
4. Confirm `MAX_STEPS` returns cleanly on a deliberately unsolvable task.

## Checkpoint — answer these

1. Why feed a tool error back to the model instead of crashing? What does the agent do with it?
2. Which errors should you retry, and which must you NOT retry? Why?
3. Why must every `tool_call_id` get a result even when the tool failed?
4. What real cost does a missing iteration guard create?
