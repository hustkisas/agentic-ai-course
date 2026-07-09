# Token estimation & cost tracking

!!! success "Full lesson"
    Scheduled: **2026-08-05** · Week 5 · Phase 2. Read, study `estimate.ts`, answer the checkpoint.

## Why this matters

Two reasons, both practical:

1. **To trigger compaction** (last lesson), you need to know how full the context is — that requires estimating tokens *before* you send.
2. **Cost control.** Every call costs money per input + output token, and it re-sends the whole transcript each turn. On your course's provider, large runs even need to be cleared with the maintainer. Knowing your token/$ footprint is a core agent-engineering skill, and a natural fit for your quantitative background.

## Two ways to count tokens

- **Exact (after the fact):** the API response includes a `usage` object — `prompt_tokens`, `completion_tokens`, `total_tokens`. This is ground truth, but you only get it *after* the call.
- **Estimated (before the call):** you need a guess *before* sending, to decide whether to compact. Two options:
    - a real tokenizer library (e.g. `tiktoken`) — accurate but model-specific and a dependency;
    - a cheap heuristic — good enough for budget decisions.

## The heuristic (Pi's approach — study `estimate.ts`)

Pi's `packages/ai/src/utils/estimate.ts` uses a deliberately simple heuristic:

- **~4 characters per token** for text (`CHARS_PER_TOKEN = 4`).
- Images ≈ a fixed chunk (~4800 chars-equivalent).
- The clever part: `estimateContextTokens` **anchors on the last successful assistant `usage.totalTokens`** and only *estimates the messages added since then*, rather than re-estimating the whole transcript. That's more accurate than naive char-counting of everything, because the anchor is exact.

Minimal version you can use:

```python
def estimate_tokens(messages):
    chars = sum(len(str(m.get("content", ""))) for m in messages)
    return chars // 4          # ~4 chars/token

def context_fraction(messages, context_window):
    return estimate_tokens(messages) / context_window
```

Then trigger compaction when `context_fraction(...)` crosses, say, 0.7 (leaving reserve for the response).

## Tracking cost

Cost = tokens × price-per-token, and input/output are priced differently (output is usually pricier). Accumulate from the exact `usage` on each response:

```python
# prices are $ per 1M tokens (look up your model's numbers)
PRICE_IN, PRICE_OUT = 2.50, 10.00   # example only

total_cost = 0.0
def add_cost(usage):
    global total_cost
    total_cost += usage.prompt_tokens/1e6*PRICE_IN + usage.completion_tokens/1e6*PRICE_OUT
```

Pi does this in `calculateCost(model, usage)` — filling a per-call cost breakdown (input/output/cacheRead/cacheWrite), including provider-specific quirks like Anthropic's 1-hour cache-write premium.

!!! note "Prompt caching"
    Many providers cache the *stable prefix* of your prompt (system + early messages) so re-sending it costs less on subsequent turns. This is why cost isn't simply "tokens × price every time." You don't need to implement it, but know it exists — it's why long stable system prompts are cheaper than they look.

## Exercise (~45 min)

1. Read `estimate.ts`; write down its two tricks (4-chars/token, and anchoring on last usage).
2. Add `estimate_tokens` + `context_fraction` to your agent; print context fraction each turn.
3. Wire it to your `maybe_compact` from last lesson: compact when fraction > 0.7 instead of on message count.
4. Add cost accumulation from each response's `usage`; print running total after a multi-step run. (Use your model's real prices, or the example numbers.)

## Checkpoint — answer these

1. Why do you need *estimated* tokens before a call when you already get exact `usage` after it?
2. What is Pi's estimation heuristic, and what's the accuracy trick beyond "4 chars per token"?
3. Why are input and output tokens priced differently, and why does re-sending the transcript make cost grow per turn?
4. What is prompt caching and why does it make a long stable system prompt cheaper than a naive count suggests?
