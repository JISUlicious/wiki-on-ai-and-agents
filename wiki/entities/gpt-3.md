---
title: GPT-3
type: entity
created: 2026-05-13
updated: 2026-05-13
sources:
  - gpt-3-brown-2020.md
family: gpt
predecessor: "[[gpt-2]]"
successors:
  - "[[gpt-4]]"
parameters: "175B (largest of 8 sizes; 125M / 350M / 760M / 1.3B / 2.7B / 6.7B / 13B / 175B)"
release_date: 2020-05-28
status: complete
importance: high
tags:
  - model
---

# GPT-3

GPT-3 is [[openai|OpenAI]]'s 175-billion-parameter [[decoder-only-transformer|decoder-only Transformer]] language model, introduced in "[[gpt-3-brown-2020|Language Models are Few-Shot Learners]]" (Brown et al., NeurIPS 2020, Best Paper).

## Architecture

Standard [[gpt-2|GPT-2]]-style architecture with one tweak — **alternating dense and locally-banded sparse attention patterns** in each layer (à la Sparse Transformer, Child et al. 2019), for memory efficiency at the 2048 context length.

| | Layers | dmodel | Heads | Params |
|---|---|---|---|---|
| Smallest | 12 | 768 | 12 | 125M |
| GPT-3 175B | **96** | **12288** | **96** | **175B** |

dhead = 128 throughout. 8 model sizes between these endpoints, enabling a clean scaling analysis.

## Training

- **Corpus**: ~300B tokens. Filtered Common Crawl (~60%), WebText2 (22%), Books1 (8%), Books2 (8%), English Wikipedia (3%). Common Crawl was filtered with a quality classifier and aggressively deduplicated.
- **Objective**: [[autoregressive-language-modeling|next-token prediction]].
- **Compute**: ~3640 PetaFLOP/s-days for the 175B model — by far the largest LM training run of its era.

## Key contribution

[[in-context-learning|In-context learning]] / [[few-shot-learning|few-shot prompting]]. With sufficient scale, tasks can be specified entirely via natural-language descriptions and a handful of in-context demonstrations — no fine-tuning required. GPT-3 demonstrated that **few-shot performance scales faster than zero-shot performance** as the model grows, validating the meta-learning hypothesis.

This made the **OpenAI API** (June 2020) and eventually the entire prompt-engineering paradigm viable. ChatGPT (Nov 2022) is built on a fine-tuned GPT-3.5.

## Lineage

| Predecessor | Successor |
|---|---|
| [[gpt-2]] (1.5B) | [[gpt-4]] (undisclosed, multimodal, instruction-tuned) |

## Results (highlights)

- Few-shot SOTA or competitive on translation (especially X→En), Cloze, LAMBADA, common-sense reasoning, [[squad|SQuAD]].
- **Emergent 3-digit arithmetic** around 13B parameters — a famous example of capability emergence with scale.
- Human evaluators cannot reliably distinguish GPT-3's generated news articles from real ones.
- Struggles on natural language inference and bidirectional / inverse-prompt tasks.

## Release

GPT-3 was released via the **OpenAI API** (June 2020) rather than open weights — a deliberate departure from the open releases of GPT-1 and (eventually) GPT-2. This established the closed-weights-via-API distribution model that defined the subsequent commercial LLM era.

## References

- [[gpt-3-brown-2020]]
