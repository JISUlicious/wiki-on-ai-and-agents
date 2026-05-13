---
title: Hallucination
type: concept
created: 2026-05-13
updated: 2026-05-13
sources:
  - rag-lewis-2020.md
status: draft
importance: high
---

# Hallucination

In the LLM literature, **hallucination** refers to a model generating text that is fluent and plausible-sounding but **factually incorrect** or unsupported by any input or training source. The model "makes things up" with confidence — fabricating names, dates, citations, code APIs, study results, etc.

## Why it happens

Pre-trained LLMs are trained to maximize next-token likelihood, not to be truthful. They learn associations and patterns that produce plausible continuations, but plausibility ≠ truth. When the model is uncertain or the prompt enters a domain underrepresented in training, the highest-likelihood continuation can be a confident fabrication rather than a hedge or refusal.

## Mitigations

- **[[retrieval-augmented-generation|RAG]]**: ground answers in retrieved passages from an authoritative corpus, with citations.
- **Tool use / verification** (e.g. [[react|ReAct]]): let the model query external systems before answering.
- **[[rlhf|RLHF]] / preference tuning**: train the model to prefer hedged, accurate answers over confident wrong ones.
- **Self-consistency / chain-of-thought** ([[chain-of-thought-prompting|CoT]]): make the reasoning explicit so errors are catchable.
- **Self-critique loops** (e.g. [[reflexion|Reflexion]]): let the model evaluate and revise its own outputs.

Hallucination is not fully solved by any current method and remains one of the central reliability problems for production LLM systems.

## References

- [[rag-lewis-2020]]
