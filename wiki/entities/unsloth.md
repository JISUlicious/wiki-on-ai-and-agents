---
title: Unsloth
type: entity
created: 2026-10-27
updated: 2026-10-27
sources:
  - unsloth-qwen3.5-fine-tune-guide-2026.md
  - unsloth-gemma-4-fine-tune-guide-2026.md
tags:
  - product
status: draft
importance: medium
---

# Unsloth

An open-source fine-tuning library with custom kernels for LoRA, QLoRA, full fine-tuning and RL. It integrates with Hugging Face TRL, and also ships notebooks, a no-code Studio app and per-model guides.

**Vendor claims:**
- Qwen3.5: 1.5× faster training and 50% less VRAM than FlashAttention-2 setups.
- Gemma 4: about 60% less VRAM.
- MoE kernels: about 12× faster MoE training.

Its per-model VRAM tables are the most-cited current figures:
- Qwen3.5-9B bf16 LoRA 22 GB;
- 35B-A3B 74 GB;
- Gemma 4 31B QLoRA 22 GB.

See [[fine-tuning-hardware-requirements-local-llms]].

**Notable guidance:**
- **Against 4-bit QLoRA for Qwen3.5,** dense or MoE.
- **Gradient checkpointing:** keep `use_gradient_checkpointing="unsloth"` on.

## References

- [[unsloth-qwen3.5-fine-tune-guide-2026]] · [[unsloth-gemma-4-fine-tune-guide-2026]] · [[dgx-spark-bf16-lora-qwen3.5-35b-a3b-forum-2026]]
