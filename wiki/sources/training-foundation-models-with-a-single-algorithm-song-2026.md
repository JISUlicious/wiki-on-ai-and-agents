---
title: "Training Foundation Models with a Single Algorithm"
type: source
created: 2026-11-10
updated: 2026-11-10
sources:
  - training-foundation-models-with-a-single-algorithm-song-2026.md
arxiv_id: "2610.40564"
authors:
  - Yuda Song
  - Alvin Zhang
  - Fahim Tajwar
  - Daman Arora
  - Jeff Schneider
  - Daniel Khashabi
  - Ruslan Salakhutdinov
  - J. Andrew Bagnell
  - Aarti Singh
  - Andrea Zanette
year: 2026
venue: arXiv preprint
tags: [2026, paper]
status: complete
importance: high
---

# Training Foundation Models with a Single Algorithm

**Source:** Yuda Song, Alvin Zhang, Fahim Tajwar et al., Andrea Zanette ([[cmu|CMU]], Johns Hopkins, Aurora Innovation) — [arXiv:2610.40564](https://arxiv.org/abs/2610.40564), cs.LG, v1 29 Oct 2026. 67 pages. Surfaced via the NLP newsletter (2026-11-01).

## Summary

Foundation models are trained in stages ([[pre-training]], midtraining, [[supervised-fine-tuning|SFT]], [[reinforcement-learning|RL]]), each with its own algorithm and infrastructure. This paper argues the algorithmic split is not fundamental. The first three stages already share one maximum-likelihood loss and differ only in data. RL can be brought under the same loss through **MaxRL** (Tajwar et al., 2026): with binary rewards, the gradient of log success probability equals likelihood fitting to the policy's own *success-conditioned* distribution. So an RL update is just SFT on the model's correct samples, weighted by 1/p̂ (the inverse sampled pass rate).

**Unified training** therefore uses one weighted token-level cross-entropy over a mixture of sources: pretraining text, midtraining corpora, demonstrations, and reward-weighted RL generations. The design question shifts from "which stage and recipe" to "which data, in what proportions, and when". Conventional staged training becomes the special case where mixture weights jump abruptly. The system has a shared data buffer and one trainer, with generators appending RL samples asynchronously.

## Key points

- **Common data format.** Every example is (tokens, per-token weights). Pretraining has unit weights; SFT masks the prompt; RL masks the prompt and gives correct completions weight A = 1/p̂ and incorrect ones zero. With N = 1 sample it reduces to REINFORCE.
- **Controlled study** (124M model from scratch, 20B pretraining + 510M midtraining tokens, Reasoning-Gym-style puzzles). The best unified mixtures beat the best staged runs on accuracy and on generalization to larger unseen puzzles. Unified training also reuses midtraining data better (2–4 epochs help), while extending staged midtraining hurts.
- **Compute.** Staged training wins on accuracy at intermediate budgets and on coverage (pass@16) in-distribution. Unified wins at the smallest and larger budgets, and on generalization at almost all budgets.
- **Retention.** Mixing midtraining data into pretraining keeps held-out pretraining loss and general-benchmark scores near the pretrain-only model, while staged midtraining degrades them. This reduces forgetting without a KL penalty: no tested KL strength reproduced the effect without hurting math.
- **Real data** (0.76B model, Math12K RL + 0–5% heterogeneous midtraining mix). Math accuracy stays flat, coverage and SciQ improve, and coding pass@1 falls as the mix grows. The schedule (constant, annealed, half-annealed) trades these off.
- **[[continual-learning|Continual learning]].** After math training, every unified model reaches higher final coding accuracy under identical coding RL than the staged model, despite starting lower.
- **MaxRL vs REINFORCE** in the same framework: REINFORCE gives higher math accuracy and retention; MaxRL gives higher coverage.
- **Limits:** small scale, mostly single-task RL, no predictive [[scaling-laws|scaling laws]] for mixtures, and downstream adaptation still needs a stage.

## Concepts & entities

- [[pre-training]] · [[supervised-fine-tuning]] · [[reinforcement-learning]] · [[outcome-based-reward]] — all recast as likelihood fitting on an evolving data mixture.
- [[grpo]] · [[ppo]] — the advantage-based RL objectives this sidesteps; staged baselines use MaxRL for a clean comparison.
- [[continual-learning]] — retention and better starting points for later RL.
- [[on-policy-distillation]] · [[knowledge-distillation]] — related "fit a target distribution" views of post-training.
- [[cmu]]

## References

- [arXiv:2610.40564](https://arxiv.org/abs/2610.40564)
