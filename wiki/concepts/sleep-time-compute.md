---
title: Sleep-time Compute
type: concept
created: 2026-05-16
updated: 2026-05-16
sources:
  - sleep-time-compute-lin-2025.md
status: complete
importance: high
tags:
  - 2025
---

# Sleep-time Compute

Sleep-time compute is the practice of running LLM inference *between* user queries — during otherwise-idle time — to pre-process a persistent context into a re-represented form that makes the next test-time query cheaper. Where standard test-time scaling (o1-style sequential reasoning, parallel sampling) treats each query as stateless and pays full freight on every call, sleep-time compute exploits the fact that most real applications (document QA, coding agents, conversational assistants) carry context across turns. Lin et al. report ~5× test-time compute reduction at matched accuracy on Stateful GSM-Symbolic and Stateful AIME, +13%/+18% absolute accuracy gains when sleep-time compute is scaled up, and 2.5× lower average per-query cost on multi-query workloads.

## Core mechanism

- **Offline re-representation pass.** Between user queries, the model is prompted to generate inferences, intermediate computations, and anticipated decompositions over the persistent context, then writes them back so the next test-time query sees an already-processed substrate.
- **Stateful problem reframing.** The paper recasts reasoning benchmarks as `(context, query)` pairs and introduces Stateful GSM-Symbolic, Stateful AIME, and Multi-Query GSM-Symbolic to expose the gap pure test-time scaling cannot close.
- **Amortization across queries.** When multiple queries share one context, the sleep-time pass is paid once; Multi-Query GSM-Symbolic shows 2.5× lower average cost per query.
- **Predictability is the lever.** Gains scale with how anticipatable the test-time query is from the context — sleep-time compute is a bet on query distribution, not a free lunch.
- **Composable, not substitutive.** Sequential and parallel test-time scaling still apply on top of a sleep-time-processed context.

## What it advances

Sleep-time compute is an **offline consolidate** operation in the [[memory-management]] vocabulary:

- **Consolidate (offline)**: derive useful abstractions from raw context during idle time and persist them as new memory entries. Unlike [[reflection-mechanism]] in [[generative-agents]] (consolidation triggered by importance accumulation, inline with the agent loop), sleep-time consolidation is explicitly scheduled to the gap *between* user-facing inferences — a different temporal regime with different latency budgets.
- **Write (the consolidated output)**: the re-represented context is written into the same memory substrate the next test-time pass reads from; without that persistence layer, the sleep-time work is wasted.

It contributes nothing to store, retrieve, compact, or evict directly — it assumes those exist (e.g., via [[memgpt]] / [[letta]]) and adds a new *kind* of write to the consolidation path.

## Relation to other systems

- **vs. [[memgpt]]** — MemGPT manages *which* memory pages live in the context window (paging, eviction); sleep-time compute decides *what to compute and store* in those pages during idle time. Direct successor line — same authors, same Letta group.
- **vs. [[reflection-mechanism]] in [[generative-agents]]** — Both are LLM-driven consolidation passes that write abstract derived memories back into the stream. Reflection is triggered by importance thresholds inside the agent loop; sleep-time compute is triggered by the *absence* of a user query and is optimized for query-anticipation rather than character coherence.
- **vs. test-time scaling (o1, parallel sampling)** — Orthogonal axis. Test-time scaling assumes stateless queries and burns compute at the moment of the user-facing call; sleep-time compute moves work earlier into a cheaper latency regime. Both can stack.

## References

- Source: [[sleep-time-compute-lin-2025]]
- arXiv: <https://arxiv.org/abs/2504.13171>
- Code: <https://github.com/letta-ai/sleep-time-compute>
