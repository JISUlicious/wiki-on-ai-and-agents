---
title: "Training Compute-Optimal Large Language Models (Chinchilla)"
type: source
created: 2026-05-14
updated: 2026-05-14
sources:
  - hoffmann-chinchilla-2022.md
arxiv_id: "2203.15556"
venue: NeurIPS 2022
authors:
  - "[[jordan-hoffmann]]"
  - Sebastian Borgeaud
  - Arthur Mensch
  - Elena Buchatskaya
  - Trevor Cai
  - Eliza Rutherford
  - Jack W. Rae
  - "[[oriol-vinyals]]"
  - Laurent Sifre
first_author: "[[jordan-hoffmann]]"
introduces:
  - "[[chinchilla]]"
year: 2022
tags:
  - 2022
status: complete
importance: high
---

# Training Compute-Optimal Large Language Models (Chinchilla)

**Source**: `sources/hoffmann-chinchilla-2022.md` (extracted from `hoffmann-chinchilla-2022.pdf`)
**arXiv**: [2203.15556](https://arxiv.org/abs/2203.15556)
**Venue**: NeurIPS 2022
**Authors**: [[jordan-hoffmann]], Sebastian Borgeaud, Arthur Mensch, ..., Jack W. Rae, [[oriol-vinyals]], Laurent Sifre ([[deepmind|DeepMind]])

## Summary

This paper **corrects** the [[scaling-laws-kaplan-2020|Kaplan scaling laws]] (2020) and proposes a new compute-optimal training recipe for large language models. By training over 400 models ranging from 70M to 16B parameters on 5B to 500B tokens, the authors derive that **for compute-optimal training, model size and training tokens should be scaled in equal proportion** — for every doubling of parameters, double the training tokens.

This is a substantial departure from the Kaplan recommendation, which suggested making models much larger than data and stopping early. Under the Chinchilla view, the GPT-3-era models — GPT-3 (175B / ~300B tokens), Gopher (280B / 300B tokens), MT-NLG (530B / 270B tokens), Jurassic-1 (178B / ~300B tokens) — were significantly **undertrained on data**. They could have done much better if trained on more tokens at the same compute.

The paper validates the hypothesis empirically by training **Chinchilla** (70B parameters, 1.4T tokens) at the same compute budget as Gopher (280B / 300B tokens). Chinchilla uniformly beats Gopher (4× larger), GPT-3, Jurassic-1, and MT-NLG across a wide range of downstream tasks — reaching **67.5% on MMLU**, a 7-point gain over Gopher.

This finding reshaped subsequent training recipes. The Llama series (2023), Gemini (2023), Mistral, and most open-source LLMs post-2022 follow Chinchilla scaling — moderate parameter count, vastly more data. The Chinchilla recipe is also the reason inference-cost economics improved: small Chinchilla-optimal models are *much* cheaper to serve than large undertrained ones with comparable quality.

See [[scaling-laws]] for the broader treatment.

## Key Points

- **Methodology**: 400+ models from 70M to 16B params on 5B–500B tokens, with multiple compute budgets. Fit three different parametric models of the loss surface; all three agree on the equal-scaling recommendation.
- **Headline rule**: for fixed compute, optimal `N` (parameters) and `D` (tokens) scale roughly **as √C**. Equivalently, double the model size → double the training tokens.
- **GPT-3 era was undertrained**: at GPT-3's compute budget, the compute-optimal recipe would have used ~40B params with ~1.2T tokens — not 175B with 300B tokens.
- **Chinchilla**: 70B params, 1.4T tokens, same compute as Gopher (280B / 300B). Beats Gopher, GPT-3, Jurassic-1, MT-NLG across MMLU, BIG-bench, reading comprehension, common-sense reasoning.
- **MMLU**: 67.5% (+7 over Gopher).
- **Inference cost** as a bonus: smaller compute-optimal models are dramatically cheaper to serve.

## Entities Mentioned

- [[jordan-hoffmann]], Sebastian Borgeaud, Arthur Mensch — co-leads (the same DeepMind cluster as [[retro|RETRO]])
- [[chinchilla]] — the proof-point model
- [[oriol-vinyals]] — co-author (also on [[sequence-to-sequence-learning-sutskever-2014|seq2seq]])
- [[deepmind]] — author affiliation

## Concepts Discussed

- [[scaling-laws]] — corrected here vs. Kaplan
- [[chinchilla]] — the proof model

## Notable Quotes

> "We find that for compute-optimal training, the model size and the number of training tokens should be scaled equally: for every doubling of model size the number of training tokens should also be doubled." — Abstract

> "Chinchilla uniformly and significantly outperforms Gopher (280B), GPT-3 (175B), Jurassic-1 (178B), and Megatron-Turing NLG (530B) on a large range of downstream evaluation tasks." — Abstract

## References

_Original source: `sources/hoffmann-chinchilla-2022.md`_
_Corrects: [[scaling-laws-kaplan-2020]]_
