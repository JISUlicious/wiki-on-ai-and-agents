---
title: Inference-Time Compute
type: concept
created: 2026-05-21
updated: 2026-05-21
sources:
  - latent-thoughts-looped-transformers-saunshi-2025.md
  - cot-or-loop-xu-2025.md
status: draft
importance: high
tags:
  - 2024
---

**Inference-time compute** is the 2024–2026 paradigm of spending *more* compute *at decode time* rather than only at pre-training. Two main flavors have emerged: **token-recursion**, where the model emits more reasoning tokens (CoT, o1, [[deepseek-r1]]); and **depth-recursion**, where the model re-applies its weights at the same position ([[looped-transformer]], [[latent-reasoning]], [[recurrent-depth-transformer]]).

- Token-recursion: visible chain-of-thought tokens ([[chain-of-thought-prompting]], [[deepseek-r1]])
- Depth-recursion: latent loops over hidden states ([[looped-transformer]], [[latent-reasoning]])
- Empirical and theoretical work compares the two regimes (see [[cot-or-loop-xu-2025]])
- Closely tied to scaling agentic policies via [[agentic-rl]] and test-time search

Related: [[chain-of-thought-prompting]], [[deepseek-r1]], [[looped-transformer]], [[latent-reasoning]], [[agentic-rl]]

## References
- [[latent-thoughts-looped-transformers-saunshi-2025]]
- [[cot-or-loop-xu-2025]]
