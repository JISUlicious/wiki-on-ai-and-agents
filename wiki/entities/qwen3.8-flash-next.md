---
title: Qwen3.8-Flash-Next
type: entity
created: 2026-08-30
updated: 2026-10-20
sources:
  - qwen3.8-flash-next-model-card-2026.md
family: qwen
parameters: "125B total / 6B activated (+51B n-gram embedding, +4B MTP)"
release_date: 2026-08-27
builds-on:
  - [[mixture-of-experts]]
  - [[linear-attention]]
  - [[rope-scaling]]
status: complete
importance: medium
tags:
  - model
  - 2026
---

# Qwen3.8-Flash-Next

An open-weight multimodal MoE from [[qwen|Qwen]] ([[alibaba|Alibaba]]), released 2026-08-27 under the Qwen Community License 1.0. `model_type: qwen4_exp` — the first open-weight release under a new architecture line, with the "-Next" suffix marking it as the experimental branch ahead of the productized **Qwen3.8-Flash** on Qwen Cloud.

**125B total, 6B activated**, plus a **51B n-gram embedding table** and 4B of MTP parameters — an unusual split, since the embedding mass is roughly 40% of the model and is explicitly justified as a parameter-scaling axis that is cheaper to compute and easier to offload than more experts.

## Architecture

48 layers, hidden 2560, laid out as **12 × (3 × Gated DeltaNet → 1 × Qwen Sparse Attention)**, each block followed by MoE. So **36 of 48 layers are [[linear-attention]]** and only 12 are global attention.

- **Qwen Sparse Attention (QSA)** — the global layers are themselves sparse, using an MQA indexer (4 query heads, 1 shared key head, head dim 128) with a budget of **512 blocks / 2,048 tokens**. Selection is at *micro-block* rather than per-token granularity, which the card credits for the long-context latency gain. 24 query heads : 2 KV heads ([[grouped-query-attention|GQA]]), head dim 256, of which **only 64 dims are rotary**.
- **MoE** — 512 experts, 10 routed + 1 shared, intermediate 640.
- **N-gram embedding** — a 20M-entry bigram/trigram table indexed at layer 2.
- **Gated Residual** — 4 branches with bottleneck rank 320, modulating widened [[residual-stream|residual streams]] via a data-dependent read gate and a per-branch scalar write gate. A direct descendant of the same idea as [[attention-residuals-kimi-team-2026|AttnRes]]: the residual stream stops being a plain unweighted sum. See [[depth-mixing]].
- **MTP** — 1 layer, trained multi-step.
- Vision encoder: depth 27, hidden 1152, patch 16, spatial merge 2.

Trained with **Muon and AdamW applied to different weight categories**, with batch-size warmup eliminated in favour of starting at the target batch size, justified by refitted [[scaling-laws]].

## Context: 256K native, 1M by opt-in

`max_position_embeddings: 262144` is the trained window. The 1M figure is **[[rope-scaling|YaRN]] at `factor: 4.0`**, and it is *not* in the shipped config — which ships `"rope_type": "default"`. You enable it by overriding `rope_parameters` and setting an env var to permit the longer window.

Three architectural properties make the 4× stretch cheap: only **64 of 256 head dims are rotary** (`partial_rotary_factor: 0.25`), `rope_theta` is already **10,000,000**, and only 1 layer in 4 uses global attention — those 12 attending to a fixed ~2K-token selection at any length. The quadratic term is largely designed out before RoPE scaling is asked to do anything.

The extension is aimed at long *generation* as much as long input: the stated limit covers input plus output, and the card's own recommendation is 262,144 reasoning tokens plus 131,072 response tokens — 393,216 of output budget, which only fits under the 1M ceiling.

> [!warning] The extension path is documented, not evidenced
> Every benchmark that states a context window states **256K, the native length**. The one genuine long-context result — **LVBench 76.6** — runs at ~224k video tokens, also inside the native window. No long-text retrieval suite (RULER, NIAH, LongBench, HELMET, MRCR) is reported at all.
>
> Enabling YaRN also has a standing cost: OSS frameworks implement it **statically**, so `factor: 4.0` degrades short-context quality on every request, not only long ones. Qwen recommends sizing the factor to the actual workload.
>
> [[cracks-in-the-foundation-long-context-extension-bertsch-2026|Bertsch et al.]] found that aggressive GQA plus hybrid/sliding attention — this model's exact profile — compound to as much as −47% relative *after* context extension, invisibly in loss. That makes the missing evaluation load-bearing rather than pedantic.

## Reported results

| | Qwen3.8-Flash-Next | Qwen3.8-27B | DeepSeek-V4-Flash-0731 | Claude-Opus-4.6 (Max) |
|---|---:|---:|---:|---:|
| SWE-bench Multilingual | **81.0** | 73.8 | — | 77.5 |
| SWE-bench Pro | **62.5** | 61.7 | 56.0 | 53.4 |
| DeepSWE 1.1 | **58.7** | 42.2 | 54.4 | — |
| LiveCodeBench v6 | **91.9** | 90.3 | 90.6 | 88.8 |
| GPQA Diamond | **91.7** | 89.2 | 90.8 | 91.3 |
| CoWorkBench | **73.9** | 70.7 | 45.1 | 68.2 |
| JobBench | **55.7** | 33.4 | 41.3 | 36.6 |
| HLE | 35.9 | 30.8 | 33.8 | **40.0** |

Multimodal: AndroidWorld 84.5, LVBench 76.6, RealWorldQA 88.5, ERQA 72.3, OSWorld 2.0 partial 52.3.

Note the activated-parameter comparison — 6B activated against Qwen3.8-27B's 27B dense and DeepSeek-V4-Flash's 13B — which is the release's actual claim.

## Lineage

Part of the [[qwen|Qwen]] family; the architecture line (`qwen4_exp`) is new rather than an increment on Qwen3. The Gated DeltaNet + gated-attention pairing it revises is shared ancestry with [[moonshot-ai|Moonshot's]] [[kimi-linear-kimi-team-2025|Kimi Linear]] hybrid — both settle on a 3:1 linear-to-full ratio, reached independently.

## Related

- [[qwen]] · [[alibaba]] · [[qwen3.8-flash-next-model-card-2026]]
- [[rope-scaling]] — the context-extension mechanism.
- [[linear-attention]] · [[grouped-query-attention]] · [[mixture-of-experts]] · [[kv-cache]] · [[residual-stream]]
- [[kimi-linear-kimi-team-2025]] · [[moonshot-ai]] — the convergent hybrid design.
- [[vllm]] · [[sglang-radixattention-zheng-2023]] — the serving stacks that implement the YaRN override.

## References

- [[qwen3.8-flash-next-model-card-2026]] · [[cracks-in-the-foundation-long-context-extension-bertsch-2026]]

## Related 2026 sources

- [[sparse-attention]] — Qwen Sparse Attention's 2,048-token indexer budget is the *trained* form of what MInference assigns offline and Token Sparse Attention selects online
- [[persistent-state-dataflow-accelerator-linear-attention-decode-gupta-2026]] — an FPGA accelerator targeting this architecture's GDN layers specifically — it cites 'Qwen3-Next uses 75% GDN layers', matching this page's 36-of-48 split
- [[trained-sparse-attention]] — what the sparse-attention layers in this hybrid are, and why token-axis retrofits fit it better than attention-axis ones
- [[uniprefill-block-wise-dynamic-sparsification-fan-2026]] — the hybrid-architecture rule, measured on the sibling Qwen3-Next-80B-A3B: sparse attention ≤1.11×, [[token-pruning|token pruning]] 1.08–1.68× and negative below 16K
