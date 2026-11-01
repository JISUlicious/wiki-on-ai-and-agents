---
title: "Comparing Sparsified MLPs and Transcoders: Many Transcoder Features Are Neurons in Disguise"
type: source
created: 2026-10-29
updated: 2026-10-29
sources:
  - sparsified-mlps-vs-transcoders-neurons-in-disguise-shabalin-2026.md
arxiv_id: "2610.24521"
venue: NeurIPS 2026 Workshop (Interpretability as a Science)
authors:
  - Stepan Shabalin
  - Mateusz Piotrowski
year: 2026
status: complete
importance: high
tags:
  - 2026
  - paper
---

# Comparing Sparsified MLPs and Transcoders: Many Transcoder Features Are Neurons in Disguise

**Stepan Shabalin** (Anthropic Fellows; Georgia Institute of Technology) **and Mateusz Piotrowski** ([[anthropic|Anthropic]]), equal contribution. [arXiv:2610.24521](https://arxiv.org/abs/2610.24521), v1, cs.AI, 17 Oct 2026. NeurIPS 2026 workshop paper (Interpretability as a Science), 60 pages.

## Summary

Transcoders are sparse, learned stand-ins for an MLP layer. The case for them rests on MLP neurons being [[polysemanticity|polysemantic]]. This paper asks what transcoders add over the raw neurons. A gated-MLP neuron is approximately bilinear at high activation, so each neuron, once split by sign, has one effective encoder (W_up ± W_gate) and one decoder (±W_down). With top-k thresholding the MLP itself becomes a narrow transcoder.

The authors define a **neuron feature** as a transcoder feature whose encoder and decoder both exceed a cosine threshold (0.6–0.9 depending on the transcoder; mostly 0.6–0.7) to the same neuron's directions. Across **13 open-source per-layer transcoders** on Gemma 2/3, Qwen 3 and Llama 3.x models (1B–27B, all dense), these features form a separate cluster. They appear about halfway through the model and are a small share of the dictionary, but a large share of activations in late layers.

## Key points

- **Prevalence.** In late layers of the Gemma family, neuron features are 5.9% → 8.9% → 15.6% → 37.1% of the dictionary from 1B to 27B. Their share of activations goes from 57.9% to 82.8%. They fire 16.9× more often than other features (Gemma 3 4B, L22).
- **Tracking.** A threshold on the neuron's activation predicts when its neuron feature fires with median AUC ≥ 0.977.
- **Faithfulness lives in neuron features.** The transcoder's Jacobian alignment with the MLP is about 0.3 (Gemma 3 4B L22). Neuron features alone reach that. The other features reach only about 0.05.
- **Stability.** Across two training seeds, most features that recur are neuron features. A 1M-feature dictionary found only about 22k stable features, close to the 20,480 sign-split neurons in that layer.
- **Non-neuron features barely train.** In TopK transcoders they keep median cosine +0.48 to their random initialization. Neuron features move to about 0.03. The JumpReLU pipeline behaves differently: most live features rotate.
- **Sparsified neurons as a replacement model.** After a trained linear correction, top-k neurons beat transcoders on KL (closeness to model behaviour) but lose on FVU (reconstruction). The authors read this as some transcoder structure fitting reconstruction only.
- **Neurons match or beat transcoders on sparse probing** across eight transcoders. Transcoders win clearly on **bigrams**: the dictionary feature fires on the target bigram for 75.6% of bigrams, neurons for about 0%. The MoE model Gemma 4 26B A4B's neurons manage 20.1%.
- **Case studies.** In line-break counting, 7 of 8 clean position features track one neuron. In arithmetic, ablating the neuron features for Llama-3.1-8B's addition neurons cuts accuracy from 94.3% to 30.6%. In the "red" case, transcoders mix 2–4 polysemantic neurons into a clean feature, so they do resolve some real polysemanticity.

## What it does not show

- **Only top activations.** Sort evals put neurons on par with transcoder features at the *top* of the activation range. Under quantile sampling neurons degrade. This supports "neurons are clean at high activation", not "neurons are monosemantic".
- **Scope is narrow.** Base models only, mostly English, dense models, per-layer transcoders, first-order (Jacobian) faithfulness only. MoE and cross-layer transcoders get only preliminary results.
- **No conclusion on the non-neuron remainder.** The authors say they cannot yet decide whether non-neuron features are faithful. They also say the weight-based neuron-feature definition is not robust, because non-neuron features keep part of their initial direction.

## Concepts & entities

- [[monosemanticity]] · [[polysemanticity]] · [[superposition]]: bears on the open "neurons are already clean" contradiction on the monosemanticity page. It supports that view in late layers and at high activation, and keeps polysemanticity for broad ranges and fine-grained concepts.
- [[transcoder]] · [[sparse-autoencoder]] · [[circuits]] · [[mechanistic-interpretability]]: transcoders and attribution graphs are the SAE lineage applied to MLP input-output maps.
- [[neuron-explorer-lyramakesmusic-2026]] · [[neuron-explorer]] · [[goodfire]] · [[nick-cammarata]]: same sign-split ("poles") neuron view, from different evidence.
- [[anthropic]] · [[gemma]] · [[qwen]] · [[llama-3]]

## References

- [arXiv:2610.24521](https://arxiv.org/abs/2610.24521)
