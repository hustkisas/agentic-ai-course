# Weekly review + plan next week

!!! success "Weekly ritual (every Sunday)"
    Scheduled: **2026-08-30** · Week 8 · Phase 3. 30 minutes — end of Phase 3.

## The 5-question review

1. **What did I finish vs plan?** (Multi-agent patterns? Studied Pi's orchestrator? Built supervisor + 2 sub-agents? The milestone?)
2. **What slipped, and why?**
3. **Reschedule the slippage** into next week's slots.
4. **Did I build the milestone?** A working supervisor + specialists is the Phase-3 deliverable.
5. **Are next week's blocks set?** Week 9 is Phase 4 — harness engineering (studying Pi in depth).

## Week 8 self-check

You've completed Phase 3 when you can:

- [ ] Say when multi-agent genuinely beats a single agent — and when it's just overhead.
- [ ] Describe the core patterns (supervisor/workers, planner/executor, fan-out, evaluator/optimizer).
- [ ] Explain how Pi's orchestrator coordinates agents as separate processes (registry, status, IPC, recovery).
- [ ] **Run a supervisor + ≥2 sub-agents** on a real task, with robustness and context isolation, and justify the design via an A/B vs a single agent.

## Phase 3 retrospective

You now know both *how to build agents by hand* (Phase 2) and *how the ecosystem builds them* (frameworks, SDKs, MCP, multi-agent). That combination — foundations plus tooling — is exactly what makes someone effective and not just a framework user. Write 3 sentences: what clicked about frameworks, and whether multi-agent felt worth it on your task.

## Looking ahead: Phase 4 (Week 9)

You've built breadth; now go deep on **production harness engineering** by studying Pi itself: session persistence & branching trees, end-to-end streaming, and the multi-provider/auth abstraction — then harden your own agent with persistence + a real retry policy + a new tool. This is where "it works on my machine" becomes "it's a real system."

## Coaching note

Bring your multi-agent system to the next session. Ask to be quizzed on the patterns and the "is multi-agent worth it?" judgment. Then get Phase 4 (harness engineering) written into the site.
