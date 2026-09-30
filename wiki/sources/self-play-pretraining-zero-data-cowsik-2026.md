---
title: "Self-Play Pretraining with Zero Data"
type: source
created: 2026-09-30
updated: 2026-09-30
sources:
  - self-play-pretraining-zero-data-cowsik-2026.md
arxiv_id: "2609.30063"
authors:
  - Aditya Cowsik
  - Kfir Dolev
  - Michael Y. Li
  - G. Bruno De Luca
  - Nourya Cohen
  - Yoav Levine
  - Noah D. Goodman
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# Self-Play Pretraining with Zero Data

**Cowsik, Dolev, Li (equal contribution, listed alphabetically), De Luca, Cohen, Levine, Goodman.** Affiliations: an independent researcher, Tel Aviv University, [[stanford-university|Stanford]], and LAPTh/USMB. [arXiv:2609.30063](https://arxiv.org/abs/2609.30063), cs.AI, v1 24 Sep 2026.

## Summary

The paper pretrains a byte-level learner **without any natural data**. Two randomly initialized Llama-architecture transformers co-evolve:

- A **generator** writes programs for a minimal Brainf*ck-like universal Turing machine. Every generated string is made executable and produces bounded output.
- A **learner** does next-byte prediction on the programs' outputs.

The generator is trained with a GRPO-style policy gradient plus KL to a uniform Solomonoff-like prior and reward-weighted SFT replay. Its reward is a **learning-progress signal**: the absolute AdamW-preconditioned inner product between the learner's gradient on a program's output and the learner's recent parameter movement, computed with forward-mode AD. The authors tried a "hard to predict" reward first and dropped it because adding random bytes games it. The test is zero-shot BPB on held-out natural data from many modalities.

## Key points

- **Zero-shot power laws.** Compute-optimal frontiers are built over model size, checkpoint and ensemble size, and fitted with L(C) = E + A·C^−b. Exponents b (Table 2):
  - DCLM text 0.123 (literature 0.048–0.099).
  - CIFAR-10 0.066 / 0.145 (depending on byte layout).
  - Audio 0.141 / 0.260; MIDI 0.249.
  - Metamath 0.129; C 0.116; Python 0.113.
  - DNA **0.435** (literature 0.01–0.06).

  The paper calls these "broadly similar… if a bit higher" than natural-data exponents.
- **The adaptive search is what matters.** Sampling programs from the fixed universal prior over the same program space scales "substantially more slowly". Across 1.64 × 10^8 programs drawn uniformly, no recognizable mathematical family appeared except arithmetic sequences. The generator found such families far earlier (Table 1).
- **Comparison with PCFG pretraining.** PCFG pretraining wins on text and code. Self-play wins on images, music, audio and speech.
- **Emergent [[in-context-learning]].** Accuracy reaches close to 100% on reverse-string, stack and associative recall as examples are added. MAX and MIN start at 6–8% at m = 0. For SUM, the model gets the low 4 bits right after about 4 examples and the high bits by about 8. Models trained on the universal prior and on PCFGs fail most of these tasks.
- **Generator quality rises over training.** A fresh 1M-parameter learner trained on corpora from later generator checkpoints gets higher epiplexity and lower OOD BPB.
- **Use as pre-pretraining (appendix).** A 24.4M model warm-started from self-play reaches a target loss faster than one started from scratch: ESC-50 in 320M vs 496M tokens, CIFAR-10 in 421M vs 588M.
- **Proposed interpretation.** The loss is split into contingent data D_c and universal-structure data D_u. Self-play grows D_u with compute while D_c stays at zero.

## What it does not show

- Hyperparameters were chosen using validation loss on DCLM and DNA. The authors acknowledge this as "limited leakage".
- No absolute zero-shot BPB is reported against a model trained on natural data, and the models are small. The token budget is capped at 34.36B.
- By construction it cannot learn contingent facts. The authors do not present it as a replacement for natural data.

## Concepts & entities

- [[pre-training]] · [[scaling-laws]] — compute-optimal frontiers built with zero natural tokens.
- [[grpo]] · [[reinforcement-learning]] — the generator's objective.
- [[in-context-learning]] — appears without any natural data.
- [[breaking-the-token-ceiling-byte-model-distillation-marathe-2026]] — also byte-level, but uses bytes to preserve a teacher's distribution.
- [[stanford-university]].

## References

- [arXiv:2609.30063](https://arxiv.org/abs/2609.30063)
- Universal prediction lineage: Solomonoff 1964; Hutter 2000; Grau-Moya et al. 2024 (Brainf*ck-style universal data). Pre-pretraining on formal data: Hu et al. 2025; Papadimitriou & Jurafsky 2020.
- Epiplexity: Finzi et al. 2026. Scaling recipe: Kim et al. 2026b; Wen et al. 2026. Preconditioned gradient alignment: Thrush et al. 2026.
