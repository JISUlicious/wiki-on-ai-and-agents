---
title: "Can Looped Transformers Learn to Implement Multi-step GD for In-context Learning?"
type: source
created: 2026-05-20
updated: 2026-05-20
sources:
  - looped-transformers-multi-step-gd-gatmiry-2024.md
arxiv_id: "2410.08292"
authors:
  - Khashayar Gatmiry
  - "[[nikunj-saunshi]]"
  - Sashank Reddi
  - Stefanie Jegelka
  - Sanjiv Kumar
first_author: Khashayar Gatmiry
year: 2024
introduces:
  - "[[looped-transformer]]"
tags:
  - 2024
status: complete
importance: medium
---

# Can Looped Transformers Learn to Implement Multi-step Gradient Descent for In-context Learning?

[arXiv:2410.08292](https://arxiv.org/abs/2410.08292)

## Summary

Gatmiry et al. provide the first convergence-theoretic analysis of a multi-layer (looped) transformer trained on in-context linear regression. For a linear looped transformer — a single linear self-attention block whose weights are shared and applied $L$ times — they prove that the **global minimizer of the population squared-error loss exactly implements $L$ steps of preconditioned gradient descent**, with preconditioner $A_\text{opt}$ provably close (in spectral order) to the inverse population covariance $\Sigma^{*-1}$, and with the auxiliary parameter $u_\text{opt} = 0$. This sharpens prior expressivity results (which only showed that *some* setting of weights can simulate gradient descent) into a *learnability* result about what training actually finds.

To go from "global minimizer = preconditioned GD" to "training converges to it," the authors prove a novel **gradient dominance condition** with power $(2L-1)/L$: $\|\nabla_A L(A,0)\|^2 \geq \tfrac{1}{16} L(A,0)^{(2L-1)/L}$ whenever the loss is not already tiny. Despite the non-convex landscape induced by weight sharing, this gradient-dominance bound is enough to push gradient flow to a small sub-optimality gap, and they further translate that gap into spectral closeness in parameter space. This is the first such convergence result beyond a one-layer model in this Gaussian linear-regression ICL setting, and serves as the theoretical companion to [[looped-transformer|Yang et al. 2023]]'s empirical observation that looped transformers learn iterative-algorithm-like solvers.

## Key Points

- **Population-loss minimizer = multi-step preconditioned GD.** Theorem 3.2: under Gaussian data $x_i \sim N(0, \Sigma^*)$ and $w^* \sim N(0, \Sigma^{*-1})$, any global minimizer $(A_\text{opt}, u_\text{opt})$ of $L(A,u) = \mathbb{E}[(\mathrm{TF}_L(Z_0) - y_q)^2]$ satisfies $u_\text{opt} = 0$ and $(1-c)\Sigma^{*-1} \preceq A_\text{opt} \preceq (1+c)\Sigma^{*-1}$ with $c = 8\delta d^{1/(2L)}$ and $\delta = (8Ld/\sqrt n)^{1/(2L)} \to 0$ as $n \to \infty$. So the optimal $L$-loop model is precisely $L$ steps of GD preconditioned by (essentially) the inverse population covariance.
- **The "extra" parameter $u$ must vanish at optimum.** This was an open question after [Ahn et al. 2023]; here $u_\text{opt} = 0$ falls out of the characterization, which is what *forces* the model to behave exactly like preconditioned GD rather than a more general affine update.
- **Gradient-dominance condition (Theorem 3.4).** A novel inequality of the form $\|\nabla L\|^2 \gtrsim L^{(2L-1)/L}$ — stronger than Polyak-Łojasiewicz when $L$ is large — drives the convergence proof. The authors expect this proof technique to generalize to other first-order iterative algorithms (e.g., SGD with single-instance gradient estimates, à la De Sa et al. 2022).
- **Convergence of gradient flow (Theorem 3.3).** For $\Sigma^* = I$, the gradient flow $\dot A(t) = -\nabla_A L(A(t), 0)$ reaches loss $\leq \xi$ in time $t \gtrsim (1/\xi)^{(L-1)/(2L-1)} \cdot (16L/(L-1))^{(L-1)/L}$, and the iterate $A(t)$ is spectrally close to $A_\text{opt}$ — despite the loss landscape being highly non-convex due to weight sharing.
- **Out-of-distribution generalization (Theorem 4.5).** Because the learned model implements *preconditioned GD as an algorithm*, it generalizes across regression instances drawn from different covariances at test time; the experiments confirm this, including looping at test time beyond the training loop count and continuing to decrease loss.
- **Theoretical companion to empirical work.** [[looped-transformer|Yang et al. 2023]] showed empirically that looped transformers in-context-learn linear regression with far fewer parameters than vanilla transformers; this paper supplies the matching optimization-theory story for *why* training converges to an iterative-algorithm solution.
- **Synthetic experiments (Section 5).** $d=10$, $\Sigma^* = I$. Looped models converge to near-zero loss; more loops give lower loss; an $L$-loop, 1-layer model roughly matches an $L$-layer non-looped model. $A(t)$ converges to identity even when $n < d$ (beyond what the theorem covers). Test-time looping beyond training depth keeps reducing loss on both in-distribution and OOD covariances.
- **Setup specifics.** Linear self-attention with the [Ahn et al. 2023] / [Von Oswald et al. 2023] parameterization $\mathrm{Attn}(Z; Q,P) = PZM(Z^\top Q Z)$; only the top-left $d \times d$ block of $Q$ (called $A$) and the last row of $P$ (called $u$) are trainable; the rest are zeroed out, matching the structure that expressivity proofs require for one step of preconditioned GD.

## Entities

- [[khashayar-gatmiry]] (MIT)
- [[nikunj-saunshi]] (Google Research)
- [[sashank-reddi]] (Google Research)
- [[stefanie-jegelka]] (MIT)
- [[sanjiv-kumar]] (Google Research)
- [[mit]]
- [[google-research]]

## Concepts

- [[looped-transformer]] — the architecture under study; a single attention block applied $L$ times with shared weights.
- [[gradient-descent-as-icl]] — the hypothesis that transformers implement gradient descent in-context; this paper turns the hypothesis into a theorem for the looped, linear, Gaussian setting.
- [[in-context-learning]] — the broader phenomenon being explained.
- [[optimization-theory]] — gradient flow analysis on a non-convex landscape via a Polyak-Łojasiewicz-style gradient-dominance bound.
- [[linear-self-attention]] — the attention variant used (no softmax), which makes the closed-form analysis tractable.
- [[preconditioned-gradient-descent]] — the iterative algorithm the model converges to implementing.

## Quotes

> "The global minimizer of the population training loss implements multi-step preconditioned gradient descent, with a preconditioner that adapts to the data distribution."

> "We show a fast convergence for gradient flow on the regression loss, despite the non-convexity of the landscape, by proving a novel gradient dominance condition. To our knowledge, this is the first theoretical analysis for multi-layer Transformer in this setting."

> "A value of $u_\text{opt} = 0$ implies that the optimal looped Transformer exactly implements $L$ steps of preconditioned gradient descent, with preconditioner $A_\text{opt}$."

> "For looped models trained with just 2 (or 5) loops, evaluating them with arbitrarily large number of loops during test time continues to decrease the loss even further for ID and OOD data. This suggests that the trained looped models are indeed learning a good iterative algorithm."

## References

- Ahn, Cheng, Daneshmand, Sra. *Transformers learn to implement preconditioned gradient descent for in-context learning.* arXiv:2306.00297, 2023. — Closest predecessor; handled the one-layer case and showed multi-layer expressivity at stationary points.
- Von Oswald et al. *Transformers learn in-context by gradient descent.* 2023. — Expressivity construction this paper builds on.
- Akyürek, Schuurmans, Andreas, Ma, Zhou. *What learning algorithm is in-context learning?* arXiv:2211.15661, 2022.
- Zhang, Frei, Bartlett. *Trained Transformers learn linear models in-context.* 2023. — Companion one-layer analysis.
- Garg, Tsipras, Liang, Valiant. *What can Transformers learn in-context?* 2022. — Empirical ICL-for-regression setup that this theoretical line of work formalizes.
- Yang, Lee, Nichani, Lee. *Looped Transformers are Better at Learning Learning Algorithms.* 2023. — Empirical motivation; this paper is its theoretical companion.
- Giannou et al. *Looped Transformers as programmable computers.* 2023.
- De Sa, Kale, Lee, Sekhari, Sridharan. *From gradient flow on population loss to learning with SGD.* 2022. — Cited as a setting where the gradient-dominance proof technique should extend.
- Vaswani et al. *Attention Is All You Need.* 2017.
