---
title: "Learning Agent-Compatible Context Management for Long-Horizon Tasks (AdaCoM)"
type: source
created: 2026-06-08
updated: 2026-06-08
sources:
  - learning-agent-compatible-context-management-yi-2026.md
arxiv_id: "2605.30785"
authors:
  - Lu Yi
  - Runlin Lei
  - et al.
first_author: Lu Yi
year: 2026
tags: [2026]
status: complete
importance: medium
---

# Learning Agent-Compatible Context Management for Long-Horizon Tasks (AdaCoM)

Surfaced via the NLP newsletter (Top AI Papers of the Week, 2026-05-31–06-07).

## Summary

LLM agents on long-horizon tasks (web search, deep research) accumulate context that triggers long-context degradation and reasoning failures. Prior remedies couple context control to the agent itself (agent-side control, fixed summarization), which requires retraining the agent and is impractical for closed-source agents. **AdaCoM (Adaptive Context Management)** instead trains a *separate external LLM* to manage the context of a **frozen** agent via flexible modification actions, optimized end-to-end with reinforcement learning (GRPO) against the task outcome reward.

Across diverse agents on web search and deep research benchmarks, AdaCoM improves performance by pruning stale content while preserving task constraints and progress. The learned policies reveal a **Fidelity–Reliability Trade-off**: stronger agents benefit from higher-fidelity context preservation, while weaker agents need more aggressive compression to stay in a reliable reasoning regime. Transfer experiments show the manager generalizes best across agents of *similar capability* (measured by vanilla ReAct performance), suggesting a path toward reusable context managers.

## Key points

- Trains an external context-manager LLM over a frozen agent — the agent is never retrained, enabling use with closed-source agents.
- Context management is cast as an MDP induced by the frozen agent + environment; the manager emits flexible modification actions (including leaving context unchanged when fidelity matters).
- Optimized end-to-end with GRPO using a trajectory-level outcome reward (LLM-judge binary/scored answer correctness).
- Identifies a Fidelity–Reliability Trade-off: high-performing agents want preserved raw trajectory context; low-performing agents want aggressive compression.
- Transfer is most effective across agents of comparable vanilla ReAct capability, implying reusability within a capability band.
- Authors are from Renmin University of China, Tongyi Lab (Alibaba), BUPT, and Université de Montréal.

## Concepts & entities

- [[context-assembly]] — managing what context is presented to the agent each step.
- [[memory-management]] — pruning stale content while preserving constraints/progress.
- [[reinforcement-learning]] — GRPO end-to-end optimization against outcome reward.
- [[retrieval-augmented-generation]] — web search / deep research agent setting.

## References

- [arXiv:2605.30785](https://arxiv.org/abs/2605.30785)
