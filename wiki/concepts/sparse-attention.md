---
title: Sparse Attention
type: concept
created: 2026-09-03
updated: 2026-09-03
sources:
  - minference-dynamic-sparse-attention-jiang-2024.md
  - token-sparse-attention-jo-2026.md
  - minimax-sparse-attention-lai-2026.md
builds-on:
  - [[attention-mechanism]]
status: complete
importance: high
---

# Sparse Attention

Keep every token in the sequence, but compute only a subset of the N×N attention entries. The kept pattern can be **fixed** (sliding window, A-shape sinks + local), **assigned per head offline** ([[minference-dynamic-sparse-attention-jiang-2024|MInference]]), **chosen per input online** (FlexPrefill, [[token-sparse-attention-jo-2026|Token Sparse Attention]]), or **learned** ([[minimax-sparse-attention-lai-2026|MiniMax]], [[qwen3.8-flash-next|Qwen Sparse Attention]]).

It is one family of [[sparse-prefill]], and the one with a hard ceiling: because every token still flows through the FFN and projections, **it can only ever save attention's share of prefill** — and that share is ~¼ at 10K, ~70–80% at 128K, >90% past 300K for an 8B model.

## The patterns

MInference's search over a single 30K sample assigns each head one of three, kernel-aware and FLOPs-matched:

- **A-shape** — 1K global (sink) tokens + 4K local window; static.
- **Vertical-Slash** — the last 64 queries attend to all keys; keep the top-k *columns* (verticals) and *diagonals* (slashes). **>90% of heads** end up here.
- **Block-Sparse** — mean-pool into 64×64 blocks, keep the top-100.

Token Sparse Attention instead does a per-head **gather** of the tokens whose cumulative attention mass exceeds a threshold τ, runs any unmodified kernel on the L′×L′ subproblem, and **scatters** the output back — so the selection differs per head and is reversible across layers, and unselected tokens continue through the residual untouched.

## Measured, and the split that matters

Always separate the **attention-kernel** speedup from the **end-to-end** one; abstracts quote the former.

| | Attention kernel | End-to-end prefill | Context |
|---|---|---|---|
| MInference, LLaMA-3-8B, A100 | 13× (V-S) / 30× (block) vs FlashAttention | **10×** (30 min → 3 min) | **1M** |
| MInference | — | 1.8× / 4.1× / 6.8× | 100K / 300K / 500K |
| MInference | — | *"approaches FlashAttention"*; index overhead 30% | 10K |
| Token Sparse Attention, A100 | ×1.36 | **×1.27** TTFT (31.0 → 24.4 s) | 128K |
| Token Sparse + FlexPrefill | ×2.76 | ×1.85 TTFT | 128K |
| Token Sparse Attention | ×0.97 (A100) / ×0.86 (A6000) | no gain | **8K** |
| MInference on LongBench prompts | — | **×0.47** — slower than dense | ~4–8K |

Quality holds where retrieval is local: InfiniteBench 38.2 → 38.8, RULER 84.4 → 87.0 with effective context 16K → 32K on LLaMA — but **Yi-9B loses 3.4 RULER points** (−6.0 at 128K), and static indices collapse KV-retrieval 14.4 → 0.2. Attention-map sparsity itself is only **54% at 128K and 17% at 4K** at τ = 0.005 — there is less to skip than the O(N²) framing suggests.

> [!warning] The abstract's number is the kernel's, not yours
> Token Sparse Attention's headline "up to ×3.23 with <1% degradation" appears in no table; the closest cells are ×2.76 (Table 1) and ×3.49 in a stack that sits 3.5 points below dense. MInference's "while maintaining accuracy" is an average that hides Yi-9B. And the composition claims are Pareto-frontier claims — Retr.KV loses 2–4 points when stacked on MInference/FlexPrefill.

## Where it wins

Past ~300K tokens, nothing else competes: [[token-pruning]] and [[speculative-prefill]] have no answer to a 1M prompt that must stay intact, and the attention term is by then >90% of the work. Below ~64K, the overhead of finding the pattern exceeds the attention it saves, and shrinking the sequence wins. In between, stack: a draft-picked token subset with vertical-slash attention on the survivors ([[draft-based-approximate-inference-galim-2025|SpecKV-PC]]) is the composed design.

Trained sparse attention — [[minimax-sparse-attention-lai-2026|MiniMax]]'s 28.4× less attention compute at 1M, 14.2× prefill on H800; Qwen's QSA with a fixed 2,048-token indexer budget in [[qwen3.8-flash-next]] — moves the pattern into pretraining and removes the online search cost entirely.

## Related

- [[sparse-prefill]] — the umbrella and the attention-share rule.
- [[attention-mechanism]] · [[scaled-dot-product-attention]] · [[flash-attention]] · [[attention-sink]] — A-shape's global tokens *are* sinks.
- [[linear-attention]] — the other route to sub-quadratic: change the operator rather than sparsify it.
- [[grouped-query-attention]] · [[kv-cache]] · [[long-context-llm]]
- [[token-pruning]] · [[speculative-prefill]] · [[methods-to-speed-up-prompt-processing]]

## References

- [[minference-dynamic-sparse-attention-jiang-2024]] · [[token-sparse-attention-jo-2026]] · [[minimax-sparse-attention-lai-2026]] · [[draft-based-approximate-inference-galim-2025]]

## Related 2026 sources

- [[language-models-can-control-their-own-attention-ho-2026]] — a fourth answer to *who decides* the sparsity pattern — **the model declares it in its own chain-of-thought** and the engine parses it like a tool call, driving selection cost from O(N) to zero. **Decode-time, not prefill**: attended tokens fall 52.0% on Gemma-4-31B for 1.27 pp of accuracy
- [[trained-sparse-attention]] — the lineage where the selector is trained in — NSA, MoBA, DeepSeek-V3.2's DSA — and why a retrofit cannot afford the sparsity a trained model can
- [[native-sparse-attention-yuan-2025]] — three-branch trained sparsity, ACL 2025 best paper; 9× kernel at 64K, no end-to-end number anywhere
- [[moba-mixture-of-block-attention-lu-2025]] — MoE-style top-k over pooled key blocks; 6.5× at 1M, ~1× at 512K, still O(N²·k/n); deployed for Kimi
- [[deepseek-v3.2-sparse-attention-deepseek-2025]] — the lightning indexer: FP8, few heads, top-2,048 per query; a dense model adapted with ~1T tokens of continued pre-training
- [[sts-speculative-token-sparsity-yu-2026]] — **decode-side**: draft-model attention builds a per-head mask for the target's verification pass; 1.36×/2.06×/2.88× at 10K/32K/100K generation; prefill kept dense by default
- [[specattn-speculating-sparse-attention-shah-2025]] — same idea at layer granularity with top-p; 78% of KV reads skipped at +15% perplexity — but **net slower** than plain [[speculative-decoding|speculative decoding]] (59.95 vs 68.26 tok/s)
- [[misa-mixture-of-indexer-sparse-attention-zhou-2026]] — routes each query to 8 of DSA's 64 indexer heads, parameter-free; retrofit onto DSA-trained models only
- [[gated-sparse-attention-shen-2026]] — DSA-style indexer plus G1/G2 gates, 1.7B from scratch; sparse-only is already *more* stable than dense in its own table
