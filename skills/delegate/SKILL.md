---
name: delegate
description: Run a multi-step task with each step in its own fresh subagent context, so the main conversation never accumulates per-step working memory. Use when the user types "/delegate", says "run this as subagents", "delegate the steps", "keep the context lean", or hands over a long/multi-part task where the intermediate work would otherwise bloat the thread. Optional argument: the task, or an explicit list of steps.
---

# Delegate (subagents per step)

Keep the main context lean by doing the actual work in subagents. Each step
runs in its own fresh window; only the result comes back here. Nothing else
changes for the user — they just get the finished work.

## Steps

1. **Get the task.** Use the `/delegate` argument if given (a task or an
   explicit step list). Otherwise use the task the user most recently asked
   for in this conversation.

2. **Decompose** it into the smallest set of steps that each stand alone —
   each with a clear input and a checkable output. Mark which steps are
   independent (can run in parallel) vs. dependent (must run in order).

3. **Run each step as a subagent** via the Agent tool (`general-purpose`
   unless a more specific agent fits). Give the subagent only what that step
   needs and tell it to return a compact result — the finding/artifact/paths,
   not its scratch work. Launch independent steps in one message so they run
   concurrently; chain dependent steps, passing the prior result forward.

4. **Keep only results in the main thread.** Do not re-read or restate a
   subagent's full working context here — hold its returned conclusion and
   move on. That is the whole point: the bulk stays in the subagents' windows.

5. **Synthesize** the returned results into the final answer for the user, and
   verify the pieces fit (spot-check anything a subagent asserted but you have
   not seen).

## When to escalate

If the work is large enough to want deterministic fan-out → verify →
synthesize across many agents (not just per-step isolation), say so and use
the Workflow tool instead — but only when the user has opted into that scale.
For everyday multi-step work, plain per-step subagents (this skill) are the
lazy, correct default.
