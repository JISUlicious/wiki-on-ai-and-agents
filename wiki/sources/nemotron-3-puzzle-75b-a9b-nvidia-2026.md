---
title: "Nemotron-Labs-3-Puzzle-75B-A9B: Compressing Hybrid MoE LLMs"
type: source
created: 2026-07-22
updated: 2026-07-22
sources:
  - nemotron-3-puzzle-75b-a9b-nvidia-2026.md
arxiv_id: "2607.04371"
year: 2026
authors:
  - Akhiad Bercovich
  - Pavlo Molchanov
  - Saurav Muralidharan
  - Mostofa Patwary
  - Ran El-Yaniv
tags:
  - 2026
  - paper
status: complete
importance: medium
---

# Nemotron-Labs-3-Puzzle-75B-A9B: Compressing Hybrid MoE LLMs

**Akhiad Bercovich et al.** (NVIDIA; ~70 authors, last author Ran El-Yaniv) — [arXiv:2607.04371](https://arxiv.org/abs/2607.04371), cs.AI, v2 dated 7 Jul 2026. Checkpoints released on Hugging Face in BF16, NVFP4, and FP8.

## Summary

NVIDIA compresses its hybrid Mamba-Attention-MoE model [[nemotron-3-super-nvidia-2026]] (120.7B total / 12.8B active) down to **75.3B total / 9.3B active** — 62.4% of total and 73.1% of active parameters — explicitly optimizing for *server throughput under an interactivity constraint* rather than for raw parameter reduction. The two target deployments are (1) interactive serving on a single 8×B200 node where per-user throughput is the user-experience constraint, and (2) ultra-long-context agentic workloads on a **single H100** where HBM capacity, not compute, is the binding constraint at 1M-token contexts.

The method is **Iterative Puzzle**, a sequential extension of NVIDIA's Puzzle decomposed-NAS framework. Puzzle defines a search space of per-layer "puzzle pieces," scores each, and solves a mixed-integer program to pick one alternative per layer under deployment constraints — but its single-shot formulation assumes block-replacement quality impacts are additive, ignoring higher-order interactions (pruning one MoE layer changes the representations later layers see). Iterative Puzzle instead alternates moderate pruning with short knowledge-distillation recovery phases, recomputing replacement scores in the context of the already-compressed model. Three stages, healed for 24B, 43.2B, and 52.8B tokens respectively. The search targets MoE layers specifically: heterogeneous intermediate-channel pruning of routed experts, per-layer **top-k reduction** (parent uses k=22; the compressed model ranges 4–18 active experts across layers, mean 50%), and uniform **Mamba SSM state pruning from 128 to 96 channels** (a 1.2–1.3× SSM kernel speedup at decode for batch sizes 8–512). Attention layers are left untouched because Nemotron-3-Super is already KV-cache efficient. Recovery is large-scale KD (short- then long-context) plus RL post-training; deployment adds FP8/NVFP4 PTQ and an MTP speculative-decoding head.

Results hold quality within a point or two nearly everywhere and roughly double decode-heavy serving throughput. Across six interactive operating points the boost ranges **1.60×–2.14×**; the "~2×" headline is specifically the decode-heavy 8K/64K regime, while the prefill-heavy 50K/2K regime yields 1.60–1.79×. At 1M context on one H100, the smaller ~44.5 GB NVFP4 weight footprint (vs. Super's ~70 GB) frees enough of the 80 GB HBM budget to host **8 concurrent 1M-token requests instead of 1**.

## Key points

**Architecture delta (Table, Sec. 2):**

| Quantity | Super | Puzzle-75B-A9B | Ratio |
|---|---|---|---|
| Total parameters | 120.7B | 75.3B | 62.4% |
| Active parameters | 12.8B | 9.3B | 73.1% |
| Mamba SSM state size | 128 | 96 | 75% |
| MoE routed-expert intermediate size | 2688 | 1280–2688 | mean 59.9% |
| Activated routed experts per token | 22 | 4–18 | mean 50% |
| Active routed-expert capacity (relative) | 100% | 8.7%–62.3% | mean 30.9% |

**Interactive-serving Pareto throughput** (Table 7; single 8×B200 node, NVFP4 weights / FP8 KV / FP16 Mamba state, single-step decoding, no MTP):

| Scenario | UT target | Super (tok/s) | Puzzle (tok/s) | Boost |
|---|---|---|---|---|
| 50K / 2K (prefill-heavy) | ≥100 | 5,128 | 8,210 | 1.60× |
| 50K / 2K | ≥125 | 3,784 | 6,412 | 1.69× |
| 50K / 2K | ≥150 | 2,532 | 4,523 | 1.79× |
| 8K / 64K (decode-heavy) | ≥100 | 20,939 | 42,601 | 2.03× |
| 8K / 64K | ≥125 | 13,074 | 27,918 | 2.14× |
| 8K / 64K | ≥150 | 8,522 | 18,047 | 2.12× |

**Accuracy vs. parent** (Table 3, BF16 / NVFP4; Puzzle vs. Super):
- MMLU-Pro 82.4 / 82.2 vs. 83.8 / 83.5
- AIME25 (no tools) 89.7 / 89.9 vs. 92.2 / 89.9; HMMT Feb25 93.4 / 92.9 vs. 94.2 / 93.7
- GPQA (no tools) 78.6 / 78.0 vs. 80.5 / 79.7; LiveCodeBench 81.1 / 79.9 vs. 82.1 / 81.5; SciCode 40.6 / 40.3 vs. 42.3 / 41.5; HLE 16.5 / 15.7 vs. 18.5 / 17.9
- Long context: RULER 256k 95.1 / 95.3 vs. 96.7 / 96.1; RULER 512k 94.2 / 94.8 vs. 95.7 / 95.3; **RULER 1M 92.2 / 93.2 vs. 93.9 / 93.8** — within 1–2 points
- Multilingual: MMLU-ProX 77.5 / 76.5 vs. 79.5 / 79.0; WMT24++ 85.2 / 85.1 vs. 86.8 / 86.6
- **Largest gaps** are on instruction-following and agentic evals — Arena-Hard-V2 (68.6 / 69.0 vs. 72.8 / 72.6) and specific TauBench domains, which the authors call out as most sensitive to aggressive compression and low-precision deployment.

- **Throughput/accuracy tradeoff at UT=100** (Table 4): Super single-step 71.93% acc, 20,939 TPS (1.00×); Puzzle single-step 70.74% acc, 42,601 TPS (**2.03×**, 2.16× request/min after verbosity adjustment); Puzzle+MTP 96,997 TPS (4.63×, 4.91× req/min); Nemotron-3-Nano 60.55% acc, 122,308 TPS (5.84× raw but only 3.87× req/min because of 1.60 verbosity). The lesson the authors draw: judge deployment efficiency by *effective request completion rate*, not token throughput.
- **MTP:** continued MTP training improves average acceptance length ~25–30% over Super — 4.34 (BF16) / 4.31 (NVFP4) vs. Super's 3.45 / 3.31 on SPEED-Bench at draft length 7. Unlike Super, the compressed model's NVFP4 checkpoint barely degrades MTP acceptance relative to BF16.
- **1M context on one H100:** Super's NVFP4 weights ~70 GB + ~4 GB KV per 1M-token request saturate the 80 GB budget → concurrency 1. Puzzle's ~44.5 GB weights, with the parent's attention layout preserved (so same ~4 GB/request KV cost), fit 8 requests. At that concurrency it delivers ~4× Super's aggregate decode throughput (~400 tok/s at bs=8, 20.1 ms median ITL, vs. ~94 tok/s at bs=1, 10.7 ms TPOT); prefill of a 990K-token prompt is ~1.2× faster.
- **Iterative vs. single-shot Puzzle ablation** (Table 6): the three-stage iterative procedure improves the unweighted benchmark average by **0.57 points**, with gains on MMLU-Pro, GPQA, HLE, AA-LCR, LiveCodeBench, SciCode, and RULER-256K.
- **Recovery breakdown:** short-context KD is what restores general accuracy after the final Puzzle iteration, bringing most benchmark categories to **over 97% of Nemotron-3-Super**; long-context KD helps long-input/long-generation benchmarks; the authors report "the impact of RL training in our experiments was small."

> [!warning] Internal numerical inconsistency (v2)
> The prose in Sec. 3.1 quotes throughput multipliers that do not match Table 4 on the same page: text says 2.18× (Puzzle single-step), 4.85× (Puzzle+MTP), 3.57× (Super+MTP), 7.94× raw / 5.26× req-rate (Nano), and 5.14× (Puzzle+MTP req-rate); Table 4 lists 2.03×, 4.63×, 3.04×, 5.84× / 3.87×, and 4.91×. Similarly, the introduction states the design target as "a 2× server throughput improvement... at 100 TPS," while the Table 7 caption states "our design target was a 1.6× boost at UT=100 tok/s." Table values are used above.

## Concepts & entities

- [[nemotron-3-super-nvidia-2026]] — the parent/teacher model being compressed (120.7B/12.8B); Puzzle-75B-A9B inherits its attention layout and shared-head MTP.
- [[mixture-of-experts]] — the compression is almost entirely MoE-side: heterogeneous expert-channel pruning plus per-layer top-k reduction from k=22 down to a 4–18 range.
- [[state-space-model]] — Mamba layers are pruned uniformly from a 128-channel to a 96-channel SSM state because cache IO dominates Mamba decode time at batch ≥64.
- [[quantization]] — FP8 (Hopper) and NVFP4 (Blackwell) post-training quantization checkpoints, with a per-operator precision policy; NVFP4 is used even on H100 (where it is not native) because HBM capacity is the binding constraint at 1M context.
- [[quantization-performance]] — the paper is essentially a serving-throughput study: Pareto frontiers over (TP, EP, batch size) at matched precision, reporting total throughput under user-throughput constraints.
- [[kv-cache]] — attention layers were deliberately left unpruned since the parent is already KV-cache efficient; the 1M-context concurrency win comes from weight footprint, not KV reduction (~4 GB per 1M-token request either way).
- [[long-context-llm]] — RULER at 256k/512k/1M held within 1–2 points of the parent; the 1M single-GPU concurrency result is the headline long-context claim.
- [[agent-evaluation]] — evaluated on Terminal Bench (hard subset), SWE-Bench (OpenHands), and TauBench V2 (Airline/Retail/Telecom); agentic benchmarks show the largest compression sensitivity.
- [[transformer-architecture]] — Puzzle is a decomposed NAS over per-layer "puzzle pieces" solved as a mixed-integer program, a layer-level architecture search rather than uniform scaling.

## References

- [arXiv:2607.04371](https://arxiv.org/abs/2607.04371)
