---
title: "DeepSeek-V4.1-Flash: Pushing the Limits of KV Cache Compression"
type: source
created: 2026-09-30
updated: 2026-10-20
sources:
  - deepseek-v4.1-flash-kv-cache-compression-deepseek-2026.md
arxiv_id: "2609.19969"
authors:
  - DeepSeek-AI
year: 2026
venue: "Technical report (arXiv, cs.CL, v1 17 Sep 2026); weights on Hugging Face"
status: complete
importance: high
tags:
  - 2026
  - paper
---

# DeepSeek-V4.1-Flash: Pushing the Limits of KV Cache Compression

**[[deepseek|DeepSeek-AI]]** — [arXiv:2609.19969](https://arxiv.org/abs/2609.19969), v1 17 Sep 2026. Checkpoints at `huggingface.co/deepseek-ai/DeepSeek-V4.1-Flash`.

## Summary

A 552B-backbone multimodal [[mixture-of-experts|MoE]] model (plus 196B Engram lookup parameters) built to cut the [[kv-cache]] cost of input-heavy agent workloads. Three moves stack. **Causal Encoder-Decoder (CED)**, which the paper says is inspired by YOCO, projects the global KV of all 20 decoder layers from the last encoder layer's hidden state, so prefill runs only the encoder half: 8B active parameters in prefill, 16B in decode. **Compressed Sparse Attention 2 (CSA2)** shares main KV and indexer K across layers and reuses Top-K indices, using three static per-layer modes (Full, Reindex, Reuse). The **main KV cache is stored in FP4**. The result is 890 bytes/token of global KV in HBM, about 1/4 of DeepSeek-V4-Flash. **SWA Bounded Replay** takes sliding-window KV out of the persistent SSD cache altogether, which brings persistent KV to about 1/8 of V4-Flash.

## Key points

- **Layout (§4.2.1).** 40 layers with d = 5120, split into a 20-layer encoder and a 20-layer decoder. Layers 1–2 are pure SWA. The other 18 encoder layers use CSA2 at compression m = 2, in three groups of six (1 Full + 5 Reuse). The 20 decoder layers use m = 1, in five groups of four: group 1 is 1 Full + 3 Reuse, and the rest are 1 Reindex + 3 Reuse. SWA window n_win = 128. MoE has 1 shared + 384 routed experts, 6 active.
- **Indexer change vs V3.2.** Every CSA2 layer has **32 indexer query heads**, head dim 128, and attention top-k **512**. The released V3.2 config had `index_n_heads: 64`, `index_topk: 2048`, so V4.1-Flash halves the indexer heads and quarters top-k. Only 1 layer in 6 (encoder) or 1 in 4 (decoder) runs an indexer at all.
- **Hierarchical Sparse Indexer** (decoder only, added in post-training). The first Full-mode decoder layer scores the whole context and keeps up to 2,048 blocks × 8 positions, a pool of 16,384 candidates. Later Reindex layers score only that pool, so their per-query cost no longer grows with context length.
- **FP4 main KV.** E2M1 with one E4M3 scale per 16 channels (NVFP4 without its global scale), added by QAT in post-training and quantized after RoPE. SWA KV stays in FP8 because it is sensitive to quantization. The indexer was already FP4 (MXFP4) in V4.
- **Bounded replay.** Exact SWA reconstruction needs L × n_win tokens; bounded replay uses only the last n_win and accepts approximate states. This is applied to both the encoder (a cache miss) and the decoder (every prefill). The paper reports "negligible" or "barely compromises" quality degradation but gives **no number** for it.
- **Persistent cache.** SWA KV moves to a pool that uses 10% of host DRAM and has a TTL of minutes. Global KV keeps a guaranteed lifetime of at least 72 h on SSD.
- **Other architecture.** Single-Pass [[hyper-connections|mHC]] halves activation traffic by using the previous block's input-mixing coefficients. Engram has two modules at layers 1 and 14. DSpark replaces MTP as the drafter (3 blocks, window 128, 5 draft positions), trained after pretraining with the backbone frozen. Head-wise Muon is used, and a Sinkhorn-balanced update is used for the embeddings and Engram tables.
- **Training.** 45T multimodal tokens with a batch of 100.6M tokens. Sparse attention is trained from scratch at 64K with no dense warm-up, and context is extended to 1M at 34T tokens.
- **Base results (Table 1, vs V4-Pro-Base).** MMLU-Pro 74.1 vs 73.5, MATH 61.1 vs 64.5, **SimpleQA-verified 42.3 vs 55.2**, MultiLoKo 45.5 vs 50.9.
- **Post-trained results (§5.3.2).** Codeforces 3471 vs V4-Flash 3289. DeepSWE v1.1 74.2 vs 54.4. Terminal-Bench 2.1 90.6. GPQA Diamond 90.9 vs 89.9. Raising reasoning effort from 25 to 100 lifts Terminal-Bench 2.1 from 82.4 to 90.6 at about 2.5× the output tokens.

> [!note] Abstract vs body
> §1 claims the base model is "comparable to DeepSeek-V4-Pro-Base" in world knowledge and gives "5%–10% improvements on held-out evaluations". Table 1 shows large knowledge gaps (SimpleQA −12.9, MultiLoKo −5.4). The held-out figure comes from a BPB plot (Fig. 6) that has no numeric values.

## What it does not show

- There is no ablation that separates the quality cost of CED, CSA2 reuse, FP4 KV, or bounded replay; each is described only as "negligible" or "marginal".
- The 1/4 and 1/8 footprint ratios are analytical comparisons "at the same sequence length", not measurements from serving.
- The post-training section adds no new algorithms (SFT → RL → OPD). The paper attributes all of its gains to synthesized data and environments.

## Concepts & entities

- [[kv-cache]] · [[prefix-caching]] · [[quantization]] — FP4 main KV and a persistent-cache design that separates global KV from SWA KV.
- [[sparse-attention]] · [[trained-sparse-attention]] · [[deepseek-v3.2-sparse-attention-deepseek-2025]] — CSA2 extends the DSA indexer with cross-layer reuse.
- [[multi-head-latent-attention]] — the paper cites MLA as the head-axis entry; CSA2 compresses along the sequence and layer axes.
- [[hysparse2-hybrid-sparse-attention-two-level-kv-sharing-wei-2026]] — Xiaomi's YOCO-style design from the same month. It cites V4.1's bounded replay as the approximation it avoids.
- [[recurrent-looped-transformer-zhang-2026]] — also builds on encoder-derived memory, but keeps full decoder recurrence and does not skip the decoder during prefill.
- [[dspark-confidence-scheduled-speculative-decoding-deepseek-2026]] · [[speculative-decoding]] — the drafter.
- [[layer-skipping]] · [[pop-prefill-only-pruning-he-2026]] — CED is a trained, architectural form of skipping depth during prefill.
- [[reasoning-effort]] · [[on-policy-distillation]] · [[terminal-bench]] · [[gpqa]] · [[deepseek-v3]] · [[huggingface]].

## References

- [arXiv:2609.19969](https://arxiv.org/abs/2609.19969)
- YOCO: Sun et al. 2024 (CED's stated inspiration). HySparse: Gao et al. 2026. IndexCache (Bai et al. 2026), YOIO (Sun et al. 2026b) — prior work on reusing indices across layers.
- DeepSeek-V4: DeepSeek-AI 2026b (CSA/HCA hybrid). mHC: Xie et al. 2026. Engram: Cheng et al. 2026. DSpark: Cheng et al. 2026a.
