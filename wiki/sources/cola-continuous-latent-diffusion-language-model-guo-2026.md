---
title: "Cola: Continuous Latent Diffusion Language Model"
type: source
created: 2026-06-03
updated: 2026-06-03
sources:
  - cola-continuous-latent-diffusion-language-model-guo-2026.pdf
arxiv_id: "2605.06548"
authors:
  - Hongcan Guo
  - Qinyu Zhao
  - Yian Zhao
  - Shen Nie
  - Rui Zhu
  - Yan Zeng
first_author: Hongcan Guo
year: 2026
tags: [2026]
status: complete
importance: medium
---

# Cola: Continuous Latent Diffusion Language Model

*Surfaced via the alphaXiv / NLP-newsletter digest.*

## Summary

Cola DLM is a hierarchical latent **diffusion language model** from ByteDance Seed that reframes text generation as hierarchical information decomposition rather than strict left-to-right autoregression. The pipeline has three stages: a Text VAE learns a stable text-to-latent mapping, a block-causal Diffusion Transformer (DiT) models a global semantic prior in continuous latent space, and a conditional decoder realizes the final tokens. Viewed through a unified Markov-path lens, the diffusion process performs latent *prior transport* instead of token-level observation recovery, decoupling global semantic organization from local textual realization. This yields a flexible non-autoregressive inductive bias and a natural bridge from discrete text toward continuous modalities (e.g., vision).

## Key points

- Three-stage design: Text VAE (text↔latent) → block-causal DiT (continuous-space prior) → conditional decoding.
- Diffusion runs in continuous latent space and transports a *prior* rather than recovering tokens, separating semantics from surface form.
- Block-causal prior modeling preserves cross-block causal structure while enabling within-block non-autoregressive generation and faster inference.
- Evaluated across 4 research questions and 8 benchmarks against strictly matched ~2B-parameter autoregressive and LLaDA baselines, with scaling curves up to ~2000 EFLOPs.
- Reports strong, favorable scaling behavior; argues generation quality (not likelihood) better reflects model capability.
- Highlights a likelihood-vs-generation-quality mismatch and offers preliminary evidence of unified discrete-text/continuous-vision modeling.

## Concepts & entities

- [[latent-reasoning]]
- [[transformer-architecture]]
- [[chain-of-thought]]

## References

- [arXiv:2605.06548](https://arxiv.org/abs/2605.06548)
- Project page: https://hongcanguo.github.io/Cola-DLM/
