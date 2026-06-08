---
title: "Scaling Behavior of Single LLM-Driven Multi-Agent Systems"
type: source
created: 2026-06-08
updated: 2026-06-08
sources:
  - scaling-behavior-single-llm-multi-agent-systems-li-2026.md
arxiv_id: "2606.00655"
authors:
  - Jialing Li
  - Zhouhong Gu
  - et al.
first_author: Jialing Li
year: 2026
tags: [2026]
status: complete
importance: medium
---

# Scaling Behavior of Single LLM-Driven Multi-Agent Systems

Surfaced via the NLP newsletter (Top AI Papers of the Week, 2026-05-31–06-07). Newsletter listed it as "Do More Agents Help?"

## Summary

LLM-based multi-agent systems (MAS) promise to tackle complex tasks through collaborative intelligence, but their scaling behavior and collective dynamics are under-explored. This paper systematically studies how a **homogeneous** MAS — every agent driven by the *same* single LLM — performs as the number of agents increases, deliberately isolating collaboration from model/knowledge heterogeneity. It proposes the **Sequential Iterative Multi-Agent System (SIMAS)** framework, where each agent responds based on a progressively accumulated conversation history.

The central finding: performance does **not** scale monotonically with agent count. Instead it follows a pattern of diminishing returns governed by a trade-off between collaborative synergy and coordination overhead, eventually degrading. The optimal agent count is task-dependent, and collective intelligence is an emergent property contingent on the base model's capability (typically larger models past a threshold can effectively power MAS). The degradation stems from coordination overhead (reasoning fragmentation, compounding hallucinations, cascading errors) rather than merely long-context failure, and this tendency generalizes across settings. The takeaway reframes the goal from "more agents" to the *optimal* number — design the interaction, don't just stack agents.

## Key points

- Studies homogeneous MAS (all agents = one single LLM) to isolate the scaling effect of collaboration.
- Introduces SIMAS, a sequential iterative framework over accumulated conversation history.
- Performance is non-monotonic in agent count: diminishing returns, then degradation.
- Driven by a synergy-vs-coordination-overhead trade-off; optimal agent count is task-dependent.
- Degradation attributed to coordination overhead (fragmentation, compounding hallucinations, cascading errors), not just long-context failure.
- Practical message: design the interaction structure rather than naively adding agents; authors at Fudan University.

## Concepts & entities

- [[multi-agent]] — core subject: scaling of LLM multi-agent systems.
- [[single-agent-vs-multi-agent-tran-2026]] — related analysis of when multi-agent helps.
- [[diversity-collapse-in-multi-agent-llm-systems-chen-2026]] — related failure mode in homogeneous multi-agent systems.
- [[chain-of-thought]] — single-agent reasoning baseline contrasted with MAS.

## References

- [arXiv:2606.00655](https://arxiv.org/abs/2606.00655)
