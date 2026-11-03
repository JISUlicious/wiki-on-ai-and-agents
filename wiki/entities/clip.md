---
title: CLIP
type: entity
created: 2026-11-03
updated: 2026-11-03
sources:
  - platonic-representation-hypothesis-jepas-balestriero-2026.md
family: clip
builds-on:
  - [[vit]]
release_date: 2021-01-05
status: draft
importance: high
tags:
  - model
---

# CLIP

**Contrastive Language–Image Pre-training**, from [[openai|OpenAI]] (Radford et al., "Learning Transferable Visual Models From Natural Language Supervision", [arXiv:2103.00020](https://arxiv.org/abs/2103.00020), January 2021). It trains an image encoder (a ResNet or a [[vit|ViT]]) and a text encoder jointly on about 400M image–text pairs from the web. A contrastive loss pulls each image towards its own caption and away from the others in the batch. The result is a shared embedding space that supports zero-shot classification by comparing an image with text prompts such as "a photo of a dog".

## Lineage

CLIP made contrastive image–text pretraining the standard way to connect vision and language. Its image encoders became the visual front end of many later systems, including [[multimodal-llm|multimodal LLMs]] and text-to-image models, and of image captioners such as the LLaVA recaptioner used in the 2026 PRH paper. The contrastive objective descends from the negative-sampling line of [[word-embeddings]] (see [[negative-sampling]]).

## In this wiki

- **Baseline for the [[platonic-representation-hypothesis]].** In [[platonic-representation-hypothesis-jepas-balestriero-2026]], CLIP's text encoder reaches .124 mutual 10-NN with [[dinov2|DINOv2]] on WIT-1024. A 109M text [[jepa|JEPA]] trained without any image–text loss reaches .184. CLIP's text encoder is still better on two of the pair-free tests (FOSCTTM and ImageNet-100).
- **Leakage caveat.** That paper's large-scale text partly comes from LLaVA, whose vision encoder is CLIP ViT-L/14. So some CLIP-derived image–text information enters even the "pair-free" text encoders.
- Mentioned elsewhere as an encoder in multimodal and motion models: [[vit]], [[multimodal-llm]], [[mdm]], [[kimodo]].

The model details on this page come from the CLIP paper, which has no source page in this wiki.

## References

- [[platonic-representation-hypothesis-jepas-balestriero-2026]]
- [CLIP, arXiv:2103.00020](https://arxiv.org/abs/2103.00020)
