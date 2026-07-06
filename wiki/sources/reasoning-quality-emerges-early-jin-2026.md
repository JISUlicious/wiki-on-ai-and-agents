---
title: "Reasoning Quality Emerges Early: Data Curation for Reasoning Models"
type: source
created: 2026-07-06
updated: 2026-07-06
sources:
  - reasoning-quality-emerges-early-jin-2026.pdf
arxiv_id: "2606.26797"
authors:
  - Hongyi Henry Jin
  - Wenhan Yang
  - Meysam Ghaffari
  - Carlos Morato
  - Baharan Mirzasoleiman
first_author: Hongyi Henry Jin
year: 2026
tags: [2026]
status: complete
importance: medium
---

# Reasoning Quality Emerges Early: Data Curation for Reasoning Models

Surfaced via the NLP newsletter (Top AI Papers of the Week, 2026-06-28–07-05).

## Summary

Supervised fine-tuning (SFT) on a small, high-quality set of long reasoning traces effectively elicits strong reasoning in LLMs, but existing curation relies on expensive strong-model filtering for diversity and difficulty. This UCLA (+ Optum AI / UnitedHealth) paper shows that **a trace's quality is largely decided in its opening tokens**: diverse and challenging examples can be identified using only the initial reasoning tokens, obviating post-hoc LLM filtering. Difficult problems are reliably detected from the **loss of the first ~100 reasoning tokens** evaluated at a randomly perturbed checkpoint of the pretrained model.

## Key points

- A short prefix predicts whole-trace quality: difficulty is detectable from the **first 100 tokens** out of traces up to 91k tokens long, flagged as those with higher loss at a perturbed checkpoint.
- Examples with similar loss over their **first 1k tokens** across a few noisy checkpoints (extrapolating along the fine-tuning direction) provably induce similar gradients during SFT.
- Method: **Token-Efficient Model Perturbation (TEMP)**-based data selection curates SFT sets that are both diverse and difficult without expensive strong-model filtering.
- Validated by fine-tuning **Qwen2.5-7B** and **Llama3.1-8B** on M23K (medical reasoning) and OpenThoughts-Math.
- Outperforms existing baselines by **up to 1.7%** while being **91% more token efficient**.
- Core thesis: reasoning-trace quality can be inferred early enough to rank/filter cheaply, since SFT changes parameters only slightly.

## Concepts & entities

[[reasoning]] · [[chain-of-thought]]

## References

- [arXiv:2606.26797](https://arxiv.org/abs/2606.26797)
