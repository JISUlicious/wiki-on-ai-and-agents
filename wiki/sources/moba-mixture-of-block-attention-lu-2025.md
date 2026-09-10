---
title: "MoBA: Mixture of Block Attention for Long-Context LLMs"
type: source
created: 2026-09-10
updated: 2026-09-10
sources: []
arxiv_id: "2502.13189"
authors:
  - Enzhe Lu
  - Zhejun Jiang
  - Jingyuan Liu
  - Yulun Du
  - Tao Jiang
  - Chao Hong
  - Shaowei Liu
  - Weiran He
  - Enming Yuan
  - Yuzhi Wang
  - Zhiqi Huang
  - Huan Yuan
  - Suting Xu
  - Xinran Xu
  - Guokun Lai
  - Yanru Chen
  - Huabin Zheng
  - Junjie Yan
  - Jianlin Su
  - Yuxin Wu
  - Neo Y. Zhang
  - Zhilin Yang
  - Xinyu Zhou
  - Mingxing Zhang
  - Jiezhong Qiu
year: 2025
venue: arXiv technical report (Moonshot AI)
status: complete
importance: medium
tags:
  - 2025
  - paper
---

# MoBA: Mixture of Block Attention for Long-Context LLMs

**Enzhe Lu, Zhejun Jiang, Jingyuan Liu, Yulun Du, Tao Jiang, Chao Hong, Shaowei Liu, Weiran He, Enming Yuan, Yuzhi Wang, Zhiqi Huang, Huan Yuan, Suting Xu, Xinran Xu, Guokun Lai, Yanru Chen, Huabin Zheng, Junjie Yan, Jianlin Su, Yuxin Wu, Neo Y. Zhang, Zhilin Yang, Xinyu Zhou, Mingxing Zhang, Jiezhong Qiu** ([[moonshot-ai|Moonshot AI]]; Mingxing Zhang at Tsinghua; Jiezhong Qiu at Zhejiang Lab / Zhejiang University; co-corresponding authors Xinyu Zhou and Jiezhong Qiu) — [arXiv:2502.13189](https://arxiv.org/abs/2502.13189), cs.LG, v1 18 Feb 2025, 15 pages, self-described **Technical Report**; no venue. **Code:** [github.com/MoonshotAI/MoBA](https://github.com/MoonshotAI/MoBA). Posted two days after DeepSeek's [[native-sparse-attention-yuan-2025|NSA]], which it does not cite. **Deployment claim, verbatim from the abstract:** "MoBA has already been deployed to support Kimi's long-context requests."

> [!important] What this paper is, in one paragraph
> Mixture-of-Experts routing applied to the *KV history* instead of the FFN: cut the context into blocks of B tokens, score every block for every query by the dot product of the query with the block's **mean-pooled keys**, keep the top-k blocks (plus, always, the query's own block under a causal mask), and run ordinary FlashAttention over the union. The gate has **no parameters** — it reuses the model's own keys — so MoBA is a drop-in replacement for full attention that can be switched on or off per layer at any point in training. That is its selling point over NSA: it was retrofitted onto **Llama 3.1 8B** by continual pretraining to 1M context (full attention throughout, then 100B tokens with MoBA on in 29 of 32 layers) and lands within noise of the full-attention twin on 16 benchmarks (RULER@128K 0.7818 vs 0.7849), with the attention layer's forward pass **6.5× faster at 1M tokens** and **16× at 10M**. Two caveats the abstract omits: in the reported evaluation MoBA is used **for prefill only** ("we switch to full attention during generation"), and the speedups are attention-layer time, not end-to-end.

## Problem

The stated principle is "**less structure**": let the model decide where to attend rather than imposing sinks or windows ("strongly biased", task-specific), and stay inside the Transformer rather than replacing softmax attention with [[linear-attention]] (Mamba, RWKV, RetNet), which either needs an expensive conversion of a pretrained model or training from scratch, and whose "effectiveness in complex reasoning tasks remains limited". Training-free dynamic sparse methods — Quest, [[minference-dynamic-sparse-attention-jiang-2024|MInference]], RetrievalAttention — cut inference cost but "do not substantially alleviate the intensive training costs of long-context models", which is the cost Moonshot cares about at million-token scale. The design goal that follows: an attention that "transition[s] seamlessly between full and sparse attention modes", so existing checkpoints are reusable and both training and inference get cheaper.

## Mechanism

**Block partition and gate (§2.2, Eqs. 2–6).** A context of length N is split into n = N/B blocks I_i. For a query **q**:

- affinity s_i = ⟨**q**, mean_pool(K[I_i])⟩ — the query against the *average key* of the block;
- gate g_i = 1 if s_i is among the top-k affinities, else 0;
- MoBA(**q**, K, V) = Softmax(**q** K[I]ᵀ) V[I] with I = ∪_{g_i > 0} I_i.

**Causality, two rules.** (1) No routing to future blocks: s_i = −∞ for any block with pos(**q**) < i·B. (2) The **current block** — the one containing the query — is *always* selected (g_i = 1) and attended under a causal mask, because mean-pooling over the whole current block would otherwise leak future keys into the score. The authors liken the always-on current block to *shared experts* in DeepSeekMoE / Qwen-MoE; it also guarantees a local-attention floor, the role NSA's separate sliding-window branch plays.

**No new parameters.** Because the router is the query dotted with pooled keys, MoBA "maintain[s] the same number of parameters without any addition or subtraction". Hence any layer can be full or MoBA at initialisation and the choice "can be dynamically altered during training". Sliding-window attention and attention sinks are special cases (a gate that always picks the most recent blocks, or initial + recent). The related-work section places two training-free methods as degenerate MoBAs: **Quest = MoBA with smaller blocks and a min/max-pooled block representation**; **LongHeads = MoBA with top-1 gating**.

**Selection is per head.** Algorithm 1 computes S = Q K̄ᵀ ∈ ℝ^{N×h×n} and takes top-k along the block axis for every head independently. That is the opposite of NSA's group-summed scores (and of DSA's single shared indexer), and it is the choice that makes MoBA free to retrofit while leaving the GQA-union memory-traffic problem NSA raises unaddressed — the paper never mentions GQA or MQA.

**Fine-grained segmentation.** Borrowing the fine-grained-expert result from MoE, the authors test many small blocks against few large ones at fixed sparsity (below): finer wins.

### Implementation (§2.3, Algorithm 1)

Five steps, assembled from MoE dispatch and [[flash-attention|FlashAttention]]:

1. Compute gating from pooled keys and the causal mask; produce the sparse query→block assignment G = topk(S + M, k).
2. **Re-order queries by assigned block** (the MoE "dispatch").
3. For each block, attend its assigned queries to it with **`flash_attention_varlen`** — historical blocks with `causal=False`; the current-block path separately with `causal=True`.
4. Re-order outputs back.
5. **Combine** a query's per-block partial outputs with **online softmax** (tiling, as in FlashAttention / ring attention), since a query attends to its current block plus several history blocks.

The selector's own cost is Q K̄ᵀ, O(N · n · h · d) — linear in N for a fixed block count and negligible at the tested settings, but the same "cheap selector, then sparse main attention" shape as NSA's compressed branch and DSA's indexer. For 10M-token runs they push tensor parallelism down to the query-head axis, broadcasting K/V across query-head shards.

## Experiments

### Scaling law (§3.1, Table 1, Fig. 3)

Five model pairs (MoBA vs full attention, everything else identical), Chinchilla-style budgets:

| Params | Heads | Layers | Hidden | Tokens | Block | top-k |
|---|---|---|---|---|---|---|
| 568M | 14 | 14 | 1792 | 10.8B | 512 | 3 |
| 822M | 16 | 16 | 2048 | 15.3B | 512 | 3 |
| 1.1B | 18 | 18 | 2304 | 20.6B | 512 | 3 |
| 1.5B | 20 | 20 | 2560 | 27.4B | 512 | 3 |
| 2.1B | 22 | 22 | 2816 | 36.9B | 512 | 3 |

At 8K the sparsity is 1 − 512·3/8192 = **81.25%** (the same paragraph also says "up to 75%" — inconsistent; 81.25% is the arithmetic). Validation-loss gaps stay within **1e-3** across sizes; fitted laws L = 2.625·C^−0.063 (MoBA) vs 2.622·C^−0.063 (full). **Trailing-token loss** at 32K (last 2K tokens, sparsity 95.31%): MoBA 1.546·C^−0.108 vs full 1.464·C^−0.097 — MoBA is *worse* on the long-range tokens at every size, but with a steeper exponent, which the authors read as "the loss gap is progressively narrowing". (Fig. 3's caption says "last 1K tokens"; its table says 2K.)

**Fine-grained segmentation ablation** (1.5B, 32K, sparsity fixed at 75%): 8 / 16 / 32 / 64 / 128 blocks selecting 2 / 4 / 8 / 16 / 32 → the coarsest (2 of 8) is ~1e-2 worse in loss than every finer setting.

### Switching between MoBA and full attention (§3.2)

- **Hybrid pretraining: 1.5B, 30B tokens at 32K, block 2048, top-3.** Three recipes: MoBA for 90% of tokens then full attention for the last 10%; full throughout; MoBA throughout. Position-wise loss: MoBA-only is worse at trailing positions; **MoBA→full 90/10 is "nearly identical" to full attention**, with "no significant loss spikes" at the switch. This is the paper's evidence that sparse-then-dense training is a legitimate way to buy cheap long-context pretraining.
- **Layer-wise hybrid for SFT.** MoBA "sometimes results in suboptimal performance during SFT"; the hypothesis is that SFT masks prompt tokens out of the loss, and the resulting sparse gradient propagates poorly through sparse attention. Fix: switch the **last few layers** back to full attention; Fig. 5b–c show SFT loss and trailing loss falling as full-attention layers are added.

### Llama-8B-1M (§3.3, Table 2)

Recipe (Fig. 6): start from **Llama 3.1 8B Base**; continual pretraining with **full attention** at 128K → 256K → 512K → 1M (position interpolation applied at the start of the 256K stage); then **MoBA activated for 100B tokens** with block 4096, top-12 (sparsity 1 − 4096·12/1M = 95.31% at 1M; 62.5% at 128K), **last 3 layers full attention, the other 29 MoBA**; SFT with context grown 32K → 1M. The baseline Llama-8B-1M-Full follows the same schedule with full attention throughout. **Evaluation uses MoBA for prefill only and full attention for generation** ("for better performance").

| Benchmark | Llama-8B-1M-MoBA | Llama-8B-1M-Full |
|---|---|---|
| AGIEval [0-shot] | 0.5144 | 0.5146 |
| BBH [3-shot] | 0.6573 | 0.6589 |
| CEval [5-shot] | 0.6273 | 0.6165 |
| GSM8K [5-shot] | 0.7278 | 0.7142 |
| HellaSwag [0-shot] | 0.8262 | 0.8279 |
| LooGLE [0-shot] | 0.4209 | 0.4016 |
| Competition Math [0-shot] | 0.4254 | 0.4324 |
| MBPP [3-shot] | 0.5380 | 0.5320 |
| MBPP Sanitized [0-shot] | 0.6926 | 0.6615 |
| MMLU [0-shot] | 0.4903 | 0.4904 |
| MMLU-Pro [5-shot, CoT] | 0.4295 | 0.4328 |
| HumanEval [0-shot, pass@1] | 0.6951 | 0.7012 |
| SimpleQA [0-shot] | 0.0465 | 0.0492 |
| TriviaQA [0-shot] | 0.5673 | 0.5667 |
| LongBench @32K [0-shot] | 0.4828 | 0.4821 |
| **RULER @128K** [0-shot] | **0.7818** | **0.7849** |

Differences are within a point almost everywhere: MoBA wins 7 of 16 (largest: MBPP Sanitized +0.031, LooGLE +0.019, GSM8K +0.014, CEval +0.011) and loses 9 (largest: Competition Math −0.007, HumanEval −0.006, MMLU-Pro −0.003, RULER −0.003). RULER at 128K, where MoBA runs at 62.5% sparsity, is the paper's own headline: 0.7818 vs 0.7849. Needle-in-a-haystack to 1M is "satisfactory" (Fig. 7; no number given).

### Efficiency (§3.4, Fig. 2) — attention layer only

The comparison is the **forward time of the attention layer** in Llama-8B-1M-MoBA vs -Full ("all other layers … have identical FLOPs"), against FlashAttention:

| Setting | Context | FlashAttention | MoBA | Speedup |
|---|---|---|---|---|
| Fig. 2a, the 1M model (block 4096, top-12) | ≤ 128K | indistinguishable on the plot | — | ~1× |
| | 256K | ≈ 55 ms | ≈ 25 ms | ~2× (read off) |
| | 512K | ≈ 240 ms | ≈ 60 ms | ~4× (read off) |
| | **1M** | ≈ 930 ms | ≈ 145 ms | **6.5×** (stated) |
| Fig. 2b, fixed 95.31% sparsity (64 blocks, top-3, block size ∝ N) | 32K–512K | comparable (inset) | comparable | ~1× |
| | 4M | ≈ 15 s | ≈ 1 s | ~15× (read off) |
| | **10M** | ≈ 93 s | ≈ 6 s | **16×** (stated) |

The ≈ values are read from linear-axis plots and are good to ~10%; the two bold values are the paper's. *Derived:* in Fig. 2a each query attends at most 12 × 4096 = 49K tokens while causal full attention averages N/2 = 512K at 1M, so the FLOP ratio is ~10×; the observed 6.5× is that minus dispatch/gather overhead. The authors call the complexity "sub-quadratic"; with a fixed block count n and top-k it is O(N² · k/n + N · n) — still quadratic, with constant k/n = 3/64 in Fig. 2b, which is why 16× at 10M and ~1× at 512K are both consistent with the design.

## Claims that did not check out, or need qualification

- **6.5× and 16× are attention-layer forward times**, not end-to-end prefill; at ≤ 128K the attention layer is at parity, so the end-to-end gain at ordinary contexts is nil. No decode timing and no TTFT is reported.
- **"Deployed to support Kimi's long-context requests"** is confirmed as a sentence in the abstract, but nothing in the paper describes the production configuration, and the paper's own evaluation switches to full attention for generation — so what Kimi's *decode* path does is not established here.
- **"Sparsity up to 75%"** vs the computed **81.25%** in the same paragraph (the 32K figure of 95.31% is right).
- **"Last 1K tokens"** (Fig. 3 caption) vs **"last 2K"** (Fig. 3c table) for trailing loss.
- **"Comparable"** at 8K hides that trailing-token loss at 32K is worse than full attention at every scale (prefactor 1.546 vs 1.464); the narrowing-gap reading rests on the fitted exponents.
- **No GQA/MQA discussion** — selection is per head (Algorithm 1), so on a GQA model the KV bytes loaded at decode are the union across the group, exactly the problem NSA's §2.1 raises; the paper is silent on it.
- Everything else verified: 25 authors, 18 Feb 2025, 15 pages, all Table 1 and Table 2 values, sparsity arithmetic (81.25 / 95.31 / 62.5%), block 4096 / top-12, 100B tokens, 29 + 3 layers, the 90/10 hybrid, block 2048 / top-3 for the hybrid runs, the 8→128 block ablation, the current-block and −∞ causality rules, the FlashAttention-varlen + online-softmax implementation.

## Limitations

- **Parameter-free gating caps what can be learned.** The router cannot become better than "query · mean key"; NSA's learned compression and DSA's learned indexer are both answers to that. MoBA's own related-work section concedes Quest is the same gate with a different pooling.
- **Per-head selection** ignores KV-sharing architectures (above).
- **Prefill-only in the reported evaluation**; the hybrid experiments show MoBA-only training hurts trailing tokens and SFT, and the fixes (last three layers full, full attention at generation) leave part of the model dense.
- **Reasoning after long-CoT SFT/RL is not evaluated** — the one setting NSA tests (AIME) and the one Moonshot's own k1.5 motivates on page 1.
- **Speedup baseline version is unstated** ("Flash Attention", Dao et al. 2022) and the hardware is not named.
- The largest model trained with MoBA from scratch is 2.1B; the 8B result is a retrofit.

## Related

- [[trained-sparse-attention]] — the concept page; MoBA is the *adapt-an-existing-model* end of the spectrum (zero new parameters, full ↔ sparse switching).
- [[native-sparse-attention-yuan-2025]] — the from-scratch sibling published two days earlier: learned compression + learned gate + group-shared selection + a Triton kernel, versus MoBA's parameter-free per-head gate on stock FlashAttention.
- [[deepseek-v3.2-sparse-attention-deepseek-2025]] — the production middle path: a small *learned* indexer distilled from the dense model's attention, adapted by continued training. Same top-k-per-query shape as MoBA, learned like NSA.
- [[moonshot-ai]] · [[kimi-k3-moonshot-2026]] · [[kimi-linear-kimi-team-2025]] · [[flash-kda-moonshot-2026]] · [[attention-residuals-kimi-team-2026]] — Moonshot's later long-context line went the way the MoBA introduction argues against: a *linear-attention* hybrid (Kimi Delta Attention, 3 KDA : 1 MLA) rather than block-sparse softmax. This paper is the sparse-softmax branch of that history; the Kimi Linear page already lists MoBA's LSE-based chunk selection among competing methods.
- [[mixture-of-experts]] — the routing analogy is literal: top-k gate, always-on "shared" current block, fine-grained segmentation, MoE-style dispatch/combine in the kernel.
- [[flash-attention]] · [[attention-sink]] — the varlen kernel and online-softmax combine; sinks and sliding windows are fixed-gate special cases of MoBA.
- [[sparse-attention]] · [[sparse-prefill]] · [[minference-dynamic-sparse-attention-jiang-2024]] — the training-free lineage (Quest, MInference, RetrievalAttention) MoBA is positioned against; the attention-share rule there applies to the 6.5×.
- [[linear-attention]] — the alternative MoBA explicitly declines.
- [[grouped-query-attention]] · [[kv-cache]] — the missing discussion.
- [[llama]] · [[longbench]] · [[long-context-llm]] · [[methods-to-speed-up-prompt-processing]].

## References

- [arXiv:2502.13189](https://arxiv.org/abs/2502.13189) · code [github.com/MoonshotAI/MoBA](https://github.com/MoonshotAI/MoBA)
- NSA: Yuan et al., [arXiv:2502.11089](https://arxiv.org/abs/2502.11089) — concurrent, not cited by MoBA.
- Quest: Tang et al., ICML 2024 — "MoBA with a smaller block size and min/max pooling".
- LongHeads: Lu et al., 2024 — "MoBA with a top-1 gating network".
- MInference: Jiang et al., 2024; RetrievalAttention: Liu et al., 2024; SeerAttention: Gao et al., 2024; MoA: Fu et al., 2024 — training-free prefill sparsity.
- H2O, StreamingLLM, TOVA, FastGen — decode-side KV pruning.
- DeepSeekMoE: Dai et al., 2024 — fine-grained and shared experts, both borrowed.
- Position interpolation: Chen et al., 2023; online softmax: Milakov & Gimelshein, 2018; ring attention: Liu & Abbeel, 2023.
- Llama 3.1: Grattafiori et al., [arXiv:2407.21783](https://arxiv.org/abs/2407.21783).
