# Weekly review + plan next week

!!! success "Weekly ritual (every Sunday)"
    Scheduled: **2026-07-26** · Week 3 · Phase 2. 30 minutes.

## The 5-question review

1. **What did I finish vs plan?** (ReAct lesson? Loop design doc? Started the from-scratch agent?)
2. **What slipped, and why?**
3. **Reschedule the slippage** into specific slots next week.
4. **Did I build, or just read?** You should have a *running* minimal agent by now, even a rough one.
5. **Are next week's blocks set?** Week 4 hardens the agent into your first milestone build — protect the time.

## Week 3 self-check

Ready for Week 4 when you can:

- [ ] Explain ReAct (reason → act → observe) and how it relates to tool-calling.
- [ ] Produce a one-page design doc for your agent (state, loop, stop condition, guard, error paths).
- [ ] **Run a minimal from-scratch agent** that completes a 2–3 step task and prints its trace.
- [ ] Name the ways your skeleton can currently crash (unknown tool, tool exception, bad args, API error).

If the agent doesn't run yet, that's your #1 task next week before adding robustness.

## Looking ahead: Week 4

You'll turn the skeleton into a real, robust agent: a proper tool registry, argument validation, error-handling (tool failures become observations), retries, and the iteration guard — then the **milestone project**: a from-scratch multi-tool agent. This is described as the single most valuable build in the whole plan.

## Coaching note

Bring your design doc and your running (or broken) agent to your next session. Ask to be quizzed on the loop's stop condition and the failure modes you found — Week 4 is all about closing those gaps.
