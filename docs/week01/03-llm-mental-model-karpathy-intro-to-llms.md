# LLM mental model + Karpathy "Intro to LLMs"

!!! success "Full lesson"
    Scheduled: **2026-07-10** · Week 1 · Phase 0. Watch the video, read this, answer the checkpoint.

## The one-sentence model

**A large language model is a function that, given a sequence of tokens, predicts a probability distribution over the next token — and you sample from it, append, and repeat.** Everything else (chat, tools, agents) is built on top of that single primitive.

That's worth sitting with. There is no database lookup, no reasoning engine, no "understanding" in the human sense underneath — there is a very large learned function predicting "what token comes next," trained on enormous text. The magic is that predicting the next token *well enough*, at scale, produces behavior that looks like reasoning.

## The concepts you must own

- **Token** — not a word, not a character: a sub-word chunk (~4 characters of English on average). "tokenization" is unhappy-path stuff: numbers, code, and non-English text tokenize inefficiently. Models see tokens, not text.
- **Context window** — the maximum number of tokens the model can attend to at once (input + output). This is a **hard limit** and the single most important constraint in agent engineering: transcripts grow, and when they approach the window you must prune or summarize (that's *compaction*, Week 5). Everything the model "knows" in a request must fit here — it has no memory between calls.
- **Statelessness** — each API call is independent. The model doesn't remember your last message; the *client* resends the whole conversation every time. "Memory" is an illusion your code maintains by resending history.
- **Temperature / sampling** — the model outputs probabilities; sampling picks the actual token. `temperature=0` ≈ greedy/deterministic (pick the most likely) — best for tool-calling and structured output. Higher temperature = more random/creative. As a systems person, note: **this is where non-determinism enters.** Same input can give different output. Your whole engineering posture (retries, evals, guardrails) exists to build reliability on top of this.
- **Roles** — chat models take a list of messages with roles: `system` (standing instructions / persona), `user`, `assistant` (the model's prior replies), and `tool`/`toolResult` (results you fed back). The agent loop is just growing this list.
- **Tokens = money & latency** — you pay per input and output token, and latency scales with output tokens. Context management isn't just correctness; it's cost control (Week 5 covers estimation — see `packages/ai/src/utils/estimate.ts`).

!!! note "The mindset shift from C++"
    Your old work: deterministic, provable — input X yields exact output Y. LLMs: **stochastic** — "correct" becomes "good enough, often enough." You're no longer proving correctness; you're engineering reliability around a fuzzy component. Internalizing this now saves months of frustration later.

## Primary resource (watch first)

- **Andrej Karpathy — "[1hr Talk] Intro to Large Language Models"** (YouTube). The best single on-ramp for a technical person. Watch the whole thing. Karpathy is a serious engineer explaining it to engineers — exactly your level.
- Optional deeper dive: **Karpathy — "Let's build GPT: from scratch, in code, spelled out"** (only if you want to see the internals; not required to build agents).

## Supporting resources

- **OpenAI Tokenizer** (web playground) or the `tiktoken` library — paste text and *see* it split into tokens. Try a paragraph of English, a block of code, and some numbers; notice the differences.
- Your provider's **"models" / pricing page** — note context-window sizes and per-token prices for a couple of models. This is real engineering data you'll use when choosing a model.

## Exercise (~45–60 min)

1. Watch Karpathy's Intro to LLMs (~1 hr). Take 5 bullet-point notes.
2. In the tokenizer playground, tokenize: (a) an English sentence, (b) a JSON snippet, (c) "1234567890". Record how many tokens each takes and write one sentence on why code/numbers are less efficient.
3. Look up the context window and input/output price for two models you might use. Which is cheaper, and by how much per 1M tokens?

## Checkpoint — answer these

1. In your own words, what does an LLM fundamentally *do*?
2. If the model is stateless, how does a chatbot "remember" earlier turns?
3. Why would you set `temperature=0` for tool-calling but maybe higher for brainstorming?
4. Why is the context window the most important constraint in agent engineering — what breaks when you exceed it?
