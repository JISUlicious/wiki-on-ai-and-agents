---
title: Fine-Tuning
type: concept
created: 2026-05-13
updated: 2026-05-13
sources:
  - bert-devlin-2018.md
status: complete
importance: high
---

# Fine-Tuning

Fine-tuning takes a [[pre-training|pre-trained]] model and continues training it on a downstream task with task-specific labeled data. Unlike feature-based transfer (where the pre-trained model's outputs are frozen and fed to a separate task-specific architecture), fine-tuning updates **all** of the pre-trained parameters end-to-end, typically with a small task-specific head added on top.

## In BERT and GPT

[[bert|BERT]] popularized the fine-tuning paradigm for encoder-style pre-training: every downstream task — classification, span extraction for QA, sequence labeling — is solved by fine-tuning the same pre-trained encoder with a thin task head. [[gpt-1|GPT-1]] used the same paradigm independently for an autoregressive base model.

## Modern variants

The general "fine-tuning" terminology has expanded considerably:

- **Supervised fine-tuning (SFT)**: the original meaning — labeled task data, full backprop.
- **Instruction tuning**: fine-tuning on (instruction, response) pairs to make the base model follow natural-language instructions (FLAN, T0, Alpaca, etc.).
- **RLHF**: reinforcement learning from human feedback, used to align models to human preferences after SFT (InstructGPT, [[gpt-4]], Claude).
- **Parameter-efficient fine-tuning (PEFT)**: LoRA, adapters, prefix tuning — update only a small fraction of parameters or add small auxiliary modules.

## Fine-tuning vs. in-context learning

[[gpt-3-brown-2020|GPT-3]] showed that for many tasks, sufficiently large models can be specialized via prompting alone — no parameter updates. Where fine-tuning was the dominant adaptation mechanism in the BERT/GPT-2 era, [[in-context-learning|in-context learning]] and prompting became co-dominant in the GPT-3+ era for general-purpose models.

## References

- [[bert-devlin-2018]]
