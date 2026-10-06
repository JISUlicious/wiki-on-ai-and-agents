---
title: Gemini 1.5
type: entity
created: 2026-10-06
updated: 2026-10-07
sources:
  - gemini-1-5-google-2024.md
  - gemini-1-5-technical-report-2024.md
family: gemini
release_date: 2024-02-15
status: complete
importance: medium
tags:
  - model
---

# Gemini 1.5

[[google|Google]] / [[deepmind|Google DeepMind]]'s second-generation Gemini family. It was announced **2024-02-15** with **Gemini 1.5 Pro**. An updated Pro and the smaller **Gemini 1.5 Flash** followed in the technical report ([arXiv:2403.05530](https://arxiv.org/abs/2403.05530), Mar 2024; the v5 read here is dated 2024-12-16).

## Models

| | Gemini 1.5 Pro | Gemini 1.5 Flash |
|---|---|---|
| Architecture | **Sparse [[mixture-of-experts\|MoE]] Transformer** | Dense Transformer decoder; attention and feed-forward computed in parallel |
| Training | Multiple 4096-chip TPUv4 pods | **Online-distilled** from Pro |
| Context | 128K standard; 1M in private preview at launch; 2M+ by the updated report; 10M tested in research | Same 2M+ context |
| Positioning | Matches or beats 1.0 Ultra with "significantly less compute to train" | Similar to 1.0 Ultra on several benchmarks; fastest output among the APIs compared |

## Long-context results

- **Single needle** ([[needle-in-a-haystack]]):
  - text recall is 100% up to 530K tokens, **99.7% at 1M** and **99.2% at 10M**;
  - Claude 2.1 scored 98% at 200K;
  - recall is near-perfect on hours of video and days of audio as well.
- **100 needles at once:** about **70% recall to 128K**, and **over 60% to 1M**. GPT-4 Turbo averaged about 50% at its 128K limit.
- **MRCR** (picking out one of many similar earlier outputs): Pro overtakes GPT-4 Turbo and Claude 3 Opus beyond about 32K tokens.
- **Next-token loss** keeps falling up to at least 10M tokens of context.
- **In-context learning at book scale:** translates **Kalamang** from a 500-page grammar, a dictionary and about 400 sentences given in the prompt, at the level of a human learning from the same materials.

## Lineage

- **Predecessor:** Gemini 1.0 (1.0 Ultra introduced December 2023): 32K context.
- **What changed:**
  - context grew about 30× at release, to 1M, and to 10M in research;
  - MoE for training and serving efficiency;
  - a distilled small sibling (Flash);
  - native multimodal long context (video and audio).
- **What it changed for the field:** million-token context became a product, which turned "long context vs [[retrieval-augmented-generation|RAG]]" into a live engineering choice ([[long-context-llm]]).
- **What it also showed:** single-needle recall is not effective use of the whole context. The multi-needle and MRCR drops are part of the evidence behind [[context-rot]] and [[context-engineering]].

## References

- [[gemini-1-5-google-2024]]
- [[gemini-1-5-technical-report-2024]]
