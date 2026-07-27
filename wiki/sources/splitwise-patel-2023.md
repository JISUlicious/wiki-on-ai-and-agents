---
title: "Splitwise: Efficient generative LLM inference using phase splitting"
type: source
created: 2026-07-27
updated: 2026-07-27
sources:
  - splitwise-patel-2023.md
arxiv_id: "2311.18677"
year: 2023
authors:
  - Pratyush Patel
  - Esha Choukse
  - Chaojie Zhang
  - Inigo Goiri
  - Ricardo Bianchini
tags:
  - 2023
  - paper
status: complete
importance: high
---

# Splitwise: Efficient generative LLM inference using phase splitting

**Pratyush Patel, Esha Choukse, Chaojie Zhang, Aashaka Shah, Inigo Goiri, Saeed Maleki, Ricardo Bianchini** (University of Washington; Microsoft) — [arXiv:2311.18677](https://arxiv.org/abs/2311.18677). No venue in the arXiv comments ("12 pages, 19 figures"; v1 Nov 2023, v2 May 2024).

## Summary

Splitwise reaches the same conclusion as [[distserve-zhong-2024]] — put prompt computation and token generation on separate machines — but from a **datacenter provisioning** angle rather than an SLO-scheduling one. The paper is built on a characterization of BLOOM-176B and Llama-70B running on vLLM over **production Azure traces** (coding and conversation services, captured 2023-11-11 and partially released in the AzurePublicDataset repo). The two services look completely different: the coding trace has a median of **1020 prompt tokens** but only **13 output tokens**, while conversation is nearly bimodal with a median of **129** generated tokens.

The characterization yields seven insights, and the operationally decisive ones are about hardware fit. The prompt phase is compute-bound: it uses the GPU's power budget efficiently and its latency degrades sharply under a power cap. The token phase is memory-bandwidth-and-capacity bound: its power draw barely varies with batch size, and **power-capping a token machine from 700 W to 350 W (over 50%) has almost no latency impact**. Batching behaves oppositely too — prompt-phase throughput *declines* past ~2048 batched prompt tokens, whereas token-phase throughput keeps rising with batch size up to 64, where the machine runs out of memory. Meanwhile A100→H100 buys 3.43x the FLOPs but only 1.64x the HBM bandwidth and *the same* 80 GB capacity, at 2.16x the machine cost and 1.75x the power. So the token phase — which is most of a request's end-to-end time (Insight III) — is paying for compute it structurally cannot use. Mixed continuous batching does not save this: 60–70% of conversation time is spent with 20 or fewer active tokens in the batch, and the coding trace runs with a *single* active token more than 20% of the time.

Splitwise therefore maintains three machine pools — **prompt**, **token**, and a **mixed** pool that expands and contracts to absorb load spikes and prevent fragmentation — under a two-level scheduler (cluster-level for routing and pool management, machine-level for batching). Prompt machines produce the first token and stream the KV cache to the assigned token machine over InfiniBand. The transfer overhead is attacked by **layer-wise, asynchronous KV transfer**: as each layer's KV is produced during prompt computation, its transfer is kicked off while the next layer computes, hiding most of the latency. Net cost is small: transfer is under 7% of prompt computation time, with a constant non-overlapped residue of ~8 ms (A100) / ~5 ms (H100); end-to-end it costs **0.8%** of E2E latency versus 3% for serialized transfer, visible to the user only as a **16.5%** slower *second* token (versus 64% with serialized transfer).

The payoff is that prompt and token pools can use *different hardware*. The paper evaluates four designs named by (prompt machine, token machine): **Splitwise-AA** (DGX-A100 both), **Splitwise-HH** (DGX-H100 both), **Splitwise-HA** (H100 prompt, A100 token), and **Splitwise-HHcap** (H100 both, token machines power-capped to 70%). Cluster sizing is found by simulator search against nine SLOs (P50/P90/P99 on TTFT, TBT, and E2E).

## Key points

| Result | Number |
|---|---|
| Abstract headline | up to **1.4x** higher throughput at 20% lower cost; or **2.35x** more throughput at the same power and cost |
| Conclusion headline | **1.76x** better throughput with 15% lower power at the same cost; or **2.35x** better throughput at the same cost and power |
| Iso-power, conversation trace (vs. Baseline-A100) | Splitwise-AA: **2.15x** throughput at same power and cost; Splitwise-HA: **1.18x** throughput at 10% lower cost, same power |
| Iso-cost, throughput-optimized | Splitwise-AA: **1.4x** throughput vs. Baseline-H100, at 25% more power and 2x the rack space |
| Iso-throughput, power-optimized | Splitwise-HHcap: same throughput as Baseline-H100 at **25% lower power**, same cost and space |
| Iso-throughput, cost-optimized | Splitwise-AA: same throughput as Baseline-H100 at **25% lower cost** |
| Token-phase power sensitivity | capping 700 W → 350 W: almost **no** latency impact (prompt phase: substantial) |
| A100 vs. H100 | 3.43x FLOPs, 1.64x HBM bandwidth, **1.00x** HBM capacity, 1.75x power, 2.16x cost |
| KV transfer cost | <7% of prompt compute; **0.8%** of E2E; +16.5% on second-token latency (vs. 64% serialized) |
| Batching reality under mixed continuous batching | conversation: 60–70% of time with ≤20 active tokens; coding: >20% of time with a single active token |
| Prompt-phase throughput | declines beyond ~2048 batched prompt tokens (< batch size 2 at median prompt sizes) |
| Token-phase throughput | keeps rising with batch size to 64, then OOM |
| Production trace medians | coding: 1020 prompt / 13 output tokens; conversation: 129 output tokens |
| Phase cost comparison | BLOOM-176B: prefill of 1500 input tokens ≈ generation of only **6** output tokens |

Pool sizing reflects workload shape: for coding (prompt-heavy) Splitwise-HH provisions 35 prompt / 5 token machines; for conversation (generation-heavy) it provisions 25 prompt / 15 token. Splitwise-HHcap wins outright on the conversation trace because its long token phases are exactly what a cheap, power-capped token pool is good at, while Splitwise-HA is the best fit for coding, where TTFT matters and only H100s can keep prompt latency low.

## Concepts & entities

- [[kv-cache]] — the state shipped from prompt machine to token machine; layer-wise async transfer is the paper's key systems trick for making the split cheap.
- [[vllm]] — the serving engine used for all characterization runs and as the batching baseline.
- [[paged-attention]] — the memory-management layer underneath their baseline; orthogonal to phase splitting.
- [[chunked-prefill]] — the colocated alternative; Splitwise argues mixed continuous batching leaves machines running tiny batches most of the time and cannot fix the hardware mismatch between phases.
- [[distserve-zhong-2024]] — concurrent, independent work with the same disaggregation thesis; DistServe optimizes SLO goodput and parallelism search, Splitwise optimizes cluster cost/power with heterogeneous machine pools.
- [[transformer-architecture]] — the compute-bound prompt / memory-bound generation split follows from autoregressive decoding.
- [[quantization-performance]] — the paper notes KV-cache compression as an alternative way to cut the transfer bandwidth it depends on.
- [[long-context-llm]] — longer prompts increase KV transfer size, which the layer-wise overlap is designed to absorb.

## References

- [arXiv:2311.18677](https://arxiv.org/abs/2311.18677)
