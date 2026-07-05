# HTTP, JSON, REST & SSE streaming

!!! success "Full lesson"
    Scheduled: **2026-07-08** · Week 1 · Phase 0. This is the only "web" you need before building agents.

## Why this matters

Here is the demystifying truth of the whole field: **talking to an LLM is just an HTTP POST with a JSON body, and the reply streams back as Server-Sent Events.** Every SDK (`openai`, `anthropic`) is a convenience wrapper over that one HTTP call. If you understand this page, you understand what every agent is *actually* doing on the wire — and you'll never be confused by "which SDK" again.

## HTTP in 5 minutes

An HTTP exchange is a **request** and a **response**.

**Request** has:

- a **method**: `GET` (read), `POST` (send data / create), `PUT`/`PATCH` (update), `DELETE`.
- a **URL** (endpoint): `https://api.example.com/v1/messages`.
- **headers**: metadata key/values, e.g. `Authorization: Bearer sk-...`, `Content-Type: application/json`.
- an optional **body**: for `POST`, usually JSON.

**Response** has:

- a **status code**: `200` OK, `201` created, `400` bad request, `401` unauthorized, `429` rate-limited, `500`/`503` server error. (You'll handle `429` and `5xx` with retries later — Week 4.)
- **headers** and a **body** (usually JSON).

## JSON

JSON is the data format of the whole ecosystem — nested objects `{}` and arrays `[]` of strings, numbers, booleans, null. A chat request body looks like:

```json
{
  "model": "some-model",
  "messages": [{"role": "user", "content": "Hello"}],
  "stream": true
}
```

In Python you rarely hand-write it: `json.dumps(obj)` to serialize, `json.loads(text)` / `response.json()` to parse.

## REST (the pattern)

REST is a convention for HTTP APIs: **resources** live at **URLs**, and **methods** act on them (`GET /users/42` reads user 42, `POST /users` creates one). LLM chat endpoints aren't strictly RESTful, but they follow the same shape: `POST` a JSON body to an endpoint, get JSON back. That's all you need for now.

## SSE — how token streaming works

When you set `"stream": true`, the server doesn't wait to compute the whole answer and send it at once. It holds the connection open and pushes **Server-Sent Events (SSE)** — a simple text protocol where the response `Content-Type` is `text/event-stream` and the body is a series of lines like:

```text
data: {"delta": "Hel"}
data: {"delta": "lo"}
data: {"delta": " world"}
data: [DONE]
```

Each `data:` line is one chunk. Your code reads them as they arrive, parses the JSON, and prints the `delta`. That is **exactly** why you see an LLM reply appear token-by-token, and it's the same stream the Pi codebase turns into `text_delta` events (`packages/ai/src/utils/event-stream.ts`).

Key properties: it's **one long-lived HTTP response** (not websockets), **text-based**, and **one-directional** (server → client).

## See it on the wire with curl

```bash
# A non-streaming request (httpbin echoes your POST back as JSON):
curl -s -X POST https://httpbin.org/post \
  -H "Content-Type: application/json" \
  -d '{"hello": "world"}' | head -20
```

For a real streaming LLM call you'd add `-N` (no buffering) and your provider's endpoint + `Authorization` header + `"stream": true`. You'll do that in the next lesson's project.

## Resources

- **MDN — "An overview of HTTP"** and **"HTTP response status codes"** — the canonical, readable reference.
- **MDN — "Using server-sent events"** — the SSE protocol.
- Your LLM provider's **API reference** ("streaming" section) — bookmark it; it's the ground truth for the exact JSON shape.

## Exercise (~45 min)

1. Run the `curl` above and read the echoed JSON. Change the body and re-run.
2. Find your chosen provider's streaming docs and locate: the endpoint URL, the auth header, and the exact shape of a streamed `data:` chunk. Write those three things down — you need them for the Week 1 project.
3. In Python, do one `POST` with `httpx` (non-streaming) and `print(resp.status_code)` and `resp.json()`.

## Checkpoint — answer these

1. What are the four parts of an HTTP request?
2. An LLM API call — which HTTP method, and where does your prompt go?
3. In one sentence: what is SSE and why does it make replies appear token-by-token?
4. What status codes would tell you "slow down / retry later" vs "your request was malformed"?
