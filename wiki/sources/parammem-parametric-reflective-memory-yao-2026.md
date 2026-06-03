---
title: "ParamMem: Augmenting Language Agents with Parametric Reflective Memory"
type: source
created: 2026-06-03
updated: 2026-06-03
sources:
  - parammem-parametric-reflective-memory-yao-2026.md
arxiv_id: "2602.23320"
authors:
  - Tianjun Yao
  - Yongqiang Chen
  - Yujia Zheng
  - "(et al.)"
first_author: Tianjun Yao
year: 2026
tags:
  - 2026
status: complete
importance: medium
---

# ParamMem: Augmenting Language Agents with Parametric Reflective Memory

[arXiv:2602.23320](https://arxiv.org/abs/2602.23320) · Yao, Chen, Zheng, Li, Shen, Zhang · 2026. Surfaced via the alphaXiv / NLP-newsletter digest.

## Summary
Self-reflection lets language agents iteratively refine their solutions, but in practice it tends to produce repetitive reflections that cap reasoning gains. The authors find a strong positive correlation between reflective *diversity* and task success (r up to ~0.97 on multi-hop QA), motivating mechanisms that generate diverse reflection signals rather than rephrasing the same critique.

ParamMem is a parametric memory module that encodes cross-sample reflection patterns into model parameters, so diverse reflections can be generated via temperature-controlled sampling. Built on top of it, ParamAgent is a reflection-based agent framework combining parametric memory with episodic and cross-sample memory. Across code generation, math reasoning, and multi-hop QA it beats SOTA baselines, and analysis shows it is sample-efficient, enables weak-to-strong transfer across model scales, and supports self-improvement without relying on a stronger external model.

## Key points
- Empirical premise: reflective diversity correlates strongly with task success (e.g., HotpotQA r=0.97, 2WikiMQA r=0.95, MATH r=0.91), so diversity is the lever to pull.
- ParamMem: a parametric memory module that internalizes cross-sample reflection patterns into model parameters; temperature sampling yields diverse reflections.
- Complements (rather than replaces) episodic memory and cross-sample memory in the agent loop.
- ParamAgent framework integrates all three memory types for reflection-based agents.
- Consistent gains over SOTA on code generation, mathematical reasoning, and multi-hop QA.
- Properties: sample-efficient (practical for resource-constrained settings), weak-to-strong transfer across scales, and self-improvement without a stronger teacher model.

## Concepts & entities
- [[reflexion]]
- [[self-reflection]]
- [[reflection-mechanism]]
- [[memory-management]]
- [[episodic-memory-llm]]
- [[self-improving-agent]]
- [[llm-agent]]

## References
- arXiv:2602.23320
