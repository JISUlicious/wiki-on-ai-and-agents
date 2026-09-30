---
title: Planning
type: concept
created: 2026-09-29
updated: 2026-09-30
sources:
  - reliability-limits-llm-multi-agent-planning-ao-2026.md
  - drop-the-hierarchy-and-roles-dochkina-2026.md
  - code-as-agent-harness-ning-2026.md
  - structuredagent-andor-tree-planning-lobo-2026.md
  - beyond-the-leaderboard-albayaydh-2026.md
status: draft
importance: medium
---

# Planning

**Planning** in LLM agents means working out a sequence of steps toward a goal *before* or *while* acting. It covers decomposing a task into subgoals, choosing among alternative strategies, ordering actions, and revising the plan when execution goes wrong. It differs from [[reasoning]], which derives an answer, because a plan commits to *actions* over time and has to survive contact with an environment. The wiki treats planning in three ways: as part of the agent loop, as a job for the harness, and as a structure spread across several agents.

## In this wiki

- **Inside the agent loop.** [[react|ReAct]]'s `Thought:` steps allow [[chain-of-thought-prompting|CoT]]-style planning between actions. [[tree-of-thoughts]] makes the search over plans explicit. [[llm-agent]] lists the *planner + executor* architecture (BabyAGI, AutoGPT), and [[world-model]] covers planning against a learned simulator.
- **Structured plans.** [[structuredagent-andor-tree-planning-lobo-2026|StructuredAgent]] interleaves planning and execution with AND/OR trees. AND nodes are subgoals, OR nodes are fallback strategies, and the leaves are atomic web actions.
- **Planning as harness control.** [[code-as-agent-harness-ning-2026]] (§3.1) treats planning as *harness control* rather than a capability inside the model. It moves from linear decomposition to planning over repository and knowledge graphs, then search-based and orchestration-based planning, and it names the "PLAN.md as filesystem-backed control object" pattern. Plans become contracts in the [[plan-execute-verify-loop]]. [[ralph-loop|Ralph loops]] are the minimal version of that pattern.
- **Multi-agent planning.** [[reliability-limits-llm-multi-agent-planning-ao-2026|Ao et al.]] show that a delegated network of planning agents with no new exogenous signal is dominated by a centralized Bayes decision maker. Extra roles help only when they change the information structure. [[drop-the-hierarchy-and-roles-dochkina-2026|Dochkina]] finds that self-organizing agents with a fixed ordering and autonomous role choice beat designed hierarchies, but only above a capability threshold.
- **How often plans fail.** [[beyond-the-leaderboard-albayaydh-2026]] puts planning and constraint satisfaction among its six agent failure clusters, citing TravelPlanner at 0.6% for GPT-4.

## Related

- [[reasoning]] · [[large-reasoning-model]] · [[tree-of-thoughts]] · [[react]] · [[llm-agent]] · [[multi-agent]]
- [[plan-execute-verify-loop]] · [[code-as-harness]] · [[standardized-operating-procedures]] · [[world-model]]

## References

- [[reliability-limits-llm-multi-agent-planning-ao-2026]] · [[drop-the-hierarchy-and-roles-dochkina-2026]]
- [[code-as-agent-harness-ning-2026]] · [[structuredagent-andor-tree-planning-lobo-2026]] · [[beyond-the-leaderboard-albayaydh-2026]]

## Related 2026 sources

- [[gavel-graph-world-models-llm-task-planning-wang-2026]] — GAVEL: a hand-specified scene-graph world model verifies and repairs LLM plans; Qwen3-8B 41.2%→91.8% on BEHAVIOR-1K
