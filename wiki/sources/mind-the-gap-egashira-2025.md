---
title: "Mind the Gap: A Practical Attack on GGUF Quantization"
type: source
created: 2026-07-07
updated: 2026-07-07
sources: []
arxiv_id: "2505.23786"
venue: ICML 2025 (PMLR v267)
year: 2025
authors:
  - Kazuki Egashira
  - Martin Vechev
introduces:
  - [[quantization-conditioned-backdoor]]
tags:
  - 2025
  - paper
status: complete
importance: high
---

# Mind the Gap: A Practical Attack on GGUF Quantization

**Egashira et al.** (ETH Zurich) — [arXiv:2505.23786](https://arxiv.org/abs/2505.23786), **ICML 2025**.

## Summary

Extends [[exploiting-llm-quantization-egashira-2024|the quantization-trigger attack]] to **GGUF** — the format [[llama-cpp]] and ollama actually run. This matters because GGUF's block-wise k-quant schemes are considerably more complex than the round-to-nearest bitsandbytes formats attacked in the original work, and that complexity was widely assumed to make the attack impractical.

It does not.

## The insight

The **quantization error** itself — the difference between the full-precision weight and its de-quantized value — provides enough slack to hide malicious behavior. The attack trains the malicious model under weight constraints derived from that error, rather than needing a clean analytic rounding bucket.

Evaluated across **3 LLMs × 9 GGUF quantization data types × 3 scenarios**.

## Results

Deltas from full-precision to quantized behavior:

| Scenario | Δ |
|---|---|
| Insecure code generation | **88.7%** |
| Targeted content injection | **85.0%** |
| Benign instruction refusal | **30.1%** |

## The conclusion that matters

> "The complexity of quantization schemes alone is insufficient as a defense."

This is the most practically alarming result in the [[quantization-conditioned-backdoor]] literature, because GGUF is the dominant format for locally-run community models — precisely the setting where users download third-party weights and quantize (or download pre-quantized artifacts) without institutional review.

## Concepts & entities

- [[quantization-conditioned-backdoor]] — the attack class.
- [[exploiting-llm-quantization-egashira-2024]] — the original (bitsandbytes) attack.
- [[llama-cpp]] · [[gguf]] — the ecosystem attacked.

## References

- [arXiv:2505.23786](https://arxiv.org/abs/2505.23786) — Mind the Gap (ICML 2025)
