---
title: PaLM
type: entity
created: 2026-05-13
updated: 2026-05-13
sources:
  - chain-of-thought-wei-2022.md
family: palm
predecessor: null
successors: []
parameters: "540B (largest)"
release_date: 2022-04-04
status: draft
importance: medium
tags:
  - model
---

# PaLM

PaLM (Pathways Language Model; Chowdhery et al. 2022, arXiv 2204.02311) is Google's 540-billion-parameter [[decoder-only-transformer|decoder-only Transformer]] LM. Trained on Pathways infrastructure on ~780B tokens. Featured prominently in the [[chain-of-thought-wei-2022|chain-of-thought prompting paper]] as the model whose 540B scale was sufficient to dramatically benefit from CoT.

PaLM was succeeded by PaLM 2 (2023) and then by the Gemini family (2023–) under [[deepmind|Google DeepMind]].

## References

- [[chain-of-thought-wei-2022]]
