# Context window management + study Pi's compaction

!!! success "Full lesson"
    Scheduled: **2026-08-03** · Week 5 · Phase 2. Read, study the Pi code, answer the checkpoint.

## Why this matters

Your Week 4 agent grows its `messages` list every turn. On a long task it will eventually hit the **context window** — the hard token limit from Week 1. When it does, requests fail or the model "forgets" the start. Real agents must **manage context**: keep what matters, drop or summarize the rest. This is one of the defining engineering problems of agents, and Pi solves it in `packages/coding-agent/src/core/compaction/`.

## The problem, concretely

- Every turn appends: the assistant message, plus a tool result per tool call. Tool results (file contents, search output) are often **huge**.
- Input tokens are re-sent **every** call (the model is stateless) — so a long transcript costs more and more per turn, then eventually overflows the window entirely.
- You cannot just truncate blindly: cutting mid-conversation can drop the user's original goal or a critical earlier result.

## The strategies (in increasing sophistication)

1. **Truncate old turns** — drop the oldest messages once you approach the limit. Simple, but loses information and can cut a turn in half.
2. **Summarize (compaction)** — when the transcript gets large, use the model to **summarize the older portion** into a compact note, then continue with `[summary] + recent messages`. Keeps the gist, frees tokens. This is the standard approach.
3. **Externalize memory** — store facts/files outside the context and retrieve on demand (that's RAG, later this week). The context holds pointers, not everything.

Most real agents use **2 + 3 together**: compact the conversation, and retrieve external knowledge when needed.

## How compaction works (the Pi model)

Study `packages/coding-agent/src/core/compaction/compaction.ts`. The key ideas you'll see:

- **A trigger:** `shouldCompact(contextTokens, contextWindow, settings)` — compact when you're within a reserve of the window (don't wait for a hard failure). Settings hold `reserveTokens` and `keepRecentTokens`.
- **A cut point:** `findCutPoint(...)` chooses *where* to split "summarize this old part" vs "keep this recent part" — and it does so on **turn boundaries**, so it never splits a turn's assistant message from its tool results (which would make an invalid request). This care is the whole trick.
- **The summary:** `generateSummary(...)` calls the model with a dedicated `SUMMARIZATION_SYSTEM_PROMPT` to compress the old messages, then the transcript becomes `summary + kept-recent`.
- **Carrying files forward:** `CompactionDetails { readFiles, modifiedFiles }` is preserved across compactions so the agent doesn't "forget" which files it was working on.
- **Token accounting:** it estimates tokens (see next lesson) to decide when to trigger — it even anchors on the last assistant `usage.totalTokens` for accuracy.

The lesson: **compaction is summarization on turn boundaries, triggered by a token budget, preserving critical state.** Simple to say, fiddly to get right — which is why studying a production implementation is worth it.

## A minimal compaction you can add to your agent

```python
def maybe_compact(messages, model, keep_recent=6, trigger_len=30):
    if len(messages) < trigger_len:
        return messages
    system = messages[0]
    head, tail = messages[1:-keep_recent], messages[-keep_recent:]
    convo = "\n".join(f"{m['role']}: {str(m.get('content'))[:500]}" for m in head)
    summary = client.chat.completions.create(
        model=model,
        messages=[{"role": "system", "content": "Summarize this conversation so far, preserving goals, decisions, and any file names or key results. Be concise."},
                  {"role": "user", "content": convo}],
    ).choices[0].message.content
    return [system, {"role": "user", "content": f"[Summary of earlier conversation]\n{summary}"}, *tail]
```

(Real systems trigger on *tokens*, not message count, and cut on turn boundaries — this is the teaching version. Next lesson gives you token estimation to do it properly.)

## Resources

- Pi: `packages/coding-agent/src/core/compaction/compaction.ts` (and `utils.ts` for the summarization prompt).
- Anthropic — "Effective context engineering for AI agents" (concepts: what to keep, what to drop).

## Exercise (~45–60 min)

1. Read `compaction.ts` and write down, in your words: the trigger, the cut-point rule, and what's preserved across compactions.
2. Add the minimal `maybe_compact` to your Week 4 agent; call it at the top of each loop iteration.
3. Force a long conversation (many tool calls) and confirm the transcript gets compacted without breaking the agent (watch that you never orphan a tool call!).

## Checkpoint — answer these

1. Why does a long-running agent eventually break without context management — give both the failure and the cost angle?
2. What does compaction do, and why must the cut point respect *turn boundaries*?
3. What critical state does Pi preserve across compactions, and why?
4. How do compaction and RAG (externalized memory) complement each other?
