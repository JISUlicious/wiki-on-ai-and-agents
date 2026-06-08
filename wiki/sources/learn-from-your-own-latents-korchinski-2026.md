---
title: "Learn from your own latents and not from tokens: A sample-complexity theory"
type: source
created: 2026-06-08
updated: 2026-06-08
sources:
  - learn-from-your-own-latents-korchinski-2026.md
arxiv_id: "2605.27734"
authors:
  - Daniel J. Korchinski
  - Alessandro Favero
  - Matthieu Wyart
first_author: Daniel J. Korchinski
year: 2026
tags: [2026]
status: complete
importance: medium
---

# Learn from your own latents and not from tokens: A sample-complexity theory

Surfaced via the NLP newsletter (Top AI Papers of the Week, 2026-05-31–06-07).

## Summary

Generative models (diffusion, LLMs) reach strong performance but need orders of magnitude more training data than biological learners. An alternative paradigm trains networks to predict their *own latent representations* of related views or masked regions, as in data2vec and JEPA (echoing predictive-coding accounts of cortex). This paper provides a **sample-complexity theory** for why such latent prediction is more data-efficient than token prediction.

Using a tractable **probabilistic context-free grammar** that generates visible tokens by recursively applying production rules down a tree of hidden symbols of depth L, the authors prove a sharp separation: supervised or token-level self-supervised learning need a number of samples **exponential in L** to recover the latent tree, whereas latent prediction needs samples **constant in L** (up to log factors). They confirm the bound with a hierarchical clustering algorithm, an end-to-end predictor-clusterer network, and the first sample-complexity analysis of data2vec — which they show implicitly performs hierarchical latent prediction. The upshot: explicit hierarchical stacking such as H-JEPA is largely redundant.

## Key points

- Formalizes JEPA / data2vec-style world-model objectives as predicting one's own latents rather than raw tokens.
- Data model is a probabilistic context-free grammar with depth-L hidden tree, capturing compositional structure of language and images.
- Token-level / supervised SSL: samples needed grow exponentially in tree depth L to recover the latent tree.
- Latent prediction: samples needed are constant in L (up to logarithmic factors) — the core efficiency result.
- data2vec is shown to *implicitly* perform hierarchical latent prediction, so explicit multi-scale stacking (e.g., H-JEPA) is largely redundant.
- Purely theoretical (cs.LG); authors at EPFL, Cambridge, and Johns Hopkins.

## Concepts & entities

- [[latent-reasoning]] — predicting internal latent representations rather than surface tokens.
- [[scaling-laws]] — sample-complexity / data-efficiency separations.
- JEPA, data2vec, world models, H-JEPA — discussed in text; no dedicated wiki pages.

## References

- [arXiv:2605.27734](https://arxiv.org/abs/2605.27734)
