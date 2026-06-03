---
title: "ELF: Embedded Language Flows"
type: source
created: 2026-06-03
updated: 2026-06-03
sources:
  - elf-embedded-language-flows-hu-2026.pdf
arxiv_id: "2605.10938"
authors:
  - Keya Hu
  - Linlu Qiu
  - Yiyang Lu
  - Hanhong Zhao
  - Tianhong Li
  - Yoon Kim
  - Jacob Andreas
  - Kaiming He
first_author: Keya Hu
year: 2026
tags: [2026]
status: complete
importance: medium
---

# ELF: Embedded Language Flows

*Surfaced via the alphaXiv / NLP-newsletter digest.*

## Summary

ELF (MIT) shows that *continuous* diffusion language models can be made effective with minimal adaptation from the image domain, in contrast to today's leading discrete-token DLMs. ELF is a class of continuous-time **Flow Matching** models operating in continuous embedding space: denoising trajectories run from Gaussian noise to clean token embeddings and stay continuous until the final time step (t=1), where a shared-weight network maps embeddings to discrete tokens. Because the model is continuous in both time and space, established image-diffusion techniques such as classifier-free guidance (CFG) transfer almost directly. Experiments report that ELF substantially outperforms leading discrete and continuous DLMs, reaching better generation quality (lower generative perplexity) with far fewer sampling steps.

## Key points

- Continuous-time Flow Matching defines a velocity field over continuous embedding space (continuous in both time and space).
- Discretization to tokens happens only at the final step via a shared-weight network, keeping the bulk of generation in embedding space.
- This formulation makes image-domain tools like classifier-free guidance (CFG) straightforward to apply to language.
- Reports lower generative perplexity with fewer sampling steps (e.g., 32 vs 1024) than prior discrete/continuous DLMs.
- Includes distillation variants ("w/ distill") that further cut sampling cost.
- Positions continuous embedding-space flows as a promising alternative path for non-autoregressive language modeling.

## Concepts & entities

- [[latent-reasoning]]
- [[transformer-architecture]]
- [[chain-of-thought]]

## References

- [arXiv:2605.10938](https://arxiv.org/abs/2605.10938)
- Code: https://github.com/lillian039/ELF
