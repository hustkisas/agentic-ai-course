# Build loop: tool registry + execution

!!! success "Full lesson"
    Scheduled: **2026-07-27** · Week 4 · Phase 2. Turn your skeleton's ad-hoc tool handling into a clean registry.

## Why this matters

Your Week 3 skeleton hard-coded tools with two parallel structures: a `TOOL_FNS` dict and a `TOOLS` schema list, kept in sync by hand. That doesn't scale and drifts out of sync. This lesson builds a **tool registry** — one source of truth per tool — and clean dispatch. This is exactly what Pi's tool layer does (`packages/coding-agent/src/core/tools/`).

## One source of truth per tool

Bundle everything a tool needs — name, description, schema, and the function — into one object, and register them together:

```python
import json
from dataclasses import dataclass
from typing import Callable

@dataclass
class Tool:
    name: str
    description: str
    parameters: dict          # JSON Schema
    fn: Callable

class ToolRegistry:
    def __init__(self):
        self._tools: dict[str, Tool] = {}

    def register(self, tool: Tool):
        self._tools[tool.name] = tool

    def schemas(self) -> list[dict]:
        # what you send to the model
        return [{"type": "function", "function": {
            "name": t.name, "description": t.description, "parameters": t.parameters
        }} for t in self._tools.values()]

    def dispatch(self, name: str, arguments: str):
        if name not in self._tools:
            raise KeyError(f"Unknown tool: {name}")
        args = json.loads(arguments)          # may raise json.JSONDecodeError
        return self._tools[name].fn(**args)   # may raise (bad args / tool error)
```

Now adding a tool is one `register(...)` call, and the model's schema list is generated from the same objects that hold the functions — no drift.

```python
registry = ToolRegistry()
registry.register(Tool(
    name="add", description="Add two numbers.",
    parameters={"type": "object",
        "properties": {"a": {"type": "number"}, "b": {"type": "number"}},
        "required": ["a", "b"]},
    fn=lambda a, b: a + b,
))
```

## The loop, using the registry

```python
resp = client.chat.completions.create(model=MODEL, messages=messages, tools=registry.schemas())
msg = resp.choices[0].message
messages.append(msg)
if not msg.tool_calls:
    return msg.content
for call in msg.tool_calls:
    result = registry.dispatch(call.function.name, call.function.arguments)  # error handling next lesson
    messages.append({"role": "tool", "tool_call_id": call.id, "content": str(result)})
```

Cleaner, and every tool call goes through one `dispatch` chokepoint — the perfect place to add validation, logging, and error handling (next lesson).

## Sequential vs parallel execution

The model can request several tool calls in one turn. Two ways to run them:

- **Sequential** (default, start here): run them one after another in a `for` loop. Simple, deterministic, easy to debug.
- **Parallel**: run independent calls concurrently with `asyncio.gather` (your Week 1 skill). Faster when tools are I/O-bound (network, files) and independent — which is common.

Important subtlety (this is exactly what Pi's loop handles): **results must be appended in an order the provider accepts**, and every `tool_call_id` must get a matching result. If you go parallel, collect results and append them so each `tool_call_id` is answered. Start sequential; add parallelism only once it works.

!!! note "Pi reference"
    Pi's `agent-loop.ts` has `executeToolCallsSequential` and `executeToolCallsParallel`, and a rule that if *any* tool in the batch is marked sequential, the whole batch runs sequentially. You're building a small version of that idea.

## Exercise (~45–60 min)

1. Refactor your Week 3 agent to use a `Tool` dataclass + `ToolRegistry`. Delete the parallel `TOOL_FNS`/`TOOLS` structures.
2. Confirm the multi-step arithmetic still works.
3. Register a third tool via one `register(...)` call to feel how easy adding tools now is.
4. (Stretch) Make dispatch `async` and run a batch of independent tool calls with `asyncio.gather`; verify each `tool_call_id` still gets its result.

## Checkpoint — answer these

1. Why is a single registry object per tool better than parallel dict + schema list?
2. What does `schemas()` produce and who consumes it?
3. Why is `dispatch` a good chokepoint for the robustness features coming next lesson?
4. When is parallel tool execution worth it, and what must stay true about `tool_call_id`s?
