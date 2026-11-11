---
title: LoRA (Low-Rank Adaptation)
type: concept
created: 2026-06-19
updated: 2026-11-11
sources:
  - understanding-lora-as-knowledge-memory-back-2026.md
  - the-surprising-effectiveness-of-deleting-weights-lu-2026.md
status: draft
importance: medium
tags:
  - 2026
---

# LoRA (Low-Rank Adaptation)

LoRA (Low-Rank Adaptation) is a parameter-efficient fine-tuning method that freezes the base model and learns small low-rank update matrices for selected weights. This drastically cuts the number of trainable parameters and storage cost while retaining most of full fine-tuning's quality. Beyond efficiency, LoRA is also studied as a knowledge-memory substrate for injecting and storing new information.

## References

- [[understanding-lora-as-knowledge-memory-back-2026]]

## Training systems

- [[training-memory-anatomy]] — why PEFT wins: a frozen base has no gradients and no optimizer states — it deletes the 12Ψ term

## Lineage sources

- [[fine-tuning-hardware-requirements-local-llms]] — bf16 LoRA VRAM for current models: Qwen3.5 0.8B → 27B = 3 / 5 / 10 / 22 / 56 GB; 35B-A3B MoE 74 GB; Gemma 4 E4B 17 GB ([[unsloth-qwen3.5-fine-tune-guide-2026]], [[unsloth-gemma-4-fine-tune-guide-2026]])
- [[logra-low-rank-gradient-sketches-rl-zhang-2026]] — LoGRA vs LoRA-256 in RL: less memory (7.18 vs 13.21 GiB) but LoRA wins Pass@1 and speed

## Newer evidence (2026-11-11)

- [[deletion-tuning|Deletion Tuning]] beats LoRA r=1 on adapter size (8.4 MB vs 10.0 MB at Llama8B) and on forgetting. LoRA's forward KL reaches 0.4–1.3 under GRPO, against ≤0.05 for deletion. LoRA's update geometry (stable rank 1–2, spectrum drift up to 84× FFT) is far from full fine-tuning. ([[the-surprising-effectiveness-of-deleting-weights-lu-2026]])
