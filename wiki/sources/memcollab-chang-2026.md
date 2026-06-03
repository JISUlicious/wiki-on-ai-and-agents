---
title: "MemCollab: Cross-Model Memory Collaboration via Contrastive Trajectory Distillation"
type: source
created: 2026-06-03
updated: 2026-06-03
sources:
  - memcollab-chang-2026.md
arxiv_id: "2603.23234"
authors:
  - Yurui Chang
  - Yiran Wu
  - Qingyun Wu
  - Lu Lin
first_author: Yurui Chang
year: 2026
tags:
  - 2026
status: complete
importance: medium
---

# MemCollab: Cross-Model Memory Collaboration via Contrastive Trajectory Distillation

Surfaced via the alphaXiv / NLP-newsletter digest.

[arXiv:2603.23234](https://arxiv.org/abs/2603.23234) — Chang, Wu, Wu, Lin (Penn State & AG2AI), 2026.

## Summary

Most agent-memory systems build memory for a single agent and reuse it with the *same* backbone model, tightly coupling stored knowledge to that model's reasoning style. In heterogeneous deployments — agents instantiated with backbones of different sizes, architectures, or specializations — the authors show that naive cross-model memory transfer can *degrade* performance, because stored memories entangle task-relevant knowledge with model-specific biases.

**MemCollab** is a collaborative memory framework that builds shared, cross-model memory by **contrasting reasoning trajectories** produced by different model-based agents on the same task. The contrastive process distills abstract reasoning constraints that capture shared task-level invariants while suppressing model-specific artifacts, and stores transferable strategies plus "error-forbidden" constraints rather than raw experiences. A **task-aware retrieval** mechanism conditions memory access on task category so only relevant constraints surface at inference time. On mathematical-reasoning and code-generation benchmarks, MemCollab improves both accuracy and inference-time efficiency across diverse agents, including cross-family settings.

## Key points

- Identifies the **cross-model transfer gap**: memory distilled on a source model can underperform target-model-native memory because of entangled model-specific bias.
- **Contrastive trajectory distillation**: compares preferred vs. unpreferred trajectories from different agents on the same task to extract shared invariants and cross-model failure patterns.
- Stores **high-level reasoning strategies + error-forbidden constraints**, not raw experience traces.
- **Task-aware retrieval** conditions memory access on task category to avoid retrieving irrelevant constraints.
- Evaluated on math reasoning (e.g., MATH500) and code generation; consistent accuracy *and* efficiency gains across heterogeneous backbones and model families.
- Positions collaboratively built cross-model memory as a **shared reasoning resource** for heterogeneous agent fleets.

## Concepts & entities

- [[memory-management]] — MemCollab is a cross-model agent-memory framework.
- [[multi-agent-memory|multi-agent]] — Memory is shared across agents with different backbones.
- [[latent-reasoning|reasoning]] — Distills transferable reasoning constraints and strategies.
- [[llm-agent]] — Target setting is heterogeneous LLM agents.
- [[experiential-learning]] — Learns from past problem-solving trajectories.

## References

- BoT [35] — distills reasoning templates from past experiences (related prior memory method).
- Benchmarks: mathematical reasoning (MATH500) and code-generation suites.
