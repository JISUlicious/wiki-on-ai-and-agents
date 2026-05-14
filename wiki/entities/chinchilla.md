---
title: Chinchilla
type: entity
created: 2026-05-14
updated: 2026-05-14
sources:
  - hoffmann-chinchilla-2022.md
family: gopher
predecessor: null
successors: []
parameters: "70B"
release_date: 2022-03-29
status: complete
importance: high
tags:
  - model
---

# Chinchilla

Chinchilla is the 70B-parameter language model introduced in [[chinchilla-hoffmann-2022|"Training Compute-Optimal Large Language Models"]] (Hoffmann et al., DeepMind, NeurIPS 2022) as the empirical proof of the **Chinchilla scaling laws**.

## What makes it notable

Chinchilla and Gopher used the **same training compute budget**. Gopher: 280B parameters on ~300B tokens. Chinchilla: 70B parameters on **1.4T tokens** — 4× smaller, ~4.7× more data. Chinchilla uniformly outperforms Gopher (4× larger), GPT-3 (175B), Jurassic-1 (178B), and Megatron-Turing NLG (530B) on a wide range of downstream tasks, reaching **67.5% on MMLU** — a 7-point gain over Gopher at the same compute.

The result demonstrated that **GPT-3-era models were significantly undertrained on data**. The "make the model bigger, train less" recommendation from [[scaling-laws-kaplan-2020|Kaplan et al. 2020]] was wrong; for compute-optimal training, model size and training tokens should be scaled roughly equally.

## Architecture / training

- **Parameters**: 70B (decoder-only Transformer, similar architecture to Gopher).
- **Training tokens**: 1.4T tokens (MassiveText, similar corpus to Gopher).
- **Tokenizer**: SentencePiece, 32k vocabulary.
- **Compute**: ~equal to Gopher's training run (large but not disclosed precisely).
- **Released**: weights not publicly released.

## Impact on the field

Chinchilla scaling reshaped training recipes industry-wide:

- **Llama 1** (2023): 7B–65B params trained on 1T+ tokens — follows Chinchilla.
- **Llama 2** (2023): 7B/13B/70B on ~2T tokens — Chinchilla+.
- **Gemini, GPT-4 era**: all post-2022 training runs follow some variant of Chinchilla scaling.
- **Mistral 7B**: small + heavily over-trained relative to even Chinchilla — pushes the small-model-on-vast-data direction further.

The Chinchilla recipe also has economic consequences: **smaller compute-optimal models are dramatically cheaper to serve at inference time** than large undertrained ones, even at equal quality.

## References

- [[chinchilla-hoffmann-2022]]
