# Prompting: system prompts, few-shot, structured output

!!! success "Full lesson"
    Scheduled: **2026-07-13** · Week 2 · Phase 1. Read, do the exercise, answer the checkpoint.

## Why this matters

Before an agent can *act*, it has to reliably *produce the right shape of output*. Prompting is how you steer a stochastic model toward doing what you want, consistently. This is the difference between a demo that works once and a system that works every time. As a systems person, think of prompting as **specifying the contract** with an unreliable component — the tighter the spec, the more reliable the behavior.

## The message roles (your control surface)

A chat request is a list of messages, each with a `role`. These are your levers:

- **`system`** — standing instructions, persona, rules, output format. Sent once, applies to the whole conversation. This is where most of your steering lives. *"You are a terse assistant. Always answer in one sentence."*
- **`user`** — the actual request/question.
- **`assistant`** — the model's prior replies (you resend them to give it memory).

A minimal steered call:

```python
messages = [
    {"role": "system", "content": "You are a precise assistant. Answer in exactly one sentence, no preamble."},
    {"role": "user", "content": "What is an API?"},
]
```

The system prompt is doing real work here: without it you get a chatty paragraph; with it you get one clean sentence.

## Three techniques, in order of power

### 1. Clear instructions (system prompt)
Be specific about role, task, constraints, and format. Vague prompts produce vague, variable output. "Summarize this" is weak; "Summarize this in 3 bullet points, each under 15 words, focusing on decisions made" is strong. Specificity reduces variance — your main enemy.

### 2. Few-shot examples
Show, don't just tell. Include 1–5 examples of input→output in the prompt, and the model imitates the pattern. This is the single most reliable way to control format and style.

```text
Classify the sentiment as POSITIVE, NEGATIVE, or NEUTRAL.

Input: "This is the best thing ever!"   Output: POSITIVE
Input: "It broke on day one."           Output: NEGATIVE
Input: "It arrived on Tuesday."         Output: NEUTRAL
Input: "I can't stop using it."         Output:
```

The model completes the last line following the demonstrated pattern. Few-shot turns a fuzzy request into a near-deterministic one.

### 3. Structured output (JSON)
For anything a program will consume, you need machine-readable output, not prose. Two levels:

- **Ask for it:** *"Respond with only a JSON object of the form `{\"sentiment\": \"...\", \"confidence\": 0.0}`. No other text."* Works, but the model may occasionally add stray text or malformed JSON.
- **Enforce it (schema / structured outputs):** most providers support a "JSON mode" or a response schema that *guarantees* valid JSON matching a shape. When available, always prefer this — it removes an entire class of parsing failures.

This matters enormously for agents: **tool calling (next lesson) is just structured output the runtime knows how to act on.**

## Temperature and prompting

Recall from Week 1: `temperature=0` ≈ deterministic, higher = more varied. Rule of thumb:

- **Extraction, classification, tool calling, structured output → `temperature=0`** (you want the same correct answer every time).
- **Brainstorming, creative writing → higher (0.7–1.0)**.

!!! warning "Provider quirk you'll hit"
    Some newer/reasoning models restrict sampling params — they reject `temperature`/`top_p`, or force `temperature=1`. If a call fails with a params error, remove `temperature` and steer with the prompt instead. (This is real: your course's provider has exactly these restrictions on some models.)

## The prompting mindset

> You are not "talking to" the model — you are **configuring** it. Every ambiguity in your prompt becomes variance in the output. Prompt engineering is variance reduction.

## Resources

- **Anthropic — "Prompt engineering overview"** and its prompt-engineering guide (clear, technique-by-technique).
- **OpenAI — "Prompt engineering" guide** and the "Structured Outputs" docs.
- **"Prompting guide" (promptingguide.ai)** — a solid vendor-neutral reference for zero-shot / few-shot / chain-of-thought.

## Exercise (~45–60 min)

Reuse your Week 1 script (the one that calls your OpenAI-compatible endpoint):

1. **System-prompt steering:** ask the same question three ways — no system prompt, a "one terse sentence" system prompt, and a "explain like I'm 5, use an analogy" system prompt. Note how the output changes.
2. **Few-shot:** build the sentiment classifier above. Feed 5 new sentences and check it returns only the label.
3. **Structured output:** get the model to return `{"sentiment": "...", "confidence": 0.0}` and `json.loads()` the result in Python. Then break it on purpose (ask for prose too) and watch the parse fail — that's *why* structured output matters.

## Checkpoint — answer these

1. What is the job of the `system` role, and why does it reduce output variance?
2. When would few-shot examples beat a longer instruction?
3. Why is structured (JSON) output the bridge from "chatbot" to "agent"?
4. You need the *same* extraction result every run — what temperature, and what do you do if the model rejects the `temperature` param?
