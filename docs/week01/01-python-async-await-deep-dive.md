# Python async/await deep dive

!!! success "Full lesson"
    Scheduled: **2026-07-06** · Week 1 · Phase 0. Read, do the exercise, answer the checkpoint.

## Why this matters for agents

An agent spends almost all of its wall-clock time **waiting** — waiting for the LLM to stream back tokens, waiting for an HTTP tool call, waiting for a file read. Almost none of it is spent computing. That is the opposite of your C++ background, where the work was **CPU-bound** (crunching numbers on a core).

Agent code is **I/O-bound**, and the right tool for I/O-bound concurrency in Python is `asyncio`. If you try to run 5 tool calls "in parallel" with ordinary blocking code, they run one after another. With async, the 4 that are waiting on the network yield the CPU so the others can progress. This is exactly what the Pi agent loop does when it runs tool calls concurrently.

## The mental model

`asyncio` gives you **cooperative single-threaded concurrency**. There is **one thread** and **one event loop**. Functions voluntarily "pause" at `await` points, handing control back to the loop, which runs whatever else is ready. No threads, no locks, no data races — a very different world from `std::thread`.

- **Coroutine** — a function declared `async def`. Calling it does **not** run it; it returns a coroutine object (a "recipe"). It only runs when awaited or scheduled.
- **`await`** — "pause here until this finishes, and let the event loop do other work meanwhile." You can only `await` inside an `async def`.
- **Event loop** — the scheduler. `asyncio.run(main())` starts it, runs `main()` to completion, then stops it.
- **Task** — a coroutine scheduled to run concurrently (`asyncio.create_task(...)` or `asyncio.gather(...)`). This is how you get real overlap.

!!! note "C++ analogy (and where it breaks)"
    Closest thing you know: an event loop / `epoll` reactor, or `std::async` with a single executor thread. But async is **cooperative** — a coroutine only yields at `await`. If you call a **blocking** function (e.g. `time.sleep`, heavy CPU work) inside a coroutine, you freeze the *entire* loop, because nothing preempts it. That's the #1 beginner mistake.

## Sequential vs concurrent — the whole point

```python
import asyncio, time

async def fake_tool(name, seconds):
    print(f"start {name}")
    await asyncio.sleep(seconds)   # simulates waiting on I/O (NOT time.sleep!)
    print(f"done  {name}")
    return name

async def main():
    t0 = time.perf_counter()
    # Sequential: each await blocks the next -> ~3s total
    await fake_tool("A", 1)
    await fake_tool("B", 1)
    await fake_tool("C", 1)
    print("sequential:", round(time.perf_counter() - t0, 2), "s")

    t0 = time.perf_counter()
    # Concurrent: all three overlap while waiting -> ~1s total
    await asyncio.gather(fake_tool("A", 1), fake_tool("B", 1), fake_tool("C", 1))
    print("concurrent:", round(time.perf_counter() - t0, 2), "s")

asyncio.run(main())
```

The sequential block takes ~3s; the `gather` block takes ~1s. That 3x is the entire reason async exists for agents: while one tool waits on the network, the others make progress.

## The three rules that prevent 90% of bugs

1. **Calling a coroutine doesn't run it.** `fake_tool("A", 1)` by itself does nothing until you `await` it or wrap it in a task. Forgetting `await` gives a "coroutine was never awaited" warning.
2. **Never block the loop.** Use `await asyncio.sleep(x)`, not `time.sleep(x)`. Use an async HTTP client (`httpx.AsyncClient`, `aiohttp`), not blocking `requests`. For unavoidable CPU-heavy work, offload with `await asyncio.to_thread(fn, ...)`.
3. **To overlap, you need tasks or `gather`.** A row of `await`s runs sequentially. `asyncio.gather(...)` (or `asyncio.create_task`) is what makes things concurrent.

## Resources (do these)

- **Real Python — "Async IO in Python: A Complete Walkthrough"** — the best single tutorial; read it end to end.
- **Python docs — `asyncio`: "Coroutines and Tasks"** — reference for `gather`, `create_task`, `to_thread`.
- Optional: **FastAPI docs — "Concurrency and async/await"** — a famously clear plain-English explanation (you'll use FastAPI in Week 11 anyway).

## Exercise (~45–60 min)

Install an async HTTP client and prove the concurrency win yourself:

```bash
pip install httpx
```

Write `async_fetch.py` that fetches the same 3 URLs (e.g. `https://httpbin.org/delay/1`) two ways and prints both timings:

1. **Sequentially** — `await` each request in turn.
2. **Concurrently** — with `asyncio.gather`.

You should see ~3s vs ~1s. Then deliberately break it: replace the async client with blocking `requests.get` inside your coroutine and watch the concurrent version collapse back to ~3s — that's rule #2 in action.

## Checkpoint — answer these

1. Why is agent code I/O-bound, and why does that make `asyncio` (not threads) the natural fit?
2. What does calling an `async def` function actually return before you `await` it?
3. You have three `await client.get(...)` calls on separate lines. Do they run concurrently? How do you make them concurrent?
4. What happens to the whole program if you call `time.sleep(5)` inside a coroutine, and why?
