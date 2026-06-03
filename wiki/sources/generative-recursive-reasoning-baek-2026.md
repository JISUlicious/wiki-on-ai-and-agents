---
title: "Generative Recursive Reasoning"
type: source
created: 2026-06-03
updated: 2026-06-03
sources:
  - generative-recursive-reasoning-baek-2026.pdf
arxiv_id: "2605.19376"
authors:
  - Junyeob Baek
  - Mingyu Jo
  - Minsu Kim
  - Mengye Ren
  - Sungjin Ahn
  - Yoshua Bengio
first_author: Junyeob Baek
year: 2026
tags: [2026]
status: complete
importance: medium
---

# Generative Recursive Reasoning

*Surfaced via the alphaXiv / NLP-newsletter digest.*

## Summary

GRAM (Generative Recursive reAsoning Models) from KAIST, Mila, and NYU turns **recursive latent reasoning** into probabilistic multi-trajectory computation. Recursive Reasoning Models (RRMs) implement extended computation by iteratively refining a persistent latent state with shared transition functions, decoupling reasoning depth from parameter scale and output length — but existing RRMs are deterministic, following a single latent trajectory to a single prediction. GRAM instead models reasoning as a *stochastic* latent trajectory, enabling multiple hypotheses, alternative solution strategies, and inference-time scaling along two axes: recursive depth and parallel trajectory sampling. The result is a latent-variable generative model supporting conditional reasoning p(y|x) and, with fixed/absent inputs, unconditional generation p(x). It is trained with amortized variational inference.

## Key points

- Reframes deterministic recursive reasoning as a probabilistic, multi-trajectory latent process.
- Stochastic latent trajectories yield multiple hypotheses and multi-solution coverage (e.g., valid y1 and y2).
- Inference-time scaling along two axes: recursive depth and parallel trajectory sampling.
- Unifies conditional reasoning p(y|x) and unconditional generation p(x) in one latent-variable model.
- Trained with amortized variational inference; shared transition functions decouple depth from model size and output length.
- Improves over deterministic recurrent/recursive baselines on structured reasoning and multi-solution constraint-satisfaction tasks.

## Concepts & entities

- [[latent-reasoning]]
- [[looped-transformer]]
- [[chain-of-thought]]
- [[recurrent-depth-transformer]]

## References

- [arXiv:2605.19376](https://arxiv.org/abs/2605.19376)
- Project page: https://ahn-ml.github.io/gram-website
