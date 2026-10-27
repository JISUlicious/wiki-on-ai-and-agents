---
title: LLaMA-Factory
type: entity
created: 2026-10-27
updated: 2026-10-27
sources:
  - llama-factory-hardware-requirement-2026-10-27.md
tags:
  - product
status: draft
importance: low
---

# LLaMA-Factory

An open-source unified fine-tuning framework ([hiyouga/LLaMA-Factory](https://github.com/hiyouga/LLaMA-Factory)). It supports full, freeze, LoRA, GaLore, APOLLO, BAdam, OFT and QLoRA/QOFT at 8, 4 or 2 bits, plus DPO and other post-training. Its README's **Hardware Requirement** table gives the compact per-method rule:
- full fine-tuning (mixed precision): 18x GB;
- pure bf16: 8x;
- LoRA: 2x;
- QLoRA 4-bit: x/2 (x = billions of parameters).

See [[llama-factory-hardware-requirement-2026]].

## References

- [[llama-factory-hardware-requirement-2026]] · [[fine-tuning-hardware-requirements-local-llms]]
