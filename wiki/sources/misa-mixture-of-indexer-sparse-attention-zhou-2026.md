---
title: "MISA: Mixture of Indexer Sparse Attention for Long-Context LLM Inference"
type: source
created: 2026-09-10
updated: 2026-09-10
sources: []
arxiv_id: "2605.07363"
authors:
  - Ruijie Zhou
  - Fanxu Meng
  - Yufei Xu
  - Tongxuan Liu
  - Guangming Lu
  - Muhan Zhang
  - Wenjie Pei
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# MISA: Mixture of Indexer Sparse Attention for Long-Context LLM Inference

**Ruijie Zhou, Fanxu Meng, Yufei Xu, Tongxuan Liu, Guangming Lu, Muhan Zhang, Wenjie Pei** — [arXiv:2605.07363](https://arxiv.org/abs/2605.07363), cs.LG, v1 8 May 2026. The paper carries **no institutional affiliation line**: the only affiliation given is the code repository `github.com/MuLabPKU/TransArch`, and the corresponding authors are Meng (`fxmeng@stu.pku.edu.cn`) and Pei. [[peking-university|Peking University]] is therefore inferred from the email domain and the lab's GitHub organisation, not stated. A NeurIPS paper checklist is appended, so this is a NeurIPS 2026 submission; no acceptance is recorded.

> [!important] What this paper is, in one paragraph
> A **training-free, inference-time retrofit of the DSA lightning indexer** in models that were *already trained* with [[deepseek-v3.2-sparse-attention-deepseek-2025|DeepSeek Sparse Attention]] — DeepSeek-V3.2 and GLM-5. "Mixture of indexer" means the indexer's own H^I heads (64 on V3.2, 32 on GLM-5) are treated as a pool of [[mixture-of-experts|MoE]] experts: a parameter-free router, computed on mean-pooled key blocks of 1,024 tokens, picks the **h = 8 heads** most relevant to each query token, and only those heads run the per-token scoring scan. Nothing about the main attention, the token budget (k = 2,048), or the model weights changes. With no training at all it matches the full 64-head indexer on LongBench within 0.20 points on V3.2 (50.85 vs 51.05) and beats it on GLM-5 (46.43 vs 46.01), keeps Needle-in-a-Haystack fully green to 128K, and the TileLang kernel runs **~3.82× faster than DSA's indexer kernel** on one H200 — a kernel-only number, by the authors' own limitation section; there is no prefill/decode split and no end-to-end figure.

## Where it sits in the lineage

The [[trained-sparse-attention]] page files methods by how the selector enters the model: [[native-sparse-attention-yuan-2025|NSA]] from scratch, [[moba-mixture-of-block-attention-lu-2025|MoBA]] by fine-tune, DSA by ~1T tokens of continued pre-training. MISA is none of these. It is a **retrofit that only applies to models already in the trained column**: it takes a DSA-trained indexer and skips 56 of its 64 heads per query. It cannot make a dense model sparse, and it does not touch the training-free retrofits on [[sparse-attention]] / [[sparse-prefill]] ([[minference-dynamic-sparse-attention-jiang-2024|MInference]] and kin), which attack the *main* attention of a dense model. The paper's own vocabulary calls DSA "inference-time sparse attention", which conflicts with this wiki's filing (DSA was adapted in with 943.7B sparse-stage tokens); read "inference-time" in the paper as "the selection happens at inference", not "no training was needed".

## The problem: the indexer is the cost DSA left behind

Under DSA the main attention runs over only k = 2,048 tokens, but the lightning indexer still scores **every** prefix token with **every** indexer head: per query O(H^I·L), per layer O(H^I·L²) over a prefill. DeepSeek-V3.2 uses H^I = 64 indexer heads even though its 128 main-attention heads all share one selected token set (sparse MLA runs in MQA mode with one latent entry per token), because head diversity is what makes the aggregated score I(t,s) = Σ_j w(t,j)·ReLU(q(t,j)·k(s)) good; the authors report "measurable retrieval degradation as H^I shrinks". Two prior attacks on this cost keep every head active: **HISA** (hierarchical block-to-token search, block size 128, top-m blocks then per-token refinement inside them) and **IndexCache** (a few layers compute the indexer, the rest reuse their top-k). MISA's claim is that the head axis is orthogonal to both and can be stacked on either.

## Mechanism

**Block-pooled router (Eqs. 7–8).** Partition the prefix into M = ⌈L/B⌉ blocks with **B = 1,024** (HISA's B is 128 — MISA's is deliberately "an order of magnitude larger", since it only has to decide *which heads*, not *which regions*). Mean-pool each block's indexing keys to k̃_b. For query t, compute per-head, per-block affinities with the same gating weights DSA already uses:

`A(t,j,b) = w(t,j) · ReLU(q(t,j) · k̃_b)`,  `E(t,j) = (1/M) Σ_b |A(t,j,b)|`,  `H_t = TopK_j(E(t,j), h)`.

HISA reduces the same A tensor *across heads* to get a per-block score; MISA reduces it *across blocks* to get a per-head importance. Router cost is O(H^I·M) per query — at 128K with B = 1,024 that is M = 128 pooled keys, one 64×128 matmul per query. The router has **no parameters and no training**: it reuses the DSA indexer's own q^I, w^I and pooled k^I.

**Sparse token scoring with active heads (Eqs. 9–10).** Only the h routed heads compute the per-token score Î(t,s) = Σ_{j∈H_t} w(t,j)·ReLU(q(t,j)·k(s)); top-k over Î picks the 2,048 tokens fed to the unmodified sparse MLA. Per-query cost falls from O(H^I·L) to **O(h·L + H^I·M)**. With h = 8 that is an 8× cut on V3.2 (64 → 8) and 4× on GLM-5 (32 → 8). Routing is **per query token**; every main-attention head then sees the same token set, as in DSA. The authors' argument for why MoE-style routing works here and not for attention outputs (MoH, Mixture-of-Attention-Heads): the routed heads only decide *which similarity patterns are consulted for selection*; the attention that writes into the [[residual-stream|residual stream]] stays dense over the chosen set, so a very small h is survivable.

**MISA† (Eqs. 11–12).** A two-stage variant: the routed pass keeps an enlarged candidate set of **k′ = 8,192** tokens, then the *full* 64-head DSA indexer re-scores only those candidates and keeps the top 2,048. Cost O(h·L + H^I·M + k′·H^I). Unlike HISA the coarse pass stays at token granularity — no block is kept or dropped whole — which is why its candidate set has higher recall of DSA's true top-k (Appendix A). (Figure 3's caption says k′ = 8,912; every other mention says 8,192.)

## Setup

| | |
|---|---|
| Models | DeepSeek-V3.2 (H^I = 64 indexer heads, H = 128 main heads, single shared KV entry); GLM-5 ([[z-ai\|Z.ai]]; H^I = 32) — both "natively support DSA" |
| Defaults | h = 8 in both prefill and decode; B = 1,024; k = 2,048; MISA† k′ = 8,192 |
| Baselines | **DSA** (all heads); **Block-Sparse** — a MoBA-style selector, 128-token blocks scored by mean indexing key, m = k/B blocks kept; **HISA** (B = 128) |
| Training | **none** — "all sparse methods are applied at inference time without any additional training" |
| Hardware | 8× NVIDIA H200 for evaluation; kernel timing on a single H200; kernel written in TileLang |
| Provenance | the DSA, Block-Sparse and HISA rows of Table 1 are **copied from the HISA paper**; MISA rows and everything else are the authors' own runs |

## Results

### LongBench (Table 1), six category averages

| Model | Indexer | Heads | SQA | MQA | Sum | FS | Syn | Code | **Avg** |
|---|---|---|---|---|---|---|---|---|---|
| DeepSeek-V3.2 | DSA | 64 | 50.89 | 52.66 | 22.11 | 62.24 | 69.83 | 48.56 | **51.05** |
| | Block-Sparse | 64 | 48.36 | 49.76 | 21.90 | 59.45 | 68.67 | 49.09 | 49.54 |
| | HISA | 64 | 49.17 | 51.96 | 22.13 | 61.62 | 70.83 | 48.99 | 50.78 |
| | MISA | 8 | 50.83 | 51.62 | 22.31 | 61.77 | 70.00 | 48.54 | 50.85 |
| | MISA† | 8 | 50.91 | 52.27 | 22.04 | 62.10 | 69.83 | 48.45 | 50.95 |
| GLM-5 | DSA | 32 | 41.23 | 27.89 | 18.39 | 63.20 | 68.84 | 56.53 | 46.01 |
| | Block-Sparse | 32 | 38.35 | 24.29 | 16.95 | 60.64 | 60.49 | 55.29 | 42.67 |
| | HISA | 32 | 42.45 | 27.62 | 17.90 | 63.78 | 69.35 | 56.79 | 46.32 |
| | MISA | 8 | 41.64 | 29.16 | 18.22 | 63.53 | 68.76 | 57.24 | 46.43 |
| | MISA† | 8 | 41.16 | 28.67 | 18.56 | 63.57 | 69.26 | 57.83 | **46.51** |

MISA is within 0.20 of DSA on V3.2 and above it on GLM-5; MISA† is within 0.10 on V3.2 with every category within 0.4. The abstract's "within 0.5 average points" is the loose version of the same numbers. Block-Sparse — the MoBA-style baseline — trails DSA by 1.51 (V3.2) and 3.34 (GLM-5) points with the full head pool, the paper's evidence that block-uniform selection is what hurts, not head count. Note where MISA *does* lose to DSA on V3.2: Multi-Doc QA (51.62 vs 52.66) and Few-shot (61.77 vs 62.24); MISA† recovers most of it.

### Needle-in-a-Haystack (Figure 2, DeepSeek-V3.2, up to 128K)

Heatmaps only, no table. DSA, MISA and MISA† are described as a near-perfect green grid across depth × length; Block-Sparse shows "visible accuracy holes at intermediate depths once the context exceeds ~32K"; HISA "minor degradations at the deepest needle positions", where a top-m block filter can drop the needle's block.

### Selection fidelity (Appendix A)

Per-layer IoU between the selected set and DSA's own top-2,048, on the long subset of LSHT (10 examples, all layers, V3.2), positions ≥ 2,048: **MISA† retains > 92% of DSA's tokens in every layer** and beats HISA at every length and layer. This is the two-stage variant; the single-stage MISA's IoU is not reported.

### Indexer kernel speed (Figure 3, single H200)

Latency vs prefix length, indexer kernel only. Single-stage MISA is faster than DSA's kernel "across the entire sequence-length range"; **MISA† only beats DSA past 32K** (its second stage costs a fixed k′·H^I). The headline is "approximately a 3.82× speedup over DSA's original indexer kernel for long contexts" — the length at which 3.82× is measured is not stated in text. The 8× head reduction does not appear in wall-clock because of memory traffic, load imbalance from the routed head set, and router overhead. The paper calls this "3.82× end-to-end speedup" in §5.3 and then, in §7, says the opposite: "the speed experiments only measure the latency of the TileLang kernel rather than the end-to-end latency of the full model." The limitation section is the accurate one.

### Ablations (all on DeepSeek-V3.2, NIAH at 128K)

- **Active heads h ∈ {1, 2, 4, 8, 16}** (Fig. 4): h = 1, 2 leave visible holes; h = 4 is "suboptimal, particularly at 128K"; h = 8 is "essentially indistinguishable" from 64 heads; h = 16 adds nothing at twice the cost. For MISA† (Fig. 6), h = 1 matches single-stage h = 2, and h = 2 matches single-stage h = 4 — the re-rank stage tolerates under-routing.
- **Router score** (Fig. 7): the gating weight w(t,j) alone and ‖q(t,j)‖₂ alone — signals that never look at the prefix — both collapse on the hard regions; the block-pooled score is the only one that recovers DSA.
- **Block size B** from 128 to 131,072 (a single global pooled key), eleven values (Fig. 8): accuracy "largely insensitive"; only the largest B degrade the deepest needles.

## Limitations and what the numbers do not say

The authors list three: (i) kernel-only timing, no full-model latency; (ii) MISA reduces the indexer's *compute*, "it does not reduce the memory access volume to the KV cache in that stage" — every indexing key is still read; (iii) no fine-tuning — jointly training the router with the indexer "should further close the residual gap".

> [!warning] Kernel-only, and the kernel is a small share of the model
> No prefill speedup, no decode speedup, no TTFT, no throughput: the only timing is the indexer kernel in isolation, and the paper never separates prefill from decode (h = 8 is simply used for both). The indexer is also still **O(L²)** — MISA shrinks its constant by up to 8× and leaves its scaling alone, so the wiki's standing note on [[deepseek-v3.2-sparse-attention-deepseek-2025|DSA]] (the indexer is the quadratic term DSA kept) still holds. How much of a 128K prefill the indexer actually is, the paper does not say; it calls the indexer "the dominant cost on long contexts", which is true *within the attention path* after DSA has cut the main attention to O(L·k). This wiki's back-of-envelope, not the paper's: per query at the end of a 128K prefill the V3.2 indexer is 64 heads × 128 dims × 131,072 keys ≈ 1.07 GMAC, the sparse MLA over 2,048 latent entries ≈ 0.29 GMAC, and the model's ~37B active parameters cost ≈ 37 GMAC per token — so the indexer is roughly 3% of per-token compute at the end of a 128K prefill (less averaged over it) and reaches ~10–20% only around 1M tokens. A 3.82× indexer kernel is therefore a low-single-digit end-to-end prefill saving at 128K on this model. The [[arithmetic-intensity]] question for decode — whether skipping heads changes bytes moved — is answered negatively by limitation (ii).

Two smaller provenance notes. The baseline rows in Table 1 are borrowed from the HISA paper rather than re-run, so DSA-vs-MISA there is a cross-paper comparison. And "GLM-5" here is the base model with H^I = 32 indexer heads; it is not the wiki's [[glm-5-2]] page, whose IndexShare (one indexer per four layers) is an IndexCache-style *layer-axis* saving that MISA's head-axis saving would compose with.

## Related

- [[trained-sparse-attention]] — the lineage this paper retrofits onto; MISA is a training-free layer on top of the trained column, not a member of it.
- [[deepseek-v3.2-sparse-attention-deepseek-2025]] — the indexer being routed; [[flashmemory-lookahead-sparse-attention-wang-2026]] — DeepSeek-V4's successor, which the introduction describes as "DSA applied on top of a 4× compressed (block-level) KV stream".
- [[native-sparse-attention-yuan-2025]] · [[moba-mixture-of-block-attention-lu-2025]] — the Block-Sparse baseline is a MoBA-style selector; the paper's reading is that block granularity, not head count, is what loses on retrieval.
- [[gated-sparse-attention-shen-2026]] — the other 2026 DSA descendant on this wiki: a from-scratch architecture that changes the indexer's activation, where MISA changes nothing and trains nothing.
- [[sparse-attention]] · [[sparse-prefill]] · [[minference-dynamic-sparse-attention-jiang-2024]] — the training-free retrofits for *dense* models; different target.
- [[mixture-of-experts]] — head-as-expert routing; [[multi-head-latent-attention]] (no page yet) — the MQA-mode sparse MLA that makes one shared token set per query possible; [[grouped-query-attention]].
- [[flash-attention]] · [[kv-cache]] · [[long-context-llm]] · [[arithmetic-intensity]] · [[longbench]]
- [[deepseek]] · [[z-ai]] · [[glm-5-2]] · [[glm-5]] (no page yet) · [[peking-university]]

## References

- [arXiv:2605.07363](https://arxiv.org/abs/2605.07363) — v1, 8 May 2026 (the version read here). Code: `github.com/MuLabPKU/TransArch`.
- DeepSeek-V3.2 indexer configuration (`index_n_heads: 64`, `index_head_dim: 128`, `index_topk: 2048`) confirmed against the released model config, both V3.2-Exp and V3.2.
