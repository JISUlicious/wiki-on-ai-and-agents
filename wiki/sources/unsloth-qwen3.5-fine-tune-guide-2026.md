---
title: "Unsloth: Qwen3.5 Fine-tuning Guide"
type: source
created: 2026-10-27
updated: 2026-10-27
sources:
  - unsloth-qwen3.5-fine-tune-guide-2026.md
tags:
  - 2026
  - docs
status: complete
importance: medium
year: 2026
venue: "Unsloth documentation"
url: https://unsloth.ai/docs/models/qwen3.5/fine-tune
---

# Unsloth: Qwen3.5 Fine-tuning Guide

**Source:** `sources/unsloth-qwen3.5-fine-tune-guide-2026.md`, from [unsloth.ai/docs/models/qwen3.5/fine-tune](https://unsloth.ai/docs/models/qwen3.5/fine-tune), fetched 2026-10-27. These are vendor documentation figures from [[unsloth]], not independent measurements.

## VRAM for bf16 LoRA (Unsloth's figures)

| [[qwen\|Qwen3.5]] model | bf16 [[lora\|LoRA]] VRAM |
|---|---|
| 0.8B | 3 GB |
| 2B | 5 GB |
| 4B | 10 GB |
| 9B | 22 GB |
| 27B | 56 GB |
| 35B-A3B (MoE) | **74 GB** |
| 122B-A10B (MoE) | **256 GB** (multi-GPU: `device_map="balanced"`) |

- **Full fine-tuning:** "works as well. Note it will use 4x more VRAM."
- **Speed claim:** "1.5× faster and uses 50% less VRAM than FA2 setups."

## The QLoRA warning

> "It is not recommended to do QLoRA (4-bit) training on the Qwen3.5 models, no matter MoE or dense, due to higher than normal quantization differences."

For MoE models (35B-A3B, 122B-A10B, 397B-A17B), Unsloth recommends "bf16 setups (e.g. LoRA or full fine-tuning)", and says 4-bit MoE QLoRA "is not recommended due to BitsandBytes limitations".

Its MoE kernels claim "~12x faster MoE training … with >35% less VRAM & ~6x longer context". Keep `use_gradient_checkpointing="unsloth"` on.

> [!warning] Contradiction: QLoRA on Qwen3.5
> The general [[qlora-dettmers-2023|QLoRA]] result is that NF4 with double quantization "fully recovers 16-bit performance". Unsloth advises against 4-bit training on Qwen3.5 specifically, dense or MoE. Third-party posts report 4-bit runs of Qwen3.5-35B-A3B in about 17.5 GB on an RTX 5090; that claim is unverified. Treat 4-bit Qwen3.5 training as a quality risk to check on your own evaluations, not a free memory saving.

## Related

[[fine-tuning-hardware-requirements-local-llms]] · [[lora]] · [[qlora-dettmers-2023]] · [[training-memory-anatomy]] · [[mixture-of-experts]] · [[dgx-spark-bf16-lora-qwen3.5-35b-a3b-forum-2026]]
