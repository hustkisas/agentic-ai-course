# PROJECT: raw HTTP script that streams an LLM reply

!!! success "Milestone project"
    Scheduled: **2026-07-12** · Week 1 · Phase 0 · **MILESTONE**. This is the checkpoint that proves Week 1 stuck.

## Goal

Write a Python script that calls an LLM API **directly over HTTP** — no `openai`/`anthropic` SDK, no framework — and **streams the reply to your terminal token-by-token**. This welds together everything from Week 1: async (optional here but encouraged), HTTP/JSON, SSE parsing, and the roles/tokens mental model.

Why no SDK? Because doing it once by hand means you'll forever understand what every SDK and agent is really doing. After this, SDKs become a convenience, not a mystery.

## Setup

- Pick a provider and get an API key. Any works; keep the key in an **environment variable**, never hard-coded:
  ```bash
  export LLM_API_KEY="your-key-here"
  ```
- Install one HTTP library: `pip install httpx` (supports both sync and streaming).
- Skim your provider's **streaming** API reference for three things (from Lesson 2's exercise): the **endpoint URL**, the **auth header**, and the **shape of a streamed `data:` chunk**.

## Specification (definition of done)

Your script must:

1. Read the API key from the environment (fail with a clear message if missing).
2. Send a `POST` with a JSON body containing a `system` message, one `user` message, and `stream: true`.
3. Read the streamed response and print each text delta **as it arrives** (visible token-by-token, not all at once at the end).
4. Stop cleanly at the stream's end marker.
5. Not crash on a missing key or an HTTP error status — print a readable message instead.

## Skeleton (fill in the provider-specific parts)

```python
import os, json, httpx

API_KEY = os.environ.get("LLM_API_KEY")
if not API_KEY:
    raise SystemExit("Set LLM_API_KEY in your environment first.")

URL = "https://api.YOUR-PROVIDER/v1/CHAT-ENDPOINT"     # from the docs
HEADERS = {
    "Authorization": f"Bearer {API_KEY}",              # some providers use a different header
    "Content-Type": "application/json",
}
body = {
    "model": "SOME-MODEL",                             # from the docs
    "stream": True,
    "messages": [
        {"role": "system", "content": "You are a concise assistant."},
        {"role": "user", "content": "Explain what an API is in two sentences."},
    ],
}

with httpx.stream("POST", URL, headers=HEADERS, json=body, timeout=60) as resp:
    resp.raise_for_status()
    for line in resp.iter_lines():
        if not line or not line.startswith("data:"):
            continue
        data = line[len("data:"):].strip()
        if data == "[DONE]":
            break
        chunk = json.loads(data)
        # TODO: pull the text delta out of `chunk` per your provider's shape
        delta = ...                                    # e.g. chunk["choices"][0]["delta"].get("content")
        if delta:
            print(delta, end="", flush=True)           # flush = appears immediately
print()   # final newline
```

The one part you must work out from the docs is the exact path to the text delta inside each `chunk` — that's the point of the exercise.

## Stretch goals (optional)

- Wrap it in `async def` with `httpx.AsyncClient` and `aiofiles`-free `async for` streaming (ties back to Lesson 1).
- Accept the prompt as a command-line argument.
- Print token/usage info from the final chunk if the provider sends it.

## Checkpoint / reflection

1. Trace one `data:` chunk from bytes on the wire to a character on your screen. What are the steps?
2. Where exactly does the "typewriter" streaming effect come from in your code?
3. If you removed `flush=True`, what would happen and why?
4. What did doing this without an SDK make concrete that you'd have missed otherwise?

When your terminal prints an answer token-by-token from a raw HTTP call you wrote yourself — **Week 1 is done.** Commit the script; you'll reuse this exact streaming logic when you build your agent loop in Weeks 3–4.
