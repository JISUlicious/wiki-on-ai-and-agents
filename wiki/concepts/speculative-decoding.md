---
title: "Speculative Decoding"
type: concept
created: 2026-06-03
updated: 2026-09-10
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

Two frontier drafting directions push on the *drafter* and the *verification schedule*: **parallel / semi-autoregressive drafters** (propose a whole block in one pass) and **adaptive verification** (decide how much of the block to check). [[dspark-confidence-scheduled-speculative-decoding-deepseek-2026|DSpark]] ([[deepseek|DeepSeek]] + [[peking-university|PKU]], 2026) combines both — a semi-autoregressive drafter (parallel backbone + lightweight sequential module) to fight *suffix/acceptance decay*, plus **confidence-scheduled verification** that tailors verification length per request to protect serving throughput under load (reported 60–85% faster per-user generation).

## Key points

- Lossless: accepted tokens match the target distribution exactly.
- Draft model proposes; target model verifies in one parallel pass.
- Speedup scales with the draft model's acceptance rate.
- Bottleneck: sequential dependence between drafting and verification.

> [!note] Field measurement: an MTP head on a dense 27B, [[apple-silicon|Apple Silicon]] (2026-09-10)
> Qwen3.8-27B dense, 4-bit, M4 Air 32 GB, oMLX, single stream, contributed by a peer session. Drafter `mlx-community/Qwen3.8-27B-MTP-4bit`, **0.22 GB — 1.4% of the 15.7 GiB resident model**, block size 3.
>
> | | 120 tokens | 240 tokens |
> |---|---|---|
> | baseline decode | 6.6 tok/s | 6.7 tok/s |
> | + MTP head | 12.92 | 14.11 |
> | speedup | 1.96× | **2.11×** |
>
> Acceptance *rises* with generation length — 83.3% at 9 rounds, 91.7% at 42, **94.6% at 83** (2.89 tokens per round of a possible 3). That is well above what a generic draft model achieves, and the peer's hypothesis is structural: an MTP head is trained to predict its own parent's next tokens, not to approximate a different network's distribution. If it generalizes, it is a stronger argument for MTP heads over draft-model speculation than the speedup alone. One model, one machine; not yet reproduced.

## Related

- [[autoregressive-language-modeling]] — the sequential generation process being accelerated.
- [[kv-cache]] — interacts with how draft/target caches are managed.
- [[inference-time-compute]] — broader inference-efficiency context.

## References

- [[speculative-speculative-decoding-kumar-2026]] — generalizes speculative decoding by parallelizing drafting and verification.
- [[dspark-confidence-scheduled-speculative-decoding-deepseek-2026]] — semi-autoregressive drafting + confidence-scheduled verification (DeepSeek/PKU 2026).
- Leviathan et al. 2023; Chen et al. 2023 — original speculative decoding papers.

> [!note] Quantization connection
> QSpec ([arXiv:2410.11305](https://arxiv.org/abs/2410.11305)) applies speculative decoding *across precisions* — a fast lossy W4A4 drafter verified by an accurate W4A16 model, for 1.18–1.64× over W4A16 alone. See [[quantization-performance]].

## Related 2026 sources

Surfaced via newsletter ingests; see [[index]].

- [[speculate-while-you-reason-ji-2026]] — self-speculation of the agent's own next tool call beats external drafters *and* is cheaper by reusing the prefix KV cache
- [[speculative-prefill]] — **not the same thing** — a draft scoring *input* tokens so the target prefills a subset; accelerates prefill, not decode. A draft with an MTP head can serve both, but they are two levers
