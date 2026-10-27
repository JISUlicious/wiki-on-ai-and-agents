---
title: "Unsloth: Gemma 4 Fine-tuning Guide"
type: source
created: 2026-10-27
updated: 2026-10-27
sources:
  - unsloth-gemma-4-fine-tune-guide-2026.md
tags:
  - 2026
  - docs
status: complete
importance: medium
year: 2026
venue: "Unsloth documentation"
url: https://unsloth.ai/docs/models/gemma-4/train
---

# Unsloth: Gemma 4 Fine-tuning Guide

**Source:** `sources/unsloth-gemma-4-fine-tune-guide-2026.md`, from [unsloth.ai/docs/models/gemma-4/train](https://unsloth.ai/docs/models/gemma-4/train), fetched 2026-10-27. These are vendor documentation figures from [[unsloth]].

## VRAM (Unsloth's figures)

| [[gemma\|Gemma 4]] model | Method | VRAM |
|---|---|---|
| E2B | training / LoRA | 8 GB (LoRA "8-10GB") |
| E4B | training | 10 GB |
| E4B | LoRA | **17 GB** |
| 31B (dense) | QLoRA | **22 GB** |
| 26B-A4B (MoE) | LoRA | **> 40 GB** |
| E2B | RL | 9 GB |

- **Speed claim:** "~1.5x faster with ~60% less VRAM than FA2 setups (no accuracy loss)."
- **Advice for small models:** train **E4B with QLoRA** rather than E2B with LoRA, because the quantization accuracy difference is "miniscule". (Unlike the Qwen3.5 guide, QLoRA is endorsed here.)
- **MoE (26B-A4B):** use LoRA, not full fine-tuning, and prefer bf16 LoRA "if memory allows".
- **Implementation detail:** E2B and E4B share KV state across layers (20 and 18 shared layers). Training with `use_cache=False`, as most QLoRA tutorials set, breaks that sharing, and Unsloth handles it. This is an instance of [[cross-layer-kv-sharing]] interacting with training code.

> [!note] Third-party figures conflict
> Some 2026 blogs list Gemma 4 E4B LoRA at 8 GB and 26B QLoRA at 12 GB. Those contradict Unsloth's 17 GB and 22 GB (31B) figures, so they are not used in the wiki.

## Related

[[fine-tuning-hardware-requirements-local-llms]] · [[gemma]] · [[lora]] · [[qlora-dettmers-2023]] · [[unsloth-qwen3.5-fine-tune-guide-2026]]
