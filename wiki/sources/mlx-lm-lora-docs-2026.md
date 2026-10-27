---
title: "MLX-LM: Fine-Tuning with LoRA or QLoRA (LORA.md)"
type: source
created: 2026-10-27
updated: 2026-10-27
sources:
  - mlx-lm-lora-docs-2026-10-27.md
  - mlx-lora-finetune-qwen3.5-sciences44-2026.md
tags:
  - 2026
  - docs
status: complete
importance: medium
year: 2026
venue: "GitHub docs (ml-explore/mlx-lm)"
url: https://github.com/ml-explore/mlx-lm/blob/main/mlx_lm/LORA.md
---

# MLX-LM: LoRA / QLoRA fine-tuning on Apple silicon

**Sources:**
- `sources/mlx-lm-lora-docs-2026-10-27.md`: the official [[mlx]] LoRA guide.
- `sources/mlx-lora-finetune-qwen3.5-sciences44-2026.md`: a community experiment (sciences44/mlx-lora-finetune) fine-tuning Qwen3.5 for text-to-SQL.

## How it works on a Mac

[[apple-silicon]]'s **unified memory is the VRAM**, so the limit is total RAM minus what the OS needs.
- **LoRA is the default.** Point `--model` at a quantized model and training becomes QLoRA automatically. Full fine-tuning (`--fine-tune-type full`) and DoRA are also available.
- **Memory levers, from the docs' "Memory Issues" section:**
  1. Use QLoRA.
  2. Lower `--batch-size` (default 4) and use gradient accumulation.
  3. Lower `--num-layers` (LoRA layers; default 16) to 8 or 4.
  4. Shorten or split examples.
  5. `--grad-checkpoint`.
- **The docs' example:** a **32 GB** machine runs a quantized model at batch 1 with 4 layers, at about 250 tokens/s on an **M1 Max**.

## Measured peak memory (community experiment, Qwen3.5, 600 iterations)

| Model | Peak RAM | Result (fine-tuned SQL) |
|---|---|---|
| Qwen3.5-0.8B | 3.9 GB | 86.5% |
| **Qwen3.5-2B** | **5.9 GB** | **95.0%** (best) |
| Qwen3.5-4B | 11.1 GB | 71.0% |
| Mistral-Nemo-12B (4-bit) | 8.2 GB | 57.0% |

- **Memory guide from the repo:**
  - 16 GB Mac: `BATCH_SIZE=1 NUM_LAYERS=4`;
  - 32 GB or more: batch 4;
  - 64 GB or more: turn gradient checkpointing off for speed.
- **Training time:** the 2B run took about 15 minutes for 600 iterations, training 0.48% of parameters.
- **Bigger wasn't better here.** On this narrow task the 2B model beat the 4B and 12B models.

## Related

[[fine-tuning-hardware-requirements-local-llms]] · [[mlx]] · [[apple-silicon]] · [[lora]] · [[qlora-dettmers-2023]]
