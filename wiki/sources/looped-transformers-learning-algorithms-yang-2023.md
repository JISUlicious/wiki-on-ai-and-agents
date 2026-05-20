---
title: "Looped Transformers are Better at Learning Learning Algorithms"
type: source
created: 2026-05-20
updated: 2026-05-20
sources:
  - looped-transformers-learning-algorithms-yang-2023.md
arxiv_id: "2311.12424"
venue: ICLR 2024
authors:
  - Liu Yang
  - "[[kangwook-lee]]"
  - Robert Nowak
  - "[[dimitris-papailiopoulos]]"
first_author: Liu Yang
year: 2023
introduces:
  - "[[looped-transformer]]"
tags:
  - 2023
status: complete
importance: high
---

# Looped Transformers are Better at Learning Learning Algorithms

[arXiv:2311.12424](https://arxiv.org/abs/2311.12424) — Yang, Lee, Nowak, Papailiopoulos. ICLR 2024. University of Wisconsin–Madison.

## Summary

This paper provides the first strong empirical evidence that a **learned** [[looped-transformer]] can capture the iterative structure of [[in-context-learning]] algorithms when trained from scratch on data-fitting problems. The authors take the standard decoder-only architecture from Garg et al. (2022) and replace it with a single-layer transformer block that is applied repeatedly with input injection (`Y_{t+1} = M(Y_t + P)`), trained end-to-end with a truncated-backprop-through-time loss over the last `T` of `b` loop iterations. On in-context linear regression with `d=20`, `k=40`, the looped model matches a 12-layer standard transformer and nearly tracks the ordinary-least-squares solver while using **0.79M vs. 9.48M parameters (~1/12, i.e. <10%)**.

The work bridges the gap between Giannou et al.'s (2023) hand-constructed theory showing that looped transformers can in principle emulate iterative algorithms and the practical question of whether such structure is **learnable** via gradient descent. Beyond linear regression, the looped transformer matches or beats the standard transformer on sparse linear functions, decision trees, and 2-layer ReLU networks — and a model-probing analysis (Akyürek et al.) shows that, unlike the standard transformer which decodes `X^T y` and `w_OLS` around layer 10 then drifts, the looped transformer *steadily refines* these target representations across loop iterations, consistent with executing an iterative optimizer in latent space.

## Key Points

- **Looped architecture with input injection.** Replace L=12 stacked blocks with one block applied b times; at each step inject the prompt: `Y_{t+1} = M(Y_t + P)`. Without input injection (plain weight-tying), the model diverges past the trained iteration count; with it, the model converges to a stable fixed point that extrapolates beyond training depth.
- **Truncated loss over a window `[b-T, b]`.** Backprop only through the last `T` iterations of `b` total, mirroring truncated BPTT. The authors settle on `b=20`, `T=15` for linear regression after a sweep over `b ∈ {12,20,30,40,50}`, `T ∈ {5,10,15}`.
- **Matches standard transformer with <10% parameters.** GPT-2 backbone with `D=256`, `h=8`: unlooped L=12 has 9.48M params; looped L=1 has 0.79M. In-distribution performance on linear regression is essentially tied with OLS and the 12-layer baseline.
- **Effective depth exceeds raw layer count.** A 1-layer looped model needs ~20 iterations to match an 8-layer standard transformer, indicating the looped and standard models learn *different* per-step representations.
- **Sample efficiency.** Because of weight sharing across loops (a form of [[parameter-sharing]]), the looped model fits the target function class with fewer distinct prompts during training than the standard transformer.
- **Inductive bias toward simpler solutions.** OOD evaluation shows the looped model handles skewed-covariance inputs and noisy linear regression better (reduced double-descent peak), but underperforms on scaling shifts (`x = 3z`) — consistent with a simplicity bias rather than learning the true closed-form algorithm.
- **Generalizes beyond linear regression.** Looped models match or beat standard transformers on sparse linear functions, depth-4 decision trees, and 2-layer ReLU networks. Required `(b, T)` correlate with task complexity but not monotonically (the 2-layer NN needs *fewer* iterations than linear regression).
- **Probing evidence for iterative refinement.** A 2-layer MLP probe trained to decode `X^T y` and `w_OLS` from intermediate activations shows the looped transformer monotonically improves these reconstructions across loop iterations, while the unlooped transformer peaks mid-stack and then transforms representations into something else.

## Entities

- [[uw-madison]] — institutional affiliation of all four authors.
- [[kangwook-lee]] — co-author; Wisconsin faculty working on ICL and looped/recurrent transformer theory.
- [[dimitris-papailiopoulos]] — co-author; Wisconsin faculty working on transformer learnability.
- Robert Nowak — co-author, Wisconsin (statistical learning theory). Liu Yang — first author, then PhD student at Wisconsin.

## Concepts

- [[looped-transformer]] — the architecture this paper introduces and empirically validates for learning iterative ICL algorithms.
- [[in-context-learning]] — the regression-style ICL setup (Garg et al. 2022) used as the testbed.
- [[gradient-descent-as-icl]] — the mechanistic hypothesis (von Oswald et al., Akyürek et al.) that motivates iterating one transformer block: each loop step ~ one gradient step.
- [[parameter-sharing]] — how the looped model achieves <10% parameter count vs. the unlooped baseline.
- [[transformer-architecture]] — base model is a GPT-2 decoder; only the depth/loop structure changes.

## Notable Quotes

> "Experimental results suggest that the looped transformer achieves performance comparable to the standard transformer in solving various data-fitting problems, while utilizing less than 10% of the parameter count."

> "Can looped transformers emulate iterative learning algorithms more efficiently than standard, non-recursive transformers? Within the specific function classes we tested, the answer is positive."

> "Contrasting with the standard transformer, which decodes the target probe optimally around the 10-th layer, the looped transformer steadily refines its representation, improving the decoding of the target probe with each subsequent loop iteration."

> "Rather than mastering the true algorithm applied to the linear regression problem irrespective of shifts in the input distribution, the looped transformer exhibits an inductive bias favoring simpler solutions when compared to the unlooped transformer."

## References

- Garg, Tsipras, Liang, Valiant (2022) — ICL of function classes, the experimental setup this paper inherits.
- Giannou et al. (2023) — hand-constructed looped transformers as universal iterative solvers; theory that this paper makes learnable.
- von Oswald et al. (2022); Akyürek et al. (2022); Ahn et al. (2023); Mahankali et al. (2023); Zhang et al. (2023a) — gradient-descent-as-ICL line of work.
- Dehghani et al. (2018) — Universal Transformer; earlier weight-tying-across-depth proposal.
- Bai et al. (2019, 2021, 2022) — Deep Equilibrium / implicit models; closest prior on infinite-depth fixed-point networks.
- Code: <https://github.com/Leiay/looped_transformer>
