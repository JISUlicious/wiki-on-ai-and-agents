---
title: Platonic Representation Hypothesis
type: concept
created: 2026-11-03
updated: 2026-11-03
sources:
  - platonic-representation-hypothesis-jepas-balestriero-2026.md
status: draft
importance: medium
builds-on:
  - [[word-embeddings]]
---

# Platonic Representation Hypothesis

The **Platonic Representation Hypothesis** (PRH) is the conjecture that neural networks trained on different data and modalities converge toward the *same* representation of the world as they scale: a shared statistical model of the reality that generated the data. The name refers to the shadows in Plato's cave, which are different projections of one underlying world.

It was proposed by Huh, Cheung, Wang and Isola in "Position: The Platonic Representation Hypothesis" (ICML 2024, [arXiv:2405.07987](https://arxiv.org/abs/2405.07987)). Their evidence was empirical: the nearest-neighbour structure of language models' embeddings agrees more with vision models', such as [[dinov2|DINOv2]], as the language models get larger. Their supporting argument was an idealized contrastive learner. That paper has no source page in this wiki yet.

## How alignment is measured

- **Mutual k-NN** (usually k = 10): for paired items, such as an image and its caption, the overlap between their k nearest neighbours in each model's space. No fitting is involved.
- **Linear CKA:** a kernel similarity between the two sets of embeddings.
- **Retrieval through one rotation:** fit an orthogonal map on some pairs and test whether it retrieves the right partner. This tests the stronger claim that the spaces are equal *up to rotation*.

The benchmark of the original paper is WIT-1024 (Wikipedia images with their captions).

## From hypothesis to theorem (2026)

[[platonic-representation-hypothesis-jepas-balestriero-2026|Balestriero and LeCun (2026)]] prove a version of it for [[jepa|JEPAs]]:

- **The theorem.** In an idealized setting, JEPAs trained separately on any number of modalities learn the same embedding up to one rotation, provided their positive pairs relate the same world states.
- **The recipe.** What matters is the choice of positives. Two observations of one scene ("good views") give an embedding that belongs to the world. Two corruptions of one observation ("bad views") give one that partly encodes the modality's own nuisance.
- **The reading of LLMs.** Next-token prediction has the form of a bad-view JEPA, which the authors offer as the reason language models align with vision only partly, and mainly through scale.
- **The evidence.** A 109M text JEPA trained with good views aligns with DINOv2 better than [[qwen-2-5|Qwen2.5-72B]], [[llama-3|Llama-3-70B]] and [[clip|CLIP]]'s text encoder on WIT-1024, and reaches next-token prediction's alignment with 42× fewer samples.

The theorem is a population statement about a finite world and second-order anti-collapse; the experiments do not meet its conditions exactly. It also excludes JEPAs with learned predictors, which is the form used as [[world-model|world models]]. See the source page for the caveats on generated captions and tuning.

## Why it matters

- If it holds, representations become **composable**: separately trained encoders can be joined by one linear map, or even aligned with no paired data, as the 2026 paper does to build an image captioner.
- It reframes a scaling observation as a question of **training objective**. If the 2026 result generalizes, scale is one route to shared representations and an inefficient one.
- It is the cross-modal version of the older finding that [[word-embeddings]] trained separately on two languages are related by a rotation that can be found without parallel text.

## Related

- [[linear-representation-hypothesis]]: a within-model claim that concepts are linear directions; the PRH concerns agreement between different models.
- [[mechanistic-interpretability]] · [[multimodal-llm]] · [[scaling-laws]]

## References

- [[platonic-representation-hypothesis-jepas-balestriero-2026]]
