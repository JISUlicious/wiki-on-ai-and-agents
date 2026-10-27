---
title: "LLaMA-Factory: Hardware Requirement table"
type: source
created: 2026-10-27
updated: 2026-10-27
sources:
  - llama-factory-hardware-requirement-2026-10-27.md
tags:
  - 2026
  - docs
status: complete
importance: medium
year: 2026
venue: "GitHub README (hiyouga/LLaMA-Factory)"
url: https://github.com/hiyouga/LLaMA-Factory
---

# LLaMA-Factory: Hardware Requirement table

**Source:** `sources/llama-factory-hardware-requirement-2026-10-27.md`, the "Hardware Requirement" section of the upstream [[llama-factory]] README, fetched 2026-10-27 and marked "*estimated*". It is the most compact **per-method rule of thumb** available.

| Method | Bits | 7B | 14B | 30B | 70B | **x B** |
|---|---|---|---|---|---|---|
| Full (`bf16`/`fp16` mixed precision) | 32 | 120 GB | 240 GB | 600 GB | 1200 GB | **18x GB** |
| Full (`pure_bf16`) | 16 | 60 GB | 120 GB | 300 GB | 600 GB | **8x GB** |
| Freeze / LoRA / GaLore / APOLLO / BAdam / OFT | 16 | 16 GB | 32 GB | 64 GB | 160 GB | **2x GB** |
| QLoRA / QOFT | 8 | 10 GB | 20 GB | 40 GB | 80 GB | **x GB** |
| QLoRA / QOFT | 4 | 6 GB | 12 GB | 24 GB | 48 GB | **x/2 GB** |
| QLoRA / QOFT | 2 | 4 GB | 8 GB | 16 GB | 24 GB | **x/4 GB** |

## How it relates to the theory

- **Full fine-tuning in mixed precision:** 18 bytes per parameter is the 16 bytes of [[training-memory-anatomy]] (fp16 weights and gradients, fp32 master copy, Adam moments) plus about 2 bytes of activations and overhead.
- **Pure bf16 (8 bytes per parameter):** drops the fp32 master copy and keeps the optimizer moments in bf16.
- **LoRA (about 2 bytes per parameter):** essentially the frozen bf16 weights. The adapters, their optimizer state and modest activations fit in the table's margin.
- **QLoRA:** roughly the quantized weight size.
- **Older numbers differ.** Earlier copies of this README listed full fine-tuning at 160 GB for 7B; the current table says 120 GB (mixed) or 60 GB (pure bf16). Third-party pages still quote the older figures.

All rows assume short context and small batches. Long sequences add activation memory on top, quadratic in sequence length without FlashAttention ([[training-memory-anatomy]]).

## Related

[[fine-tuning-hardware-requirements-local-llms]] · [[llama-factory]] · [[lora]] · [[qlora-dettmers-2023]] · [[zero-redundancy-optimizer]] · [[8-bit-optimizers-dettmers-2021]]
