---
title: "Reaching Agreement Among Reasoning LLM Agents"
type: source
created: 2026-06-03
updated: 2026-06-03
sources:
  - reaching-agreement-among-reasoning-llm-agents-ruan-2025.md
arxiv_id: "2512.20184"
authors:
  - Chaoyi Ruan
  - Yiliang Wang
  - Ziji Shi
  - "(et al., NUS)"
first_author: Chaoyi Ruan
year: 2025
tags:
  - 2025
status: complete
importance: medium
---

# Reaching Agreement Among Reasoning LLM Agents

[arXiv:2512.20184](https://arxiv.org/abs/2512.20184) · Ruan, Wang, Shi, Li (NUS) · 2025. Surfaced via the alphaXiv / NLP-newsletter digest.

## Summary
Multi-agent systems improve answer quality by having several agents reason collectively rather than relying on a single inference pass. But existing orchestration uses static heuristics — fixed loop limits, barrier synchronization — which waste compute, suffer high latency from straggler agents, and risk committing to transient (unstable) agreements. The paper reframes reliable multi-agent reasoning as a problem analogous to classical distributed consensus and supplies a formal foundation for it.

It contributes a formal model of "multi-agent refinement" (correctness guarantees + semantics of agent reasoning), then introduces Aegean, a consensus protocol built for stochastic reasoning agents, implemented in Aegean-Serve — a consensus-aware serving engine doing incremental quorum detection across concurrent agent runs so execution can terminate early once enough agents converge. On four math-reasoning benchmarks it gives provable safety and liveness while cutting latency 1.2–20x versus SOTA baselines, with answer quality within 2.5%.

## Key points
- Diagnoses two failure modes of heuristic multi-agent workflows: straggler-dominated latency (slowest model 11–64x slower) and premature finalization of transient agreements.
- Maps non-deterministic LLM reasoning onto deterministic distributed-consensus guarantees (quorum intersection ensures committed answers are stable).
- Aegean protocol: agents refine positions over rounds after seeing peer rationales; guarantees safety, liveness, and "refinement monotonicity."
- Aegean-Serve embeds consensus coordination in the serving layer, using incremental quorum detection for early termination.
- Results: 1.2–20x latency reduction, quality within 2.5% of baselines, on four math reasoning benchmarks.
- Gains hold across both local GPU deployments and commercial API providers.

## Concepts & entities
- [[self-consistency]]
- [[inference-time-compute]]
- [[autogen]]
- [[metagpt]]
- [[reflection-mechanism]]

## References
- arXiv:2512.20184
