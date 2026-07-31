# Study Pi's orchestrator (supervisor.ts)

!!! success "Full lesson (code study)"
    Scheduled: **2026-08-26** · Week 8 · Phase 3. A real, production multi-agent coordinator — read the source.

## Why this matters

Last lesson was patterns in the abstract. This lesson is a *real* implementation: Pi's `orchestrator` package coordinates **multiple independent agent processes**. Reading it shows how the "supervisor/workers" pattern looks when it has to survive real life — process crashes, restarts, IPC, and cleanup. Study `packages/orchestrator/src/supervisor.ts` (and skim `serve.ts`, `handler.ts`, `rpc-process.ts`, `ipc/`).

## The big architectural choice: separate processes

Pi's orchestrator doesn't run sub-agents as function calls in one Python process (like your Week 8 build will). Each agent is a **separate OS process** — a `pi --mode rpc` child — coordinated over a Unix-socket IPC server. Why go to that trouble?

- **Isolation:** one agent crashing can't take down the others or the coordinator.
- **Independence:** each agent has its own memory, working directory, model, and session.
- **Concurrency:** true parallel work across processes.
- **Language/tool agnostic:** the coordinator talks to children over a protocol, not a shared runtime.

This is the same conceptual "supervisor/workers" pattern, hardened into a **distributed-systems** shape — territory your systems background will appreciate.

## What to look for in `supervisor.ts`

The `OrchestratorSupervisor` (exported as a singleton `supervisor`) is the manager. Trace these responsibilities:

- **A registry of live workers:** `liveInstances: Map<id, LiveInstance>`, where each entry bundles the child's record, its RPC process handle, subscribers, and cleanup handles. (Compare: your Week-4 tool *registry*, but for whole agents.)
- **Lifecycle:** `spawnInstance({cwd, label})` creates a record (status `starting`), spawns an `RpcProcessInstance`, wires up its event/exit/UI handlers, syncs its session, and marks it `online`. `stopInstance` and `shutdown` tear things down cleanly.
- **Delegation / message routing:** `handleRpc(id, command)` forwards a command to a specific child and returns its response. `openRpcStream(...)` sets up a live event stream from a child back to the caller.
- **Failure handling:** `handleUnexpectedRpcExit` marks a crashed child `error` and cleans up; `recoverAfterRestart()` reconciles state if the coordinator itself restarted (marks previously-online children stopped, disconnects stale registrations).
- **Status model:** `InstanceStatus = starting | online | stopping | stopped | error` — an explicit state machine per worker (again, systems thinking: model the states, handle every transition).

## The supporting cast (skim)

- **`rpc-process.ts`** (`RpcProcessInstance`): wraps one child process. `send(command)` assigns an id and returns a promise resolved when a matching `{type:"response", id}` arrives (a classic request/response-over-a-stream correlation by id — like matching `tool_call_id`s!). Other lines are dispatched as events or UI requests. `dispose()` SIGTERMs and awaits exit.
- **`ipc/` (server/client/protocol):** newline-delimited JSON over a Unix socket; a persistent bidirectional mode for streaming session events. `serve.ts` wires signals (SIGINT/SIGTERM) to a single-flight `shutdown()`.
- **`handler.ts`:** maps IPC requests (`spawn/list/status/stop/rpc/rpc_stream`) to supervisor calls — the clean boundary between "wire protocol" and "supervisor logic."

## The lessons to extract (transfer to your own build)

1. **A supervisor owns a registry of workers and their lifecycle** — spawn, track state, route messages, tear down.
2. **Every worker needs an explicit status state machine**, and you must handle *unexpected* transitions (crashes), not just the happy path.
3. **Delegation is request/response correlated by id** — the same pattern as tool calls, one level up.
4. **Cleanup and recovery are first-class**, not afterthoughts: signals, single-flight shutdown, restart reconciliation. This is what "production" adds over a demo.

Your Week 8 build will be the *in-process* version of this (sub-agents as function calls). Studying Pi shows you what the *distributed* version requires — so you understand both ends of the spectrum.

## Resources

- Pi: `packages/orchestrator/src/supervisor.ts` (primary), then `rpc-process.ts`, `handler.ts`, `serve.ts`, `ipc/server.ts`.
- Your Week 1 notes on Pi's `agent-loop.ts` — the orchestrator coordinates many things that each run a loop like that.

## Exercise (~45–60 min, reading)

1. Read `supervisor.ts`. Write down: how a worker is spawned, where its status lives, how a command is routed to a specific worker, and what happens when a worker crashes.
2. In `rpc-process.ts`, find how a `send()` promise gets matched to the child's response. Note the resemblance to tool-call id matching.
3. Contrast Pi's process-per-agent design with the in-process (function-call) multi-agent you'll build next lesson: list one advantage of each.

## Checkpoint — answer these

1. Why does Pi run each agent as a separate process instead of in-process? Name two benefits.
2. What does the supervisor's `liveInstances` map hold, and what's the per-worker status state machine?
3. How is "send a command to worker X and get its result" implemented over the stream?
4. What do crash-handling and restart-recovery add that a happy-path demo lacks?
