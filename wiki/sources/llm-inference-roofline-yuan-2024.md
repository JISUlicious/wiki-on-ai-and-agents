---
title: "LLM Inference Unveiled: Survey and Roofline Model Insights"
type: source
created: 2026-07-07
updated: 2026-07-07
sources: []
arxiv_id: "2402.16363"
year: 2024
authors:
  - Zhihang Yuan
tags:
  - 2024
  - survey
status: complete
importance: high
---

# LLM Inference Unveiled: Survey and Roofline Model Insights

**Yuan et al.** — [arXiv:2402.16363](https://arxiv.org/abs/2402.16363), 2024.

## Summary

A survey of LLM inference efficiency whose lasting contribution is a **roofline analysis** that makes the prefill/decode split quantitative. It supplies the per-layer arithmetic-intensity measurements that explain why [[quantization]] accelerates decode but not prefill.

## The core measurement

Llama-2-7B on an NVIDIA A6000 (155 TFLOPS FP16, 768 GB/s -> ridge point ~200 OPs/byte):

| Phase | Layer | Arithmetic intensity | Bound |
|---|---|---|---|
| Prefill | q/k/v/o_proj | **1,024** | compute |
| Prefill | gate/up/down_proj | **1,215** | compute |
| Prefill | qk_matmul | 114 | memory |
| Decode | all projections | **1** | memory |
| Decode | attention ops | **0.99** | memory |

A ~1000x gap between the two phases of one model. Decode sits ~200x below the ridge point (pure bandwidth); prefill sits far above it (pure compute).

## Consequences the paper states directly

- "When the batch size is large, compressing the network's weights from 4 bits to 2 bits or 1 bit does not lead to a decrease in the inference time."
- "As the sequence length increases, the prefill stage becomes more compute-bound," limiting quantization benefits.

Both are the roofline expressed operationally: weight-only quantization relieves a constraint that stops binding once arithmetic intensity crosses the ridge.

## Concepts

- [[quantization-performance]] — the synthesis built on this analysis.
- [[quantization]] · [[kv-cache]]

## References

- [arXiv:2402.16363](https://arxiv.org/abs/2402.16363)
