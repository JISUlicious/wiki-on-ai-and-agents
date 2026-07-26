---
title: "Can Language Models Actually Retrieve In-Context? Drowning in Documents at Million Token Scale"
type: source
created: 2026-07-22
updated: 2026-07-22
sources:
  - can-lms-actually-retrieve-in-context-gollapudi-2026.md
arxiv_id: "2607.01538"
year: 2026
authors:
  - Siddharth Gollapudi
  - Nilesh Gupta
  - Prasann Singhal
  - Sewon Min
tags:
  - 2026
  - paper
status: complete
importance: high
---

# Can Language Models Actually Retrieve In-Context? Drowning in Documents at Million Token Scale

**Siddharth Gollapudi, Nilesh Gupta, Prasann Singhal, Sewon Min** (UC Berkeley; Nilesh Gupta at UT Austin) — [arXiv:2607.01538](https://arxiv.org/abs/2607.01538), cs.CL, 1 Jul 2026. Preprint; supported by the NVIDIA Academic Grant Program.

## Summary

The first systematic study of **in-context retrieval (ICR)** at the two scales a practical retriever actually demands: **million-token corpora** and **length generalization far beyond training size**. ICR casts retrieval as conditional generation — put the whole corpus in context and have the model decode the identifier of the relevant document — which would collapse the two-stage RAG pipeline into one model and allow retrieval behavior beyond inner-product similarity. Prior work either used proprietary systems without controlled evaluation or studied reranking over small candidate pools, so corpus-scale ICR was untested.

The paper introduces **BLOCKSEARCH**, a **0.6B** LM retriever (Qwen3-0.6B backbone, trained on 8× A100s) that keeps the block-sparse-attention ICR architecture and adds three things for scale: randomized per-document identifier codes (so the model can't overfit code to position or semantics), in-batch negative training (one prefill of b×16 documents yields b training signals), and an **on-policy auxiliary loss** that trains on the model's own rolled-out digit prefixes to fix exposure bias. Every model trains with only 256 documents in context, yet BLOCKSEARCH length-generalizes **~10×** beyond that — matching a dense retriever at moderate corpus sizes (95.8% vs. 95.5% Recall@1 on MS MARCO at ~45k tokens) — before collapsing beyond roughly 500,000 tokens.

The diagnosis of that collapse is the paper's core contribution: it is **not a ranking failure but a readout failure caused by attention dilution**. The "attention ceiling" — the fraction of queries where at least one head at layer 19 ranks the gold document first by pre-softmax QK similarity — stays at ~100% across the entire N sweep, even at million-token scale. What collapses is the *normalized* mass: as N grows, irrelevant documents dominate the softmax denominator. Decomposing the L19 attention output shows total magnitude shrinks only ~36% from N=500 to N=10k while the gold-driven share falls from **0.91 to 0.01** — the layer keeps writing at full amplitude, but writes an average of distractors instead of the gold. Two length-aware fixes follow: **SSMax** (scale pre-softmax scores by log N, cancelling the (N−1) growth in the denominator) and **document-level top-B routing** at layer 16. Together they lift million-token MS MARCO Recall@1 from 0.2% to 20.5%, recovering the gap to dense retrieval, and beat the concurrent MSA-4B at 1/7 the parameters.

## Key points

**Setup.** Corpus of N documents, ≤300 tokens each, prefilled into the KV cache with a block-sparse mask (documents attend causally within their own block; the query block attends over the full corpus). RoPE positions reset at each document start; query shifted to position 300. Model decodes a four-digit code in {0,…,9999}; metric is Recall@1 via beam search over the digit sequence. Trained on RLHN (relabeled-hard-negative BEIR), ~100M tokens total, 1 positive + 15 hard negatives per query, **256 documents per training corpus**. Evaluation corpora are built from 400 queries × (gold + 24 hard negatives) = 10,000 documents, making them far harder than prior ICR evaluations.

**Recall@1 (×100) vs. corpus size** (Table 2; NQ caps at N=8,607 to stay under a ~1M-token prefill budget):

| Model | MS MARCO 0.5k | 2.5k | 5k | 10k | HotpotQA 10k | NQ 8.6k |
|---|---|---|---|---|---|---|
| Attention ceiling (R19-any) | 100.0 | 99.8 | 100.0 | 100.0 | 100.0 | 97.0 |
| Qwen3-dense (0.6B) | 95.5 | 52.8 | 38.5 | 20.2 | 79.5 | 39.6 |
| MSA-4B (concurrent, ~7× larger) | 93.8 | 42.2 | 27.5 | 16.0 | 75.5 | 18.6 |
| BLOCKSEARCH | 95.8 | 43.8 | 18.8 | **0.2** | 0.5 | 4.8 |
| + sink | 96.5 | 45.2 | 21.2 | 2.5 | 1.0 | 10.3 |
| + SSMax | 95.5 | 49.8 | 33.8 | 16.5 | 56.8 | 42.6 |
| + routing (top-B=256 at L16) | 96.0 | 50.7 | 38.2 | 18.8 | 78.5 | 48.1 |
| + SSMax + routing | 95.5 | 50.0 | 38.2 | **20.5** | 72.5 | 46.1 |

- **The headline "0.2% → 20.5%"** is MS MARCO at N=10k (~1M tokens): SSMax+routing edges the Qwen3-dense baseline (20.2). SSMax alone is an 82× improvement over no modification at that point.
- **Attention dilution, quantified** (Table 1, MS MARCO, 400 queries): ‖a19‖ falls 47.48 → 30.27 (~36%) from N=500 to N=10k, but the gold share ‖a19_G‖/‖a19‖ falls **0.91 → 0.01**, a factor of ~130.
- **The additive sink barely helps.** A learned constant shifts the denominator uniformly and cannot rescale its N-dependence; it lifts MS MARCO N=10k only 0.2 → 2.5 and *hurts* on LIMIT at every N.
- **Ablations isolate BLOCKSEARCH's own contributions.** The position-coded variant (the prior ICR recipe) collapses to near-zero by N=5,000 on every dataset; removing the on-policy loss costs MS MARCO 18.8 → 15.0 and HotpotQA 13.0 → 6.2 at N=5,000 — so random codes and the on-policy loss contribute distinct gains.
- **vs. MSA-4B:** BLOCKSEARCH matches it at N=500/1k/2.5k on MS MARCO (95.8/75.2/43.8 vs. 93.8/70.2/42.2) at ~1/7 the parameters, though vanilla BLOCKSEARCH loses at large N until the attention fixes are added. Notably MSA-4B reports near-perfect RULER NIAH in its own paper yet degrades sharply on real retrieval — a caution about synthetic long-context benchmarks.
- **LIMIT (out-of-distribution, lexical similarity, 2 golds per query, corpus grown N=46 → 5,000, ~8k → ~850k tokens):**

| Method | N=46 | 500 | 1000 | 2500 | 5000 |
|---|---|---|---|---|---|
| Attention ceiling | 1.000 | 1.000 | 1.000 | 1.000 | 1.000 |
| BLOCKSEARCH | 0.354 | 0.094 | 0.022 | 0.000 | 0.000 |
| + SSMax | 0.439 | 0.215 | 0.141 | 0.067 | 0.054 |
| + SSMax + routing | 0.439 | 0.234 | 0.215 | 0.196 | 0.149 |
| Qwen3-dense | 0.176 | 0.080 | 0.061 | 0.047 | 0.035 |
| Random chance | 0.043 | 0.004 | 0.002 | 0.001 | 0.000 |

- The abstract's "3× higher score" on LIMIT is conservative at large N: the SSMax+routing / dense ratio is 2.9× at N=46 but **4.3× at N=5,000** (0.149 vs. 0.035). BLOCKSEARCH never trained on a lexical task, so this is strict OOD.
- **Honest caveat from the authors:** routing "delays but does not prevent the inevitable" — SSMax+routing is still declining with N (0.149 at ~850k tokens), and routing reintroduces a retrieve-then-read decomposition *inside* the model, "the very structure in-context retrieval is intended to remove." The remaining gap to the near-perfect attention ceiling means dilution, not ranking, is the open bottleneck.

## Concepts & entities

- [[retrieval-augmented-generation]] — ICR is proposed as a replacement for the two-stage RAG pipeline; the paper's own routing fix partially reinstates retrieve-then-read inside the model.
- [[dense-retrieval]] — Qwen3-dense (0.6B, same backbone, contrastive on the same data) is the primary reference point; BLOCKSEARCH matches it at million-token scale on BEIR datasets and beats it ~3–4× on LIMIT.
- [[long-context-llm]] — the core object of study, but the paper argues ICR differs fundamentally from standard long-context modeling (independent documents, not one coherent sequence) and that NIAH/RULER-style synthetic evals overstate real retrieval capability.
- [[attention-mechanism]] — attention dilution / softmax dispersion is the diagnosed failure mode; SSMax (log-N pre-softmax scaling) and length-aware sinks are the interventions.
- [[kv-cache]] — documents are prefilled into the KV cache once; document independence makes parallel encoding and cross-query corpus reuse possible (the paper drops the query prefix from prior ICR work to keep that property).
- [[maximum-inner-product-search]] — the inner-product similarity ceiling ICR is meant to escape; LIMIT is explicitly designed to defeat embedding similarity.
- [[transformer-architecture]] — block-sparse masking, per-document RoPE resets, and layer-16 document routing are all architecture-level modifications to a Qwen3-0.6B backbone.

## References

- [arXiv:2607.01538](https://arxiv.org/abs/2607.01538)
