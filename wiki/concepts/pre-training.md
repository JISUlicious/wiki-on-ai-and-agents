---
title: Pre-training
type: concept
created: 2026-05-13
updated: 2026-05-13
sources:
  - bert-devlin-2018.md
status: complete
importance: high
---

# Pre-training

Pre-training is the practice of training a model on a large, generic, self-supervised objective before adapting it to a specific downstream task. In modern NLP, pre-training has become the dominant paradigm: a single expensive pre-training run produces general-purpose representations or a general-purpose generative model, which is then cheaply specialized to many downstream uses via [[fine-tuning]], prompting, or [[in-context-learning|in-context learning]].

## Common pre-training objectives

- **Autoregressive language modeling** (next-token prediction): used by [[gpt-1]], [[gpt-2]], [[gpt-3]], [[gpt-4]] and most modern decoder-only LLMs.
- **[[masked-language-modeling|Masked language modeling]] (MLM)**: used by [[bert]] and its descendants.
- **Span corruption / denoising**: used by T5 and BART (mask out contiguous spans, predict the original).
- **Image patch masking**: e.g., MAE for vision (related cousin of [[vit]]).

## Why it works

A large, diverse, unlabeled corpus offers far more supervisory signal than any annotated dataset can. Predicting the next token (or the masked token) forces the model to learn syntax, semantics, world knowledge, reasoning shortcuts, and a great deal else — all of which transfer to downstream tasks. The amount of useful transfer scales remarkably smoothly with model size and data (cf. [[scaling-laws]]; [[gpt-3-brown-2020|GPT-3]] was a stark demonstration).

## Pre-training vs. fine-tuning vs. in-context

- **Pre-training**: expensive (often the dominant compute cost), produces a base model.
- **[[fine-tuning|Fine-tuning]]**: cheap, adapts the base model's parameters to a specific task or domain.
- **[[in-context-learning|In-context learning]]**: free at training time — the base model itself is repurposed via prompting, with no weight updates. Discovered to work surprisingly well at scale in [[gpt-3-brown-2020|GPT-3]].

## References

- [[bert-devlin-2018]]
