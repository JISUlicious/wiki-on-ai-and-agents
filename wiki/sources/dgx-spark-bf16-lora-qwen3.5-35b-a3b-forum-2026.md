---
title: "Bf16 LoRA fine-tuning of Qwen3.5-35B-A3B on DGX Spark (NVIDIA forum)"
type: source
created: 2026-10-27
updated: 2026-10-27
sources:
  - dgx-spark-bf16-lora-qwen3.5-35b-a3b-forum-2026.md
tags:
  - 2026
  - forum
status: complete
importance: low
year: 2026
venue: "NVIDIA Developer Forums (DGX Spark / GB10)"
url: https://forums.developer.nvidia.com/t/bf16-lora-fine-tuning-of-qwen3-5-35b-a3b-on-dgx-spark-no-quantization-required/363268
---

# Bf16 LoRA of Qwen3.5-35B-A3B on DGX Spark

**Source:** `sources/dgx-spark-bf16-lora-qwen3.5-35b-a3b-forum-2026.md`, a community post plus replies on the [NVIDIA forum](https://forums.developer.nvidia.com/t/bf16-lora-fine-tuning-of-qwen3-5-35b-a3b-on-dgx-spark-no-quantization-required/363268).

## Finding: loading is the bottleneck, not training

- **The hardware:** [[dgx-spark]] (GB10, 128 GB unified memory, about **119 GB usable**).
- **The model:** Qwen3.5-35B-A3B is **67 GB** in bf16.
- **Why loading fails:** standard Hugging Face loading OOMs at 66%, because the page cache of memory-mapped shards (~67 GB) plus materialized CUDA tensors (~67 GB) total about **134 GB**.
  - On unified memory the two copies compete for the same RAM.
  - `load_in_4bit=True` OOMs even earlier.
- **The fix:** load shards one at a time and evict the page cache. Peak drops to about **72 GB**, leaving about 47 GB for LoRA adapters, optimizer state and activations.
- **Training setup:** [[unsloth]] + TRL SFTTrainer, LoRA rank 16 on all attention and MLP projections, batch 1.

## Caveats from the thread

- **OOM makes the box unresponsive.** PyTorch's allocator behaves badly on unified memory.
- **GB10 build flag:** `TORCH_CUDA_ARCH_LIST` must be 12.1 (sm_121).
- **Deployment gap:** MoE LoRA adapters trained with different stacks store fused expert matrices in different layouts. A related thread reports adapters failing to load in [[vllm]].
- **Full-weight alternative:** another user runs full continued pretraining across 3 Sparks with FSDP expert sharding (11.55B parameters per rank).

This agrees with [[unsloth-qwen3.5-fine-tune-guide-2026|Unsloth's 74 GB]] figure for bf16 LoRA.

## Related

[[fine-tuning-hardware-requirements-local-llms]] · [[dgx-spark]] · [[mixture-of-experts]] · [[unsloth-qwen3.5-fine-tune-guide-2026]] · [[pytorch-fsdp-zhao-2023]]
