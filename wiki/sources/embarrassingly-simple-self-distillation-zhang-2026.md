---
title: "Embarrassingly Simple Self-Distillation Improves Code Generation"
type: source
created: 2026-06-03
updated: 2026-06-03
sources:
  - embarrassingly-simple-self-distillation-zhang-2026.md
arxiv_id: "2604.01193"
authors:
  - Ruixiang Zhang
  - Richard He Bai
  - Huangjie Zheng
  - Yizhe Zhang
  - Navdeep Jaitly
  - Ronan Collobert
first_author: Ruixiang Zhang
year: 2026
tags:
  - 2026
status: complete
importance: medium
---

# Embarrassingly Simple Self-Distillation Improves Code Generation

Surfaced via the alphaXiv / NLP-newsletter digest.

## Summary

This Apple paper asks whether an LLM can improve at code generation using only its own raw outputs — no verifier, no teacher model, no reinforcement learning. The answer is yes, via Simple Self-Distillation (SSD): sample solutions from the model under specific temperature and truncation configurations, then fine-tune on those samples with standard supervised fine-tuning. SSD lifts Qwen3-30B-Instruct from 42.4% to 55.3% pass@1 on LiveCodeBench v6, with gains concentrated on harder problems, and it generalizes across Qwen and Llama models at 4B, 8B, and 30B scale, covering both instruct and thinking variants.

To explain why such a minimal method works, the authors trace the gains to a *precision–exploration conflict* in LLM decoding. SSD reshapes token distributions in a context-dependent way: it suppresses distractor tails where precision matters while preserving useful diversity where exploration matters. A notable detail is that the training-time temperature need not equal 1 (Ttrain != 1 yields the strongest gains, +12.9 points in their setup). The framing positions SSD as a complementary post-training direction alongside RL- and verifier-based approaches.

## Key points

- SSD: sample own outputs at tuned temperature/truncation, then plain SFT on those samples — no verifier, teacher, or RL.
- Qwen3-30B-Instruct: 42.4% → 55.3% pass@1 on LiveCodeBench v6; gains concentrate on harder problems.
- Generalizes across Qwen and Llama at 4B/8B/30B, instruct and thinking variants.
- Mechanism: resolves a precision–exploration conflict by suppressing distractor tails where precision matters, preserving diversity where exploration matters.
- Training temperature matters — Ttrain != 1 gives the largest improvement (+12.9 points).
- Presented as a complementary post-training direction, not a replacement for RL/verifier methods.

## Concepts & entities

- [[self-distillation]] / [[knowledge-distillation]] — the core method (distilling from own samples)
- [[code-generation]] — the target task
- [[supervised-fine-tuning]], [[reinforcement-learning]] (the alternative SSD complements)
- [[reasoning]] (thinking-variant models tested)
- [[humaneval]], LiveCodeBench (benchmark), [[qwen]], [[llama]]
- [[apple]] (authoring lab)

## References

- [arXiv:2604.01193](https://arxiv.org/abs/2604.01193) — Zhang et al., 2026 (v1, 1 Apr 2026).
- Code: github.com/apple/ml-ssd.
