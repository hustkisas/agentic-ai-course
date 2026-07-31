# MCP (Model Context Protocol) intro + connect a tool

!!! success "Full lesson"
    Scheduled: **2026-08-23** · Week 7 · Phase 3. The standard that lets tools be reused across every agent.

## Why this matters

So far, every tool you've built is bolted directly into *your* agent's code. That doesn't scale across teams and apps: everyone re-implements a "search the wiki" or "query the database" tool. **MCP (Model Context Protocol)** is the emerging open standard — think "USB-C for AI tools" — that lets a tool/data source be written **once as a server** and used by **any** MCP-compatible agent (Claude Desktop, IDEs, your own agent, etc.). It's rapidly becoming the interop layer of the ecosystem.

## The mental model: client ↔ server

MCP standardizes the connection between an **AI application (host/client)** and **external capabilities (servers)**:

- **MCP server** — a process exposing capabilities in a standard way:
    - **Tools** — functions the model can call (exactly your Week 2 concept, now over a protocol).
    - **Resources** — data the app can read (files, records) to put in context.
    - **Prompts** — reusable prompt templates the server offers.
- **MCP client** — lives inside the agent/app; discovers what a server offers and calls it.
- **Transport** — how they talk: **stdio** (local subprocess — most common for local servers) or **HTTP/SSE** (remote servers).

The payoff: **write a tool as an MCP server once → any MCP-aware agent can use it.** No re-implementation per app.

## How it connects to what you know

An MCP tool is your Week-2 tool with one extra hop:

```
your agent (MCP client) --discover--> MCP server: "here are my tools + schemas"
your agent --call add(2,3)--> MCP server --> result --> back into your loop as a tool result
```

The model still just emits a tool call; the difference is *where the tool lives* (a separate, reusable server) and *how it's discovered* (the protocol), not the fundamental mechanic. Frameworks integrate MCP so a server's tools appear as ordinary tools to your agent. (Pi itself can reach MCP tools — its agents load MCP tool schemas on demand.)

## Using an existing server (the common case)

You rarely start by writing a server — you *connect* to one. Many exist (filesystem, GitHub, databases, web search, etc.). With a framework's MCP adapter, connecting looks roughly like:

```python
# conceptual — exact API depends on your framework's MCP integration
# 1. point at a server (e.g. a local filesystem server over stdio)
# 2. the adapter discovers its tools
# 3. those tools drop into your agent's tool list, unchanged from the model's view
tools = load_mcp_tools(server="filesystem")     # discovered, not hand-written
agent = create_react_agent(model, tools=tools)  # your Week 7 agent, now with MCP tools
```

Your agent gains capabilities it didn't implement — that's the win.

## Writing a trivial server (to demystify it)

A minimal MCP server (using the official SDK) exposes a function as a tool:

```python
# conceptual shape with the MCP Python SDK (FastMCP)
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("demo")

@mcp.tool()
def add(a: float, b: float) -> float:
    """Add two numbers."""
    return a + b

# run over stdio; any MCP client can now discover and call `add`
```

That's the same `add` you've written five times — but now *any* MCP client can use it. The standardization is the point.

## Trade-offs / cautions

- **Extra moving part:** a server process + protocol vs a plain function. Worth it for *reuse/sharing*, overkill for a one-off local tool.
- **Security:** an MCP server can expose real capabilities (filesystem, network). Only connect to servers you trust; treat their outputs as untrusted data, not instructions (prompt-injection risk).
- **Fast-moving:** the spec and SDKs are evolving; pin versions.

## Resources

- **modelcontextprotocol.io** — the spec + concepts (server, client, tools/resources/prompts, transports).
- **MCP Python SDK (`mcp` / FastMCP)** — quickstart for writing a server.
- Your framework's **MCP adapter** docs (LangChain/LangGraph and the OpenAI Agents SDK both have MCP support).

## Exercise (~45–60 min)

1. Read the MCP "Introduction" + "Core concepts" pages. Write down the three server capabilities (tools/resources/prompts) and the two transports.
2. Connect your Week 7 agent to **one** existing MCP server (a local filesystem server is a good first target) via your framework's adapter, and have the agent use one of its tools.
3. (Stretch) Write the trivial `add` MCP server above and call it from an MCP client.
4. Note one thing MCP makes better than hard-coded tools, and one new risk it introduces.

## Checkpoint — answer these

1. What problem does MCP solve that per-agent hard-coded tools don't?
2. Name the three capabilities an MCP server can expose and the two common transports.
3. How is an MCP tool call different from — and the same as — your Week-2 tool calls?
4. What's the main security concern when connecting to an MCP server?
