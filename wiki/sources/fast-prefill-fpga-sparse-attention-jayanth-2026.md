---
title: "FAST-Prefill: FPGA Accelerated Sparse Attention for Long Context LLM Prefill"
type: source
created: 2026-09-09
updated: 2026-09-09
sources: []
arxiv_id: "2602.20515"
authors:
  - Rakshith Jayanth
  - Viktor Prasanna
year: 2026
venue: null
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# FAST-Prefill: FPGA Accelerated Sparse Attention for Long Context LLM Prefill

**Rakshith Jayanth, Viktor Prasanna** (Ming Hsieh Dept. of Electrical and Computer Engineering, University of Southern California) — [arXiv:2602.20515](https://arxiv.org/abs/2602.20515), cs.AR, submitted 24 Feb 2026, v1 only, CC BY 4.0. **No venue is stated** in the paper.

## Problem

This is the hardware counterpart of [[sparse-prefill]], and it starts from the same observation that drives that page's decision rule: past a certain context length the attention term dominates prefill, so the lever is to *not compute most of it*. But the authors add a second-order point that the algorithm papers gloss over — **dynamic sparse attention makes prefill memory-bound, not compute-bound**. Two reasons: (1) sparse *index generation* — deciding which blocks each query attends to — is a data-dependent control flow with low compute intensity, so it under-utilises SIMT lanes while saturating memory bandwidth, and on GPU much of it is offloaded to the CPU; (2) unlike dense attention where every query reads the whole [[kv-cache]], sparse attention gives each query a *different* subset of KV blocks, destroying data reuse.

Once the kernel is memory-bound rather than FLOP-bound, a GPU's headline TOPS stops being the binding constraint (the [[llm-inference-roofline-yuan-2024|roofline]] framing), and a device with a *programmable* memory hierarchy becomes competitive despite having far less arithmetic. That is the whole bet of this paper: an FPGA with **5.4 TOPS beating a GPU with 222 TOPS** — 41× less peak compute — because the work is about scheduling bytes, not issuing multiplies. Claimed as the first FPGA accelerator for long-context prefill with *dynamic* sparsity; prior FPGA LLM work targeted decode, weight sparsity, or contexts under ~1K tokens.

## Mechanism

FAST-Prefill implements the **FlexPrefill** sparse-index algorithm (Lai et al., ICLR 2025) in hardware unchanged — per attention head, compute a Jensen-Shannon divergence between block-pooled and true-pooled attention, compare against τ=0.1, and dispatch the head to either a *query-aware* pattern or the conservative *vertical-slash* fallback (the same vertical-slash family that [[minference-dynamic-sparse-attention-jiang-2024|MInference]] introduced). Block size B = 128 tokens; the design also adopts [[chunked-prefill]] with a 128-token chunk to match. Three hardware contributions:

| Unit | Problem it solves | Mechanism |
|---|---|---|
| **Sparse Index Generation Unit (SIGU)** | naive index generation materialises **~4 GB** of intermediates at 128K context (>2 GB for Q̂Kᵀ alone, ~2 GB more from softmax/pooling), and reads Key blocks in head-dependent random order | Key blocks streamed **strictly in ascending block order, independent of head**, so HBM sees long contiguous bursts; per-block vertical and slash statistics are accumulated incrementally, collapsing a B×S tensor into a length-⌈S/B⌉ vector. Turns generate-then-aggregate into **stream-and-accumulate: ~4 GB → ~4 KB**. Top-k replaced by a streaming coverage selector (comparator trees, no global argsort). |
| **Sparse Attention Unit (SAU) + liveness-driven dual-tier cache** | KV access is neither streaming nor temporal, so prefetch timing is critical and GQA sharing is wasted | Execution is re-ordered **block-major** (iterate KV blocks, not query blocks); the index set is bucketised into a job list mapping each KV block → its consumers (head, query-block). A **block-use counter is an exact remaining-use count**, enabling a provably-safe *evict-on-nil* rule. The 16 MB URAM cache is split **Hot / Cold** with admission threshold T_hot = 50% of total query blocks, to stop moderately-reused blocks thrashing heavily-reused ones. Out-of-order partial outputs are handled by keyed accumulation into a banked accumulator rather than a reorder network. |
| **Hybrid Matrix Processing Unit (MPU)** | DSPs alone cap the design at ~six 32×32 systolic arrays on a U280 | INT8×INT8 is decomposed into nibbles (Eq. 8), each INT4×INT4 term realised in **LUTs via bit-plane AND-and-shift**, accumulation on FPGA carry chains. Result: **six 32×32 DSP arrays *plus* six 32×32 bit-plane LUT arrays**, INT8 multiply / INT32 accumulate. |

## Measured results

Platforms (Table I) — note the compute asymmetry:

| | NVIDIA A5000 GPU | AMD/Xilinx Alveo U280 FPGA |
|---|---|---|
| Compute units | 8,192 CUDA cores | 9,024 DSP48s |
| Frequency | 1,695 MHz | **175 MHz (achieved)** |
| Peak TOPS | 222 | **5.4** |
| Memory | 24 GB | 8 GB HBM + 32 GB DDR |
| Bandwidth | 768 GB/s | 460 GB/s (HBM) + 38 GB/s (DDR) |

Resource utilisation (Table II): LUT 838k/1,304k (64.3%), FF 1,232k/2,607k (47.3%), BRAM 2,250/4,032 (55.8%), **URAM 912/960 (95%)**, DSP 6,459/9,024 (71.6%). URAM is the binding resource — the cache has no room to grow.

**Headline speedup is stated four different ways in the same paper**, which is worth recording precisely:

| Where | TTFT claim | Energy claim |
|---|---|---|
| Abstract | "average speedup of **up to 2.5×**" | **4.5×** |
| Introduction | "up to **1.2–2.5×**" | 4.5× |
| §V-B2 / §V-B3 | "**1.5× to 2.5×**" | "up to 4.5×" |
| Conclusion | "up to **2×**" | **4×** |

Setup: batch size 1, context 4K / 8K / 16K / 32K / **128K** (64K appears in the accuracy table but not in the listed TTFT sweep), models Llama-3.2-1B-Instruct, Llama-3.2-3B-Instruct and "Qwen2.5-1B-Instruct". Baseline is the official FlexPrefill implementation quantised to INT-8. Energy is reported as Token/Joule with token count = 1 (i.e. it is really joules-per-prefill).

**Accuracy (RULER) — this is the result the abstract does not mention**, and it is the paper's weakest point:

| Model | Method | 4k | 8k | 16k | 32k | 64k | Avg |
|---|---|---|---|---|---|---|---|
| Llama-3.2-1B | FlexPrefill (BF16) | 95.67 | 60.58 | 48.32 | 53.60 | 50.24 | **61.68** |
| Llama-3.2-1B | FlexPrefill (INT8) | 53.61 | 28.12 | 27.14 | 30.12 | 28.22 | 33.44 |
| Llama-3.2-1B | **FAST-Prefill (W8A8)** | 52.14 | 27.33 | 27.12 | 28.50 | 30.30 | **33.07** |
| Llama-3.2-3B | FlexPrefill (BF16) | 93.51 | 79.09 | 74.76 | 73.08 | 68.27 | **77.74** |
| Llama-3.2-3B | FlexPrefill (INT8) | 73.08 | 72.60 | 62.27 | 56.55 | 52.75 | 63.45 |
| Llama-3.2-3B | **FAST-Prefill (W8A8)** | 71.11 | 70.40 | 59.36 | 54.87 | 50.69 | **61.28** |

The paper's claim is narrow and correct: FAST-Prefill matches its *INT-8 baseline* (33.07 vs 33.44; 61.28 vs 63.45). But the INT-8 baseline has itself already lost **46% relative on the 1B model and 21% on the 3B** against BF16. The speedup is therefore measured against a configuration that has given up most of the small model's long-context ability, and the comparison against BF16 GPU inference — the thing a practitioner would actually run — is never made. See [[quantization-quality]]: sub-4B models are exactly where W8A8 is known to hurt most.

**Ablations** (Llama-3.2-3B):

| Ablation | Effect |
|---|---|
| Liveness-driven 16 MB dual-tier cache vs. cacheless | **2.5× TTFT**, from a **65% hit rate** |
| Hybrid MPU vs. DSP-only MPU | **1.8× latency**; without it ~**85% of LUTs sit idle** |

## Does the measurement include host↔device transfer?

**The paper never says, and never names the host interconnect.** This was checked against the full text; "PCIe" does not appear anywhere in the paper, and no bus, DMA bandwidth, or transfer latency is quantified. The only relevant sentence is in §IV-A:

> "The CPU performs tokenization and context embedding on the input prompt and transfers the token embeddings to HBM. Subsequent computation proceeds entirely on the FPGA in a streaming fashion across LLM layers till the generation of the first token."

So the division of labour is stated (host does tokenisation + embedding lookup, then DMAs embeddings; everything else is on-card) but the *cost* of that DMA is not. The methodology sentence is the strongest circumstantial evidence: FPGA latency is measured with **Vitis Analyzer**, a device-side kernel-timeline tool, while the GPU is timed with the **PyTorch timing library**. That asymmetry means the FPGA number is almost certainly device-only while the GPU number is host-observed — but the paper does not confirm it either way, so **treat the 1.5–2.5× as a kernel-level result of unknown end-to-end validity**. The U280 is a PCIe card, so a real deployment pays a host round trip that is nowhere in these figures.

Related and also unstated: **where the model weights live**. The text says computation proceeds "entirely on the FPGA … across LLM layers", and the card has 8 GB HBM + 32 GB DDR against 1B–3B INT8 weights (~1–3 GB), so weights plausibly stay resident — but the paper never states it, never reports weight-load time, and never discusses what happens at a model size that does not fit.

## Limitations

- **No end-to-end / host-transfer accounting**, and no host interconnect named (above).
- **Accuracy is reported only against the INT-8 baseline**, hiding a 21–46% relative RULER loss versus BF16.
- **Four mutually inconsistent statements** of the headline speedup and two of the energy number.
- **Batch size 1 only**, and only 1B–3B models — nothing at the 7B+ scale where prefill actually hurts, and nothing about whether the design survives weights that exceed board memory.
- **No absolute latency or power figures** anywhere: speedups and Token/Joule ratios only, no milliseconds, no watts, no board TDP.
- **URAM at 95%** — the cache that provides the single largest ablation win (2.5×) cannot be scaled on this device.
- The setup names "Qwen2.5-1B-Instruct", which is not a released Qwen2.5 size (0.5B / 1.5B / 3B exist); no Qwen accuracy row is reported.
- Weight sparsity, N:M pruning, and decode-stage optimisation are explicitly out of scope and left as orthogonal future work.

## Related

- [[sparse-prefill]] — the umbrella; this paper is the hardware realisation of the *sparse attention* family, and it inherits that family's Amdahl behaviour: the win requires long contexts.
- [[sparse-attention]] · [[minference-dynamic-sparse-attention-jiang-2024]] — vertical-slash is MInference's pattern; FlexPrefill's contribution is the per-head divergence test that chooses between it and a query-aware pattern.
- [[token-sparse-attention-jo-2026]] — the software counterpart measuring the same regime on A100.
- [[kv-cache]] · [[paged-attention]] — block-major scheduling with a use-count is essentially paged KV management moved into silicon.
- [[chunked-prefill]] — adopted directly, 128-token chunks.
- [[grouped-query-attention]] — GQA block sharing is one of the reuse opportunities the liveness cache is designed to capture.
- [[quantization-performance]] · [[quantization-quality]] · [[quantization]] — W8A8 end-to-end (no dequantise-to-16-bit step) is what makes the LUT bit-plane MPU possible, and is also where the accuracy goes.
- [[llm-inference-roofline-yuan-2024]] — the memory-bound-vs-compute-bound framing that justifies a 5.4 TOPS device beating a 222 TOPS one.
- [[methods-to-speed-up-prompt-processing]] — this is a hardware tier that page does not yet cover.
- [[long-context-llm]] · [[longbench]] — 4K–128K is the target regime.
- [[fpga]] · [[hardware-accelerator]] · [[arithmetic-intensity]] — this source was the wiki's entry point for all three.
- [[nvidia]] · [[llama]] · [[qwen]] — baseline vendor and evaluated model families.
- [[persistent-state-dataflow-accelerator-linear-attention-decode-gupta-2026]] — the decode-side FPGA counterpart; together they cover both phases.
- [[prefill-decode-aware-evaluation-ai-accelerators-usami-2026]] · [[hbm-is-not-all-you-need-wei-2026]] — the phase-vs-hardware framing this result sits inside.

## References

- [arXiv:2602.20515](https://arxiv.org/abs/2602.20515)
- FlexPrefill: Lai et al., *FlexPrefill: A Context-Aware Sparse [[attention-mechanism|Attention Mechanism]] for Efficient Long-Sequence Inference*, ICLR 2025 — the algorithm implemented here.
- RULER: Hsieh et al., [arXiv:2404.06654](https://arxiv.org/abs/2404.06654) — the accuracy benchmark.
- SARATHI: Agrawal et al., [arXiv:2308.16369](https://arxiv.org/abs/2308.16369) — source of the chunked-prefill scheme.
