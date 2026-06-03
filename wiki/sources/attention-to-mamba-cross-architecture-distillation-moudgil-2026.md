---
title: "Attention to Mamba: A Recipe for Cross-Architecture Distillation"
type: source
created: 2026-06-03
updated: 2026-06-03
sources:
  - attention-to-mamba-cross-architecture-distillation-moudgil-2026.md
arxiv_id: "2604.14191"
authors:
  - Abhinav Moudgil
  - Ningyuan Huang
  - Eeshan Gunesh Dhekane
  - Pau Rodríguez
  - Luca Zappella
  - Federico Danieli
first_author: Abhinav Moudgil
year: 2026
tags: [2026]
status: complete
importance: medium
---

# Attention to Mamba: A Recipe for Cross-Architecture Distillation

*Surfaced via the alphaXiv / NLP-newsletter digest.*

## Summary

This Apple/MILA/Flatiron work presents a recipe for distilling a pretrained Attention-based Transformer into a **Mamba-like State Space Model (SSM)** without retaining any Attention blocks. SSMs like Mamba offer reduced memory and higher generation throughput, but naive Transformer-to-Mamba distillation fails to preserve teacher performance (prior work resorts to hybrid Attention+SSM blocks). The key claim is that a **principled initialization** enables a pure-Mamba recipe. They use a two-stage approach: first distill the Transformer into a linearized Attention (via a kernel-trick adaptation, "Hedgehog"-style), then distill that into an adapted Mamba. The distilled Mamba preserves Pythia-1B performance (perplexity 14.11 vs. teacher's 13.86).

## Key points

- Motivation: leverage the mature ecosystem of pretrained Transformers to bootstrap SSMs instead of training them from scratch.
- Problem: naive Transformer→Mamba distillation does not preserve teacher quality; common workaround is hybrid Attention+SSM, which the authors avoid.
- **Two-stage distillation**: (1) Transformer → linearized Attention using a kernel-trick adaptation; (2) linearized Attention → adapted Mamba with no Attention blocks.
- Principled initialization of Mamba is the core ingredient enabling a better pure-SSM recipe.
- Result: distilled Mamba matches Pythia-1B teacher (14.11 vs 13.86 perplexity) on downstream tasks.
- Thorough ablations at 1B scale / 10B tokens: sequence-mixer architecture, model-size and total-token scaling, and stage-wise token-allocation sensitivity.

## Concepts & entities

- [[transformer-architecture]]
- [[attention-mechanism]]
- [[self-attention]]
- [[kv-cache]]
- [[long-context-llm]]

## References

- [arXiv:2604.14191](https://arxiv.org/abs/2604.14191)
- Source text: `sources/attention-to-mamba-cross-architecture-distillation-moudgil-2026.md`
