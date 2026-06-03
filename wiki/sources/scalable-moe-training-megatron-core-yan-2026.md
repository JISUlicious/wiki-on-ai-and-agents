---
title: "Scalable Training of Mixture-of-Experts Models with Megatron Core"
type: source
created: 2026-06-03
updated: 2026-06-03
sources:
  - scalable-moe-training-megatron-core-yan-2026.pdf
arxiv_id: "2603.07685"
authors:
  - Zijie Yan
  - Hongxiao Bai
  - Xin Yao
  - Dennis Liu
  - NVIDIA (full author list in report)
first_author: Zijie Yan
year: 2026
tags: [2026]
status: complete
importance: medium
---

# Scalable Training of Mixture-of-Experts Models with Megatron Core

Surfaced via the alphaXiv / NLP-newsletter digest.

## Summary

This NVIDIA technical report explains the systems-level techniques behind Megatron Core's Mixture-of-Experts (MoE) training stack. MoE sparsity lets total parameters grow much faster than per-token compute, but creates coupled constraints across memory, communication, and computation, where optimizing one dimension shifts pressure to another. The report co-designs across the full stack to relieve all three "walls" at once.

Concretely it integrates fine-grained recomputation and CPU offloading (memory), optimized all-to-all token dispatchers with communication-computation overlap (communication), and Grouped GEMM, kernel fusion, and CUDA Graphs (compute). It adds Parallel Folding for flexible multi-dimensional parallelism, FP8/NVFP4 low-precision training, and efficient long-context training. On GB300/GB200 it reaches 1,233/1,048 TFLOPS/GPU for DeepSeek-V3-685B and 974/919 for Qwen3-235B, and is used to train MoE models from billions to trillions of parameters on thousands of GPUs.

## Key points

- MoE Parallel Folding decouples attention and MoE layer parallelism, breaking the EP ≤ DP constraint for flexible mapping to hardware topology.
- Memory optimizations (fine-grained recompute, BF16-moment optimizers, CPU activation offload) cut DeepSeek-V3 footprint from 199.5 GB to under 80 GB/GPU.
- High-performance dispatchers (DeepEP, HybridEP) plus overlap hide all-to-all behind expert compute.
- Grouped GEMM, kernel fusion, CUDA Graphs, and sync-free execution maximize compute utilization for dropless MoE.
- Full FP8 with selective precision protects router/embeddings/optimizer states while quantizing bulk compute.
- Production features: load balancing, token dropping, distributed checkpointing/resharding, dense-to-MoE upcycling, multi-token prediction, and RL post-training support (router replay, packed sequences).

## Concepts & entities

- [[mixture-of-experts]]
- [[scaling-laws]]
- [[pre-training]]
- [[long-context-llm]]
- [[reinforcement-learning]]
- [[deepseek]]

## References

- [arXiv:2603.07685](https://arxiv.org/abs/2603.07685)
