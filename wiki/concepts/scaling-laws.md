---
title: Scaling Laws
type: concept
created: 2026-05-13
updated: 2026-05-13
sources:
  - gpt-3-brown-2020.md
status: complete
importance: high
---

# Scaling Laws

Scaling laws are empirical **power-law relationships** between a language model's loss (typically cross-entropy on held-out data) and the resources spent training it — parameters `N`, training tokens `D`, and compute `C`. The first influential statement was Kaplan et al. 2020 ("Scaling Laws for Neural Language Models", arXiv 2001.08361), and the relationships were re-derived with corrections by Hoffmann et al. 2022 ("Chinchilla").

## Kaplan et al. 2020 (the "GPT-3 era" laws)

Across many orders of magnitude:

- Loss scales as a power law in `N` (parameters), `D` (tokens), and `C` (compute), each holding the others non-bottlenecking.
- For a fixed compute budget, **bigger models trained on relatively less data** are optimal — implying that compute should be spent on parameters rather than tokens.

This directly motivated [[gpt-3-brown-2020|GPT-3]]'s 175B-parameter scale (Kaplan was a GPT-3 co-author).

## Hoffmann et al. 2022 ("Chinchilla")

Re-derived the laws with better methodology and concluded:

- The Kaplan recipe **under-trained** models on data.
- For a fixed compute budget, **parameters and tokens should scale roughly equally** (model size and data size both ~ √C).
- Most GPT-3-era models (GPT-3, MT-NLG, Gopher) were significantly larger than compute-optimal — a 70B model trained on more data ("Chinchilla") matched or beat a 280B model.

The Chinchilla picture shifted the field toward training smaller models on more data — visible in LLaMA, LLaMA-2, Mistral, etc.

## Why scaling laws matter

- **Predictability**: you can extrapolate the loss of a model 100× larger from smaller-scale runs, before spending the compute.
- **Compute-optimal allocation**: scaling laws tell you the right N/D ratio for a given budget.
- **Capability emergence**: while loss scales smoothly, *downstream capabilities* often emerge non-smoothly with scale (see [[gpt-3-brown-2020|GPT-3's 3-digit arithmetic]] at 13B). Whether emergence is a real phenomenon or a metric artifact (Schaeffer et al. 2023) is contested.

## Beyond pre-training

Scaling laws have also been derived for:

- **Fine-tuning** (Hernandez et al. 2021): transfer benefits scale with both base-model size and target-task data.
- **Inference compute** (test-time scaling, 2024+): test-time chain-of-thought / search expenditure can substitute for parameter scale in reasoning tasks.
- **Data quality** (Penedo et al. 2023, FineWeb): the constant in the power law depends on data quality, not just quantity.

## References

- [[gpt-3-brown-2020]]
