---
title: GPT-2
type: entity
created: 2026-05-13
updated: 2026-05-13
sources:
  - gpt-2-radford-2019.md
family: gpt
predecessor: "[[gpt-1]]"
successors:
  - "[[gpt-3]]"
parameters: "1.5B (largest of four sizes)"
release_date: 2019-02-14
status: complete
importance: high
tags:
  - model
---

# GPT-2

GPT-2 is the second model in [[openai|OpenAI]]'s [[gpt-family|GPT family]], introduced in "[[gpt-2-radford-2019|Language Models are Unsupervised Multitask Learners]]" (Radford et al., 2019).

## Architecture

A [[decoder-only-transformer|decoder-only Transformer]] continuation of [[gpt-1]] with several refinements:

- **Pre-norm** (LayerNorm at the *input* of each sublayer, plus a final LayerNorm after the last block) — replaces GPT-1's post-norm and stabilizes training at scale.
- **Scaled residual initialization** — weights of residual layers initialized with `1/√N` scaling to keep activation magnitudes bounded as depth grows.
- **Byte-level [[byte-pair-encoding|BPE]]**, vocabulary 50,257.
- **Context length** doubled from 512 to 1024 tokens.

Four model sizes were released:

| Variant | Layers | dmodel | Heads | Params |
|---|---|---|---|---|
| GPT-2 Small | 12 | 768 | 12 | 117M |
| GPT-2 Medium | 24 | 1024 | 16 | 345M |
| GPT-2 Large | 36 | 1280 | 20 | 762M |
| GPT-2 XL | 48 | 1600 | 25 | 1.5B |

## Training

- **Corpus**: WebText — ~40GB / 8M documents, scraped from outbound Reddit links with ≥3 karma. Wikipedia explicitly excluded to keep benchmarks clean.
- **Objective**: [[autoregressive-language-modeling|autoregressive next-token prediction]].
- **Compute**: not disclosed in detail.

## Key contribution

[[zero-shot-learning|Zero-shot]] task performance. By framing tasks in natural language (e.g., `"translate to french: ..."`), GPT-2 performs many NLP tasks without any fine-tuning or task-specific data. Performance scales **log-linearly** with model size — a finding that motivated the [[scaling-laws|scaling-laws]] research program and the much larger [[gpt-3]].

## Lineage

| Predecessor | Successor |
|---|---|
| [[gpt-1]] | [[gpt-3]] |

## Results (at release)

- SOTA on 7/8 zero-shot LM benchmarks (perplexity / BPC).
- 55 F1 on CoQA reading comprehension, zero-shot.
- "Still underfits WebText" — a foreshadowing of the data/scale dependencies that GPT-3 would exploit.

## Release history

GPT-2 was the first frontier model with a **staged release** justified by misuse concerns:

- Feb 2019: 124M released
- May 2019: 355M
- Aug 2019: 774M
- Nov 2019: 1.5B

This precedent shaped subsequent release-policy discourse around frontier models.

## References

- [[gpt-2-radford-2019]]
