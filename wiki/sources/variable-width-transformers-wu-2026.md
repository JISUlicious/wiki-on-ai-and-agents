---
title: "Variable-Width Transformers"
type: source
created: 2026-06-29
updated: 2026-06-29
sources:
  - variable-width-transformers-wu-2026.pdf
arxiv_id: "2606.18246"
authors:
  - Zhaofeng Wu
  - Oliver Sieberling
  - Shawn Tan
  - et al.
first_author: Zhaofeng Wu
year: 2026
venue: MIT / MIT-IBM Watson AI Lab (preprint)
tags: [2026]
status: complete
importance: medium
---

# Variable-Width Transformers

Surfaced via the alphaXiv weekly digest (2026-06-24).

## Summary

Most transformers keep a constant hidden width across all layers, spending the same parameter/compute budget on every layer even though layers may play different computational roles. This paper studies nonuniform width allocation across depth, proposing a "><former" (variable-width transformer) that changes the residual dimension between layers via a parameter-free residual resizing mechanism (inactive dimensions copied upward in the residual stream). After sweeping shapes — growing, narrowing, grow-then-narrow, and narrow-then-grow — the authors find an x-shaped model works best: wider early and late layers with a narrower middle bottleneck. Because attention FLOPs and KV-cache scale with hidden dimension, shrinking the average layer width yields compute and memory savings while matching parameter count, and the bottleneck even produces qualitatively different residual-stream representations.

## Key points

- x-shaped width schedule (wide ends, narrow middle) beats parameter-matched constant-width baselines on LM loss; parameter-free residual resizing avoids extra weights.
- Evaluated on decoder-only LMs from 200M to 2B (dense) and a 3B-total / 1B-active [[mixture-of-experts]] model.
- ~3% relative perplexity improvement over parameter-matched uniform baselines across 200M-2B.
- ~22% FLOP reduction under fitted loss-matched scaling curves; ~15% smaller KV-cache memory and I/O cost (about 10% KV-cache reduction at the 200M-2B parameter-matched setting).
- Theory: when parameter-matched, dense FLOPs are identical but average layer width is strictly smaller (square-of-mean vs mean-of-squares), lowering attention FLOPs and KV-cache.
- Provides a recipe for bottleneck layer index and dimension that transfers across model budgets; code released.

## Concepts & entities

- [[transformer-architecture]] — modifies the canonical constant-width stack into a depth-varying ("x-shaped") width schedule.
- [[mixture-of-experts]] — one tested configuration is a 3B-total / 1B-active MoE variant.

## References

- arXiv: [arXiv:2606.18246](https://arxiv.org/abs/2606.18246)
- Code: https://github.com/ZhaofengWu/variable-width-transformers
- Local source: `variable-width-transformers-wu-2026.pdf`
