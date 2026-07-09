# PROJECT: from-scratch multi-tool agent

!!! success "Milestone project — the centerpiece of the course"
    Scheduled: **2026-08-02** · Week 4 · Phase 2 · **MILESTONE**. The single most valuable build in this plan.

## Goal

Assemble everything from Weeks 1–4 into **one working agent, built from scratch, no framework**: a loop + a tool registry + robust error handling + retries + an iteration guard, that completes a genuine multi-step task using at least **3 tools**. When this runs, you will *understand* agents at a level most people who only use frameworks never reach.

## Pick a task with real substance

Choose something that needs 3+ dependent tool calls and isn't pure arithmetic. Good options (pick one):

- **File/research assistant:** tools = `list_files(dir)`, `read_file(path)`, `word_count(text)`. Task: "Which file in ./notes is longest, and what's its first line?"
- **Small data task:** tools = `read_csv(path)`, `filter_rows(col, op, value)`, `average(col)`. Task: "Average price of items over $10 in data.csv."
- **Web-free utility agent:** tools = `get_time()`, `add_days(date, n)`, `weekday(date)`. Task: "What day of the week is 45 days from today?"

(Avoid tools needing internet unless you want to; local tools keep the focus on the agent, not API plumbing.)

## Requirements (definition of done)

1. **Tool registry** (Week 4 day 1): ≥3 tools, each with name, description, JSON-Schema params, and a Python fn — one source of truth.
2. **The loop** (Weeks 1/3): repeat model→tools→feedback until no tool call or `MAX_STEPS`.
3. **Robustness** (Week 4 day 2): tool errors become observations (agent recovers, doesn't crash); transient provider errors retried with backoff; iteration guard returns cleanly.
4. **Correct stop condition:** ends when the model answers with no tool call.
5. **Tracing:** prints Thought/Action/Observation per step.
6. **The task completes:** on your chosen task, the agent makes the right chain of calls and returns a correct final answer.
7. **A tiny eval set:** 3–5 test tasks it should handle, and it passes them at `temperature=0`.

## Structure (bring your pieces together)

```
agent.py
├── Tool, ToolRegistry           # week 4 day 1
├── tools (≥3) registered        # your chosen task
├── call_model_with_retry()      # week 4 day 2
├── safe_dispatch()              # week 4 day 2
├── run(user_input)              # weeks 1/3: the loop + guard + tracing
└── __main__: run your eval tasks
```

You already wrote every piece across the last two weeks — this project is assembly + polish, not new concepts.

## Stretch goals (optional, pick any)

- **Parallel tools:** make dispatch async and run independent calls with `asyncio.gather` (Week 1), preserving `tool_call_id` matching.
- **A `finish` tool:** let the model end deliberately, and compare to the "no tool call" stop condition.
- **Conversation mode:** loop on user input so it's a back-and-forth agent, not one-shot.
- **Swap the model:** run the same agent on a different model from your provider; note behavior/param differences.

## Reflection (write short answers — this cements it)

1. Trace one full run: list every step's action → observation → final answer.
2. Where exactly does your code implement each of: the loop, the stop condition, the iteration guard, error-as-observation, retry?
3. What did building this *without* a framework teach you that you'd have missed otherwise?
4. Compare your agent to Pi's `agent-loop.ts` — what does Pi do that yours doesn't, and why might that matter at scale?

## Done means done

When your from-scratch agent completes a real multi-step task, recovers from an injected tool error, and passes your small eval set at `temperature=0` — **Phase 2's core is complete.** You've built the thing. Everything after this (RAG, frameworks, multi-agent, eval, deploy) extends this foundation. Commit the code; you'll build on it in Week 6 (RAG) and Week 9 (harness hardening).
