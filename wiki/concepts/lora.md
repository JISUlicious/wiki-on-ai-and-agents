---
title: LoRA (Low-Rank Adaptation)
type: concept
created: 2026-06-19
updated: 2026-10-27
sources:
  - understanding-lora-as-knowledge-memory-back-2026.md
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
