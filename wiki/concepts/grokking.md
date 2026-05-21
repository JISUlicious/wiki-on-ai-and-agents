---
title: Grokking
type: concept
created: 2026-05-21
updated: 2026-05-21
sources:
  - loop-think-generalize-2026.md
status: draft
importance: medium
tags:
  - 2022
---

**Grokking** is the phenomenon, first highlighted by Power et al. (2022), where a neural network keeps training long after it has perfectly fit the training set and then *suddenly* generalizes to held-out data. Loop-Think-Generalize (2026) reports analogous late-phase generalization dynamics in [[recurrent-depth-transformer]] training, suggesting that depth-recursion and grokking share structural drivers (representation reorganization under continued optimization pressure).

- Training loss saturates early; test loss collapses much later
- Often associated with weight-decay-driven representation cleanup
- Observed in [[recurrent-depth-transformer]] / latent-reasoning training
- Open question: relationship to phase transitions and [[emergent-abilities]]

Related: [[recurrent-depth-transformer]], [[latent-reasoning]], [[emergent-abilities]]

## References
- [[loop-think-generalize-2026]]
