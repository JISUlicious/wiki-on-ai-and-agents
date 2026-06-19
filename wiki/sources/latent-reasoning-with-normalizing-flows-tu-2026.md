---
title: "Latent Reasoning with Normalizing Flows (NF-CoT)"
type: source
created: 2026-06-19
updated: 2026-06-19
sources:
  - latent-reasoning-with-normalizing-flows-tu-2026.md
arxiv_id: "2606.06447"
authors:
  - Guancheng Tu
  - et al.
first_author: Guancheng Tu
year: 2026
tags:
  - 2026
status: complete
importance: medium
---

# Latent Reasoning with Normalizing Flows (NF-CoT)

Surfaced via the alphaXiv weekly digest (2026-06-10).

## Summary

Explicit [[chain-of-thought]] improves LLM reasoning but forces intermediate computation through a discrete, serial token stream: every step must be verbalized before the model proceeds, even when the underlying update is semantic or only partially formed. [[latent-reasoning]] offers a higher-bandwidth alternative by computing in compact continuous states before committing to text — but prior latent methods often sacrifice the properties that make CoT effective in autoregressive models: native left-to-right generation, probabilistic sampling, KV-cache compatibility, and tractable likelihoods. NF-CoT (Tu et al., UPenn / UC San Diego / Meta) preserves all of these by modeling continuous "thoughts" with autoregressive normalizing flows.

NF-CoT instantiates a TARFlow-style normalizing flow inside the LLM backbone, defining a tractable probability model over compact continuous thoughts distilled from explicit CoT. Continuous-thought positions are produced by an NF head while text positions use the standard LM head, all within one causal stream. This gives exact likelihoods for latent thoughts, probabilistic left-to-right decoding with the original KV cache, and direct policy-gradient optimization in latent space. On code-generation benchmarks it improves Qwen3-8B-Base from 55.8 to 68.8 average pass@1 (+13.0%) and beats the diffusion-based latent reasoner LaDiR by +7.1% on average, while running 1.92x faster overall (and 2.70x faster at latent generation) than LaDiR by replacing iterative denoising with a single autoregressive flow pass.

## Key points

- NF-CoT models continuous latent "thoughts" with a TARFlow-style autoregressive normalizing flow embedded in the LLM backbone, distilled from explicit CoT.
- Dual-head causal stream: an NF head generates continuous-thought positions, the standard LM head generates text positions — preserving left-to-right decoding and the original KV cache.
- Exact likelihoods for latent thoughts enable probabilistic decoding and direct policy-gradient (RL) optimization in the latent reasoning space.
- +13.0% pass@1: improves Qwen3-8B-Base from 55.8 to 68.8 on code-generation benchmarks (Dual-Path variant: +9.4%).
- Outperforms the diffusion-based latent reasoner LaDiR by +7.1% average while being 1.92x faster overall and 2.48x cheaper per sample.
- Replacing iterative diffusion denoising with an autoregressive flow is the key efficiency win (2.70x faster latent generation; large training-throughput and FLOP savings).

## Concepts & entities

- [[latent-reasoning]] — NF-CoT is a latent reasoning framework
- [[chain-of-thought]] — the explicit-CoT baseline that NF-CoT compresses into continuous thoughts
- Related: normalizing flows, TARFlow, knowledge distillation, diffusion-based latent reasoning (plain text — no dedicated pages); affiliations UPenn / UC San Diego / Meta

## References

- [arXiv:2606.06447](https://arxiv.org/abs/2606.06447)
- Project: nf-cot.vercel.app
