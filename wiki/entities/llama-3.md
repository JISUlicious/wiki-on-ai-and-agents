---
title: Llama 3
type: entity
created: 2026-09-29
updated: 2026-09-29
sources:
  - a-mem-xu-2025.md
  - memory-r1-yan-2025.md
  - trianglemix-decoding-time-contribution-sparsity-he-2025.md
  - probe-process-benchmark-hallucination-detection-2026.md
family: llama
predecessor: [[llama]]
release_date: 2024-04-18
status: draft
importance: medium
tags:
  - model
---

# Llama 3

Third generation of [[meta|Meta]]'s open-weights [[llama|LLaMA]] family. Llama 3 launched on 2024-04-18 at 8B and 70B. The 3.1 point release in July 2024 added a 405B dense model with a context window of up to 128K tokens, and 3.2 added small 1B/3B models. In this wiki the 3.x checkpoints, above all **Llama-3.1-8B-Instruct**, are the standard open substrate for memory, fine-tuning and especially long-context / inference-efficiency experiments.

## In this wiki

- **Memory systems.** [[memory-r1-yan-2025|Memory-R1]] uses LLaMA-3.1-8B-Instruct as its main backbone. [[a-mem-xu-2025|A-MEM]] evaluates on Llama 3.2 1B/3B alongside [[qwen-2-5|Qwen2.5]].
- **Fine-tuning baseline.** In [[probe-process-benchmark-hallucination-detection-2026|PROBE]], an SFT'd Llama-3.1-8B beats frontier models at evidence finding (see [[llama]]).
- **Inference-efficiency test bed.** Most prefill-sparsity and token-pruning papers here report on Llama 3 / 3.1 8B. [[minference-dynamic-sparse-attention-jiang-2024|MInference]] uses the long-context LLaMA-3-8B-262K / -1M variants. [[trianglemix-decoding-time-contribution-sparsity-he-2025|TriangleMix]], [[speculative-prefill-liu-2025|Speculative Prefill]] and [[sliminfer-dynamic-token-pruning-long-2025|SlimInfer]] use Llama 3.1. [[cross-family-speculative-prefill-upasani-2026|Cross-family speculative prefill]] pairs a Llama-3.2-1B draft with a Llama-3.1-8B target. See [[sparse-prefill]] and [[token-pruning]].
- **Quantization.** Llama 3.x appears throughout the measurements in [[quantization-quality]] and [[quantization-performance]].

## Lineage

It inherits the dense [[decoder-only-transformer]] recipe of LLaMA / Llama 2 (the Llama 3 Herd paper describes even the 405B flagship as a dense Transformer, not a [[mixture-of-experts|MoE]]). Relative to Llama 2 it scaled parameters to 405B and context to 128K with 3.1. The generation matters less for architecture than for its open weights across sizes; per-version details belong in [[llama]] once the technical report is ingested.

## Related

- [[llama]] · [[meta]] · [[qwen-2-5]] · [[long-context-llm]] · [[sparse-attention]] · [[vllm]]

## References

- [Meta, "Introducing Meta Llama 3" (2024-04-18)](https://ai.meta.com/blog/meta-llama-3/) — release date, 8B/70B
- [The Llama 3 Herd of Models, arXiv:2407.21783](https://arxiv.org/abs/2407.21783) — 405B dense, up to 128K context
- [[a-mem-xu-2025]] · [[memory-r1-yan-2025]] · [[trianglemix-decoding-time-contribution-sparsity-he-2025]] · [[probe-process-benchmark-hallucination-detection-2026]]
