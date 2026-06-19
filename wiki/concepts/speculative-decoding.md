---
title: "Speculative Decoding"
type: concept
created: 2026-06-03
updated: 2026-06-03
sources:
  - speculative-speculative-decoding-kumar-2026.md
status: complete
importance: medium
tags:
  - 2026
---

# Speculative Decoding

Speculative decoding is a lossless technique for accelerating autoregressive LLM inference. A small, fast **draft model** proposes the next few tokens, which a larger, slower **target model** then verifies in a single parallel forward pass. A verification algorithm guarantees the accepted tokens follow exactly the target model's distribution, so output quality is unchanged — the speedup comes purely from doing more work per target-model pass instead of one token at a time. The target model decides how many speculated tokens to accept and samples one bonus token after the accepted prefix.

The technique was introduced by Leviathan et al. (2023) and Chen et al. (2023). Its main remaining bottleneck is the **sequential dependence** between speculation and verification: verification must finish before the next draft round begins. [[speculative-speculative-decoding-kumar-2026|Speculative speculative decoding (SSD)]] attacks exactly this dependence by pre-speculating against predicted verification outcomes in parallel.

## Key points

- Lossless: accepted tokens match the target distribution exactly.
- Draft model proposes; target model verifies in one parallel pass.
- Speedup scales with the draft model's acceptance rate.
- Bottleneck: sequential dependence between drafting and verification.

## Related

- [[autoregressive-language-modeling]] — the sequential generation process being accelerated.
- [[kv-cache]] — interacts with how draft/target caches are managed.
- [[inference-time-compute]] — broader inference-efficiency context.

## References

- [[speculative-speculative-decoding-kumar-2026]] — generalizes speculative decoding by parallelizing drafting and verification.
- Leviathan et al. 2023; Chen et al. 2023 — original speculative decoding papers.
