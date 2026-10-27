---
title: NVIDIA DGX Spark
type: entity
created: 2026-10-27
updated: 2026-10-27
sources:
  - dgx-spark-bf16-lora-qwen3.5-35b-a3b-forum-2026.md
tags:
  - product
status: draft
importance: low
---

# NVIDIA DGX Spark

[[nvidia]]'s desktop "personal AI supercomputer", built on the GB10 Grace Blackwell chip (sm_121), with **128 GB of unified CPU–GPU memory**, about 119 GB usable.

In this wiki it is the smallest single box that fits **bf16 LoRA of Qwen3.5-35B-A3B** (74 GB). That works only after working around a loading peak: memory-mapped page cache plus materialized tensors, about 2× the model size ([[dgx-spark-bf16-lora-qwen3.5-35b-a3b-forum-2026]]).

**Capacity, not throughput.**
- About **273 GB/s** LPDDR5x bandwidth and about 500 TFLOPS dense FP4.
- Against an 80 GB H100 (about 3.35 TB/s, ~990 TFLOPS BF16 per datasheet): roughly **12× less bandwidth and ~8× less BF16 compute** (estimated).
- So it fits bigger fine-tunes but runs them much more slowly. See [[fine-tuning-hardware-requirements-local-llms]] §3b.

**Known rough edges:**
- an OOM makes the machine unresponsive;
- PyTorch must be built for arch 12.1;
- MoE LoRA adapters may not load in vLLM.

## References

- [[dgx-spark-bf16-lora-qwen3.5-35b-a3b-forum-2026]] · [[fine-tuning-hardware-requirements-local-llms]]
