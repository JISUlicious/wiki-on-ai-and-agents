---
title: "TMR: Text-to-Motion Retrieval Using Contrastive 3D Human Motion Synthesis"
type: source
created: 2026-10-24
updated: 2026-10-24
sources:
  - tmr-text-to-motion-retrieval-petrovich-2023.md
tags:
  - 2023
  - paper
status: complete
importance: medium
authors:
  - Mathis Petrovich
  - Michael J. Black
  - Gül Varol
introduces:
  - [[tmr]]
year: 2023
venue: ICCV 2023
arxiv_id: "2305.00976"
---

# TMR: Text-to-Motion Retrieval

[arXiv:2305.00976](https://arxiv.org/abs/2305.00976), 2023, by Petrovich (LIGM / MPI-IS), Black (MPI-IS) and Varol (LIGM). ICCV 2023 is inferred from the paper's disclosure file.

## Idea

[[tmr|TMR]] is a **joint text–motion embedding** for retrieval.
- **Built on TEMOS:** a VAE-style transformer motion encoder, and a text encoder over frozen DistilBERT features.
- **Adds a symmetric InfoNCE contrastive loss** (τ = 0.1).
- **Keeps the motion-synthesis branch.** Removing it lowers retrieval, which the authors attribute to "bag-of-words" shortcuts.
- **Filters false negatives:** about 17.3% of in-batch negatives whose texts are very similar (MPNet similarity > 0.8) are dropped from the loss.

## Results

- **HumanML3D:** text-to-motion R@1 5.68 vs TEMOS 2.12 on the full test set. The abstract's headline is median rank "54 to 19" under the protocol that counts near-duplicate texts as correct.
- **Ablations:** InfoNCE beats a margin loss (41.93 vs 34.46 R@3 on KIT), and negative filtering adds about 6 points.

## Why it matters beyond retrieval

TMR became the **text-alignment evaluator** for motion generation: retrieval precision (R@k) and FID are computed in TMR's embedding space.
- [[kimodo]] trains its own TMR on 700 hours of mocap and computes R@3 over the **whole test set** rather than the older batches of 32.
- It ships that evaluator as `TMR-SOMA-RP-v1`. Mathis Petrovich is a co-first author of Kimodo.

> [!note] An evaluator that can generate favours itself
> TMR's own generator scores **92.44** R@1 under TMR on HumanML3D, against **67.16** for real motions. Evaluators that can also synthesize prefer their own model's outputs. Keep this in mind when one lab's evaluator scores that lab's generator.

**Limitations:** small training data (about 23K motions); poor generalization to motion in the wild.

## Related

[[tmr]] · [[text-to-motion]] · [[human-motion-diffusion-model-tevet-2022]] · [[kimodo-kinematic-motion-diffusion-nvidia-2026]]
