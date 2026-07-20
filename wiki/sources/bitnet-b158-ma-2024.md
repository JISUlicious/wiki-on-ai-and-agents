---
title: "The Era of 1-bit LLMs: All Large Language Models are in 1.58 Bits"
type: source
created: 2026-07-07
updated: 2026-07-07
sources: []
arxiv_id: "2402.17764"
year: 2024
authors:
  - Shuming Ma
  - Hongyu Wang
  - Furu Wei
tags:
  - 2024
  - paper
status: complete
importance: medium
---

# The Era of 1-bit LLMs: BitNet b1.58

**Ma, Wang, ..., Wei** (Microsoft Research) — [arXiv:2402.17764](https://arxiv.org/abs/2402.17764), 2024. Follows **BitNet** ([arXiv:2310.11453](https://arxiv.org/abs/2310.11453)).

## Mechanism

`nn.Linear` is replaced by `BitLinear`. Weights are constrained to **{-1, 0, +1}** via an **absmean** function: scale W by its mean absolute value, then round-and-clip to the ternary set. Activations are quantized to 8 bits (absmax, per-token). The name: log2(3) = **1.58 bits**.

The payoff is architectural rather than merely storage-related: matmuls become **addition-only** — no multiplications — which opens the door to dedicated hardware.

## Results

Trained on RedPajama for 100B tokens, at 3B parameters: PPL **9.91 vs LLaMA-3B's 10.04**, memory **2.22 GB (3.55x less)**, latency **1.87 ms (2.71x faster)**. The paper claims parity with FP16 starting from 3B at equal size and token count.

A real released model followed: **BitNet b1.58 2B4T** ([arXiv:2504.12285](https://arxiv.org/abs/2504.12285)) — 2B params, 4T tokens, "the first open-source, native 1-bit LLM at the 2-billion parameter scale", with GPU and CPU kernels. **BitNet a4.8** ([arXiv:2411.04965](https://arxiv.org/abs/2411.04965)) pushes activations to 4 bits.

> [!warning] Not a quantization method for existing models
> BitNet is **trained from scratch in ternary precision** — it is neither PTQ nor conversion-style QAT. Its results say nothing about quantizing an existing FP16 checkpoint to ternary, and the "1-bit LLM" branding is marketing-adjacent: weights are ternary at 1.58 bits and activations are 8-bit.

## Concepts

- [[quantization]] — the "never create outliers" family.

## References

- [arXiv:2402.17764](https://arxiv.org/abs/2402.17764)
