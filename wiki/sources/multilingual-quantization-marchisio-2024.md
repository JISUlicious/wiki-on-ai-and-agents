---
title: "How Does Quantization Affect Multilingual LLMs?"
type: source
created: 2026-07-07
updated: 2026-07-07
sources: []
arxiv_id: "2407.03211"
venue: Findings of EMNLP 2024
year: 2024
authors:
  - Kelly Marchisio
  - Sara Hooker
tags:
  - 2024
  - paper
status: complete
importance: high
---

# How Does Quantization Affect Multilingual LLMs?

**Marchisio, Hooker et al.** (Cohere For AI) — [arXiv:2407.03211](https://arxiv.org/abs/2407.03211), **Findings of EMNLP 2024**.

Models: Command R (35B), Command R+ (103B), Aya 23 (8B, 35B). Methods: W8 per-column, W4-g GPTQ, W8A8.

## The most important number in the quantization-evaluation literature

*"Harmful effects of quantization are apparent in human evaluation, which automatic metrics severely underestimate: a 1.7% average drop in Japanese across automatic tasks corresponds to a 16.0% drop reported by human evaluators on realistic prompts."*

| Language | Automatic benchmarks | Human evaluators |
|---|---|---|
| French | **-0.3%** | **-16.6%** |
| Japanese | **-1.7%** | **-16.0%** |

A **10-55x underestimation**. Also: LLM-as-a-Judge on challenging prompts **-25.9%**; human judgment on real-world prompts -10.5%; MGSM math **-13.1%**.

Crucially, *discriminative* tasks (XWinograd, XCOPA, XStoryCloze) were "minimally impacted" - and those are exactly the task formats that dominate leaderboards. The evaluation instrument, not the model, is what makes quantization look free.

## Disparate impact by script and language

- Non-Latin script **-1.9%** vs Latin **-0.7%** (103B); -3.7% vs -3.0% for 8B Aya.
- Degradation correlates **negatively with training-set size per language** - low-resource languages take the hit.

## A methodological aside worth noting

Changing scaling granularity (group-wise vs column-wise) recovers **>6pp on math** - larger than most papers' entire reported bit-width effect. Recipe hyperparameters are a first-order confound in this literature.

## Concepts

- [[quantization-quality]] · [[quantization-safety]] · [[quantization]]

## References

- [arXiv:2407.03211](https://arxiv.org/abs/2407.03211)

