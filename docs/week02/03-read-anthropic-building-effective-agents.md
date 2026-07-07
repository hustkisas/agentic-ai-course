# Read: "Building Effective Agents"

!!! success "Full lesson (guided reading)"
    Scheduled: **2026-07-17** · Week 2 · Phase 1. Read the article, then work through the notes and checkpoint below.

## Why this lesson is just "read one article"

Anthropic's **"Building Effective Agents"** is the single best conceptual map of this field. It's short, vendor-neutral in spirit, and written by people who ship agents. Reading it now — *after* you understand the loop and tool calling — will snap a lot of pieces into place and save you from a common trap: reaching for a heavy framework when a simple pattern would do.

**Read it here:** search "Anthropic Building Effective Agents" (anthropic.com/engineering/building-effective-agents).

Below is what to look for and how to lock it in.

## The one distinction that matters most: workflows vs agents

The article's central idea:

- **Workflows** — LLM calls orchestrated through **predefined code paths**. *You* decide the control flow; the LLM fills in steps. Predictable, cheap, testable.
- **Agents** — the **LLM decides its own control flow**, choosing tools and next steps dynamically (the Week 1 loop). Flexible, powerful, but less predictable and more expensive.

The key judgment, and the article's main advice:

> **Don't build an agent when a workflow will do.** Add autonomy only when the task genuinely needs the model to decide the path. More autonomy = more capability *and* more cost, latency, and failure surface.

For your systems-engineering instincts: this is the classic "don't reach for the most general, most dynamic solution when a fixed pipeline is simpler and more reliable."

## The building blocks to note as you read

- **The augmented LLM** — the base unit: an LLM + retrieval + tools + memory. Everything is built from this.
- **Prompt chaining** — decompose a task into a fixed sequence of LLM calls (each step feeds the next). A workflow.
- **Routing** — classify the input, then send it to a specialized path. A workflow.
- **Parallelization** — run subtasks concurrently (sectioning) or get multiple votes (voting), then aggregate. A workflow.
- **Orchestrator–workers** — a central LLM dynamically breaks work into subtasks and delegates. Edges into agent territory. (This is your Week 8 multi-agent build.)
- **Evaluator–optimizer** — one LLM generates, another critiques, loop until good enough. (Foreshadows Week 10 evals.)
- **Agents** — the autonomous loop, for open-ended tasks where you can't predict the steps in advance.

## Principles worth writing on a sticky note

1. **Start simple.** Use the simplest pattern that works; add complexity only when it demonstrably helps.
2. **Make the agent's decisions visible.** You must be able to see what it did and why (foreshadows Week 5 tracing/observability).
3. **Invest in the tool interface.** Clear tool names, descriptions, and schemas matter as much as the prompt (exactly last lesson's point).
4. **Guardrails and stopping conditions.** Cap iterations, validate tool inputs, handle failure — agents left unbounded misbehave (your Week 4 work).

## Resources

- **Primary:** Anthropic — "Building Effective Agents" (read in full; ~20–30 min).
- Optional companion: Anthropic's "Effective context engineering for AI agents" if you want to go deeper on context management (relevant to your Week 5).

## Exercise (~30–45 min)

1. **Read the article end to end.** No skimming.
2. **Write it back:** in your own words (≤10 bullets), list the patterns from "augmented LLM" up to "agents", and for each note *one sentence* on when you'd use it.
3. **Classify three tasks.** For each, decide: workflow or agent? And which specific pattern?
   - "Translate every incoming support email to English, then route to the right team."
   - "Given a vague bug report, investigate a codebase and propose a fix."
   - "Summarize a document, and if it's a contract, also extract the parties and dates."
4. **Reflect:** which pattern best describes the loop you'll build in Weeks 3–4?

## Checkpoint — answer these

1. In one sentence each: what's the difference between a workflow and an agent?
2. What's the article's main piece of advice about *when* to build an agent?
3. Name three of the building-block patterns and when you'd reach for each.
4. Of the three tasks in the exercise, which truly needs an agent (not a workflow), and why?
