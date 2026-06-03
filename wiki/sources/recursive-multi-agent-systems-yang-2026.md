---
title: "Recursive Multi-Agent Systems"
type: source
created: 2026-06-03
updated: 2026-06-03
sources:
  - recursive-multi-agent-systems-yang-2026.pdf
arxiv_id: "2604.25917"
authors:
  - Xiyuan Yang
  - Jiaru Zou
  - Rui Pan
  - Ruizhong Qiu
  - Pan Lu
  - Shizhe Diao
first_author: Xiyuan Yang
year: 2026
tags: [2026]
status: complete
importance: medium
---

# Recursive Multi-Agent Systems

Surfaced via the alphaXiv / NLP-newsletter digest.

## Summary

Recursive ("looped") language models have emerged as a new scaling axis by iteratively refining the same model computation over latent states to deepen reasoning. This work extends that principle from a single model to multi-agent systems, asking whether agent collaboration itself can be scaled through recursion. **RecursiveMAS** casts the entire multi-agent system as a unified latent-space recursive computation, connecting heterogeneous agents in a collaboration loop via a lightweight **RecursiveLink** module that enables in-distribution latent-thought generation and cross-agent latent state transfer.

## Key points

- RecursiveLink enables latent-space communication between heterogeneous agents instead of text-based message passing.
- An **inner-outer loop learning algorithm** co-optimizes the whole system via shared gradient-based credit assignment across recursion rounds.
- Theoretical analysis shows RecursiveMAS is more efficient than text-based MAS and maintains stable gradients during recursive training.
- Evaluated under 4 collaboration patterns across 9 benchmarks (math, science, medicine, search, code).
- Delivers an average **+8.3% accuracy**, **1.2×–2.4× inference speedup**, and **34.6%–75.6% token reduction** vs. single/multi-agent and recursive-computation baselines.
- Demonstrates a scaling law along the recursion-round axis for multi-agent collaboration.

## Concepts & entities

[[multi-agent]] · [[latent-reasoning]] · [[llm-agent]] · [[self-improving-agent]]

## References

- [arXiv:2604.25917](https://arxiv.org/abs/2604.25917)
- Project page: https://recursivemas.github.io
