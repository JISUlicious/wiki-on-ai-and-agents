---
title: JEPA (Joint-Embedding Predictive Architecture)
type: concept
created: 2026-11-03
updated: 2026-11-03
sources:
  - platonic-representation-hypothesis-jepas-balestriero-2026.md
  - learn-from-your-own-latents-korchinski-2026.md
status: draft
importance: medium
---

# JEPA (Joint-Embedding Predictive Architecture)

A **Joint-Embedding Predictive Architecture** is a self-supervised learning setup in which a model predicts the *embedding* of one part of its input from the embedding of another, rather than reconstructing pixels or predicting tokens. [[yann-lecun|Yann LeCun]] proposed it in "A Path Towards Autonomous Machine Intelligence" (2022) as the basis for [[world-model|world models]] that predict in an abstract space and can ignore unpredictable detail.

## How it works

- **Two views.** Two related inputs, called a *positive pair*, are encoded. Examples: two crops of an image, a masked and an unmasked region, two frames of a video, or two captions of one picture.
- **A prediction term.** One embedding is predicted from the other, either directly or through a learned predictor that may be conditioned on an action.
- **An anti-collapse term.** Without it, mapping every input to the same vector would minimize the prediction loss. Methods differ here: VICReg and Barlow Twins regularize the covariance, contrastive methods push negatives apart, and LeJEPA's **SIGReg** (Balestriero and LeCun, 2025) pushes embeddings toward an isotropic Gaussian.

Named members of the family include I-JEPA (images), V-JEPA (video) and LeJEPA. These have no pages in this wiki yet.

## What the wiki's sources say

- **Data efficiency.** [[learn-from-your-own-latents-korchinski-2026]] proves, on a probabilistic grammar, that predicting one's own latents (JEPA or data2vec style) can need exponentially fewer samples than predicting tokens.
- **Shared representations across modalities.** [[platonic-representation-hypothesis-jepas-balestriero-2026]] proves that JEPAs trained separately on different modalities learn the same embedding up to one rotation when their positives relate the same world states (see [[platonic-representation-hypothesis]]).
  - The choice of positives decides this. "Good views" are two observations of one scene; "bad views" are two corruptions of one observation.
  - The paper argues that next-token prediction has the form of a bad-view JEPA.
  - A 109M text JEPA trained with good views aligns with vision models better than language models up to 72B parameters.
  - Caveat: the analysis uses an identity predictor, so it does not cover the action-conditioned JEPAs used as world models.

Both results come from the JEPA side of a debate with next-token prediction, and the 2026 paper's authors are affiliated with a JEPA-focused company (AMI Labs). Neither has yet been independently replicated in this wiki's sources.

## Related

- [[world-model]] · [[latent-reasoning]] · [[platonic-representation-hypothesis]]
- [[dinov2]]: self-distillation on crops of one image, which the 2026 paper classes as bad views.
- [[clip]]: contrastive image–text training, the paired-data alternative.

## References

- [[platonic-representation-hypothesis-jepas-balestriero-2026]]
- [[learn-from-your-own-latents-korchinski-2026]]
