---
title: "Quantization Hurts Reasoning? An Empirical Study on Quantized Reasoning Models"
type: source
created: 2026-07-07
updated: 2026-07-07
sources: []
arxiv_id: "2504.04823"
venue: COLM 2025
year: 2025
authors:
  - Ruikang Liu
tags:
  - 2025
  - paper
status: complete
importance: medium
---

# Quantization Hurts Reasoning? An Empirical Study on Quantized Reasoning Models

**Liu et al.** — [arXiv:2504.04823](https://arxiv.org/abs/2504.04823), **COLM 2025**.

Models: DeepSeek-R1-Distill-Qwen 1.5B/7B/14B/32B, R1-Distill-LLaMA 8B/70B, QwQ-32B. Tasks: AIME-120, MATH-500, GSM8K, LiveCodeBench.

## Task difficulty scales the damage

*"Harder tasks suffer up to 4x greater degradation than simpler ones."* At W4A4KV4 on the 32B model:

| Task | Delta |
|---|---|
| AIME (hardest) | **-3.9%** |
| MATH-500 | -1.2% |
| LiveCodeBench | -1.1% |
| GSM8K (easiest) | **-0.0%** |

AIME-120 on DeepSeek-R1-Distill-Qwen-32B (BF16 = 74.49): W8A8 74.54 (+0.05), KV4 74.41, W4A16 73.67, W4A4 72.17, W3A16 71.26.

Notably this puts **math well above code** in sensitivity - relevant to the unresolved question of whether code is disproportionately damaged ([[quantization-quality]]).

## Model size dominates

Small reasoning models collapse where large ones shrug. W3G128: **Qwen-1.5B -16.58%** vs **Qwen-32B -3.23%**.

Aggressive quantization also *lengthens* outputs in small models (W3G128, W4A4KV4 on 1.5B) - the model reasons longer and still does worse.

## Concepts

- [[quantization-quality]] · [[quantization]] · [[chain-of-thought]]

## References

- [arXiv:2504.04823](https://arxiv.org/abs/2504.04823)

