---
title: Gemini 1.5
type: entity
created: 2026-10-06
updated: 2026-10-06
sources:
  - gemini-1-5-google-2024.md
family: gemini
release_date: 2024-02-15
status: draft
importance: medium
tags:
  - model
---

# Gemini 1.5

[[google|Google]] / [[deepmind|Google DeepMind]]'s second-generation Gemini model, announced **2024-02-15**. The first release was **Gemini 1.5 Pro**, "a mid-size multimodal model" said to perform at about the level of Gemini 1.0 Ultra.

## Headline claims (vendor)

- **Architecture:** a Transformer with **[[mixture-of-experts|MoE]]**.
- **Context:**
  - **128K** standard;
  - **1M tokens** in private preview;
  - **10M** tested in research;
  - Gemini 1.0 had 32K.
- **[[needle-in-a-haystack|Needle in a haystack]]:** 99% recall up to 1M tokens.
- **Long-context in-context learning:** translates Kalamang from a grammar book supplied in the prompt.

## Lineage

- **Predecessor:** Gemini 1.0 (December 2023), mentioned in the post.
- **Changed:** MoE, and a long-context jump of roughly 30× over 1.0.
- **Significance for the wiki:** the first frontier model to make million-token context a product, which made "long context vs [[retrieval-augmented-generation|RAG]]" a live question. See [[long-context-llm]].
- **Caveat:** the [[context-rot]] work cautions that one-needle recall is not the same as using the whole context.

## References

- [[gemini-1-5-google-2024]]
