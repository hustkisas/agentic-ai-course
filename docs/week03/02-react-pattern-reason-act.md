# ReAct pattern (reason + act)

!!! success "Full lesson"
    Scheduled: **2026-07-22** · Week 3 · Phase 2. Read, do the exercise, answer the checkpoint.

## Why this matters

Plain tool-calling lets a model act. **ReAct** (Reason + Act) makes it act *well* on multi-step problems by having it **think before each action** and **observe after**. It's the reasoning discipline behind most capable agents, and understanding it will make your from-scratch agent (this week's project) noticeably smarter.

## The idea

ReAct interleaves three things in a loop:

```
Thought:  reason about what to do next, given the goal and what you've observed
Action:   call a tool (with arguments)
Observation: the tool's result, fed back in
... repeat until you can give a Final Answer
```

The model narrates its reasoning ("I need the file's size, so I'll read it first"), takes one action, sees the result, then reasons again with that new information. It's the difference between guessing an answer in one shot and *working the problem*.

## ReAct vs plain tool-calling

They're closely related — modern tool-calling *is* ReAct's "Action/Observation" made native. The distinctions worth knowing:

- **Classic ReAct** was a *prompting* technique: you instructed the model to output `Thought:`/`Action:`/`Observation:` as **text**, and your code parsed that text to find the action. Fragile (parsing free-form text), but it worked before tool-calling APIs existed.
- **Modern tool-calling** makes the "Action" a **structured** object (Week 2) instead of parsed text — far more reliable. The "Thought" survives as the model's reasoning/message content, or as explicit reasoning in reasoning models.

So today you rarely hand-roll the `Thought:/Action:` text format. But the **mental model** — reason, act, observe, repeat — is exactly what your agent loop implements. ReAct is the *why* behind the loop's shape.

## Where "reasoning" lives now

- **Regular models:** encourage reasoning by prompt ("Think step by step about what tool to use and why, then call it") — the model's message content carries the thought.
- **Reasoning models** (o-series, GPT-5.x, Claude with thinking): they reason internally before answering; you often just let them, and they decide tool calls more reliably. (Note: some of these reject `temperature` and want `max_completion_tokens` — a real quirk from Week 2.)

Either way, the loop is the same: the model reasons, emits an action, you execute, you feed back the observation.

## Why this makes agents better

- **Grounding:** each action's *observation* corrects the model's assumptions before the next step (it doesn't barrel ahead on a wrong guess).
- **Decomposition:** hard tasks become a sequence of small, checkable steps.
- **Debuggability:** the thought/observation trace tells you *why* the agent did what it did (foreshadows Week 10 tracing).

## Resources

- **"ReAct: Synergizing Reasoning and Acting in Language Models"** (Yao et al.) — the original paper; read the abstract + figure 1 for the core idea.
- **promptingguide.ai — "ReAct" section** — a concise, practical explanation with examples.

## Exercise (~30–45 min)

Using your Week 2 one-tool setup:

1. Add a system prompt instructing the model to **first state a one-line reason** ("Thought: …") before deciding whether to call a tool, then act.
2. Ask a question that needs the tool. Observe the reasoning it emits alongside/before the tool call.
3. Now ask a two-step question (e.g. "What is 21+21, and is that more than 40?"). Watch it reason, call `add`, observe the result, then reason again to answer the comparison. That second reasoning-with-observation step **is** ReAct.

## Checkpoint — answer these

1. What are the three repeating elements of ReAct, and what does each contribute?
2. How does modern structured tool-calling relate to classic ReAct — what changed and what stayed?
3. Why does feeding the *observation* back before the next action make the agent more reliable?
4. Where does the "Thought" live when you use a reasoning model vs a regular one?
