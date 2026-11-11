---
title: State-Space Model (SSM)
type: concept
created: 2026-05-16
updated: 2026-11-10
sources:
  - titans-behrouz-2025.md
  - s4-structured-state-spaces-gu-2021.md
  - mamba-selective-state-spaces-gu-2023.md
  - mamba-2-transformers-are-ssms-dao-2024.md
  - gated-delta-networks-yang-2024.md
  - rwkv-7-goose-peng-2025.md
  - griffin-gated-linear-recurrences-de-2024.md
  - xlstm-extended-lstm-beck-2024.md
  - jamba-hybrid-transformer-mamba-lieber-2024.md
  - you-dont-need-long-context-belcak-2026.md
  - competing-learning-in-hybrid-architectures-shen-2026.md
status: complete
importance: high
---

# State-Space Model (SSM)

A **state-space model** is a sequence layer built on a linear dynamical system. It compresses everything seen so far into a **fixed-size state** `h`, updated once per token:

```
continuous:  h'(t) = A h(t) + B x(t),   y(t) = C h(t)
discrete:    h_t = Ā h_{t−1} + B̄ x_t,   y_t = C h_t        (Ā, B̄ from A, B and a step size Δ)
```

**Cost:** generation is O(1) memory and O(1) time per token, with no [[kv-cache]]. Training runs in parallel over the sequence, as a convolution, a scan or chunked matmuls.

**The central trade-off,** in Mamba's words: attention "explicitly does not compress context at all"; an SSM is only as good as its compression ([[mamba-selective-state-spaces-gu-2023]]).

## The lineage (primary sources in this wiki)

| Step | Year | What changed | Why it mattered |
|---|---|---|---|
| [[s4-structured-state-spaces-gu-2021\|S4]] | 2021 | HiPPO initialization; normal-plus-low-rank A; **time-invariant**, so it trains as an FFT convolution | first to solve LRA Path-X (16K steps); 60× faster generation than Transformers; still trailed on language |
| [[mamba-selective-state-spaces-gu-2023\|Mamba]] | 2023 | **selection**: Δ, B, C depend on the input; hardware-aware fused scan | first attention-free model to match Transformer++ scaling; 5× inference throughput; induction heads extrapolate to ~1M tokens |
| [[mamba-2-transformers-are-ssms-dao-2024\|Mamba-2 / SSD]] | 2024 | scalar-times-identity A gives **state space duality**: an SSM equals masked [[linear-attention]] | 2–8× faster training via matmuls; states 64–256 wide; ~10% attention layers is best |
| [[gated-delta-networks-yang-2024\|Gated DeltaNet]] | 2024 | Mamba-2's gate + the **delta rule** (targeted overwrite) | beats Mamba-2 on LM and retrieval; adopted in production hybrids ([[gated-deltanet]]) |

**Parallel recurrent lines that reached the same place:**
- [[rwkv-7-goose-peng-2025|RWKV-7]]: a generalized delta rule with vector gates; claims regular-language expressivity.
- [[griffin-gated-linear-recurrences-de-2024|Griffin]]: the RG-LRU plus local attention; matches Llama-2 on about 1/7 of the tokens.
- [[xlstm-extended-lstm-beck-2024|xLSTM]]: a matrix-memory LSTM; extrapolates where Llama collapses.

**After SSD, "SSM", "linear attention" and "gated linear RNN" are largely one design family.** The members differ in how the state forgets (fixed or input-dependent decay; scalar, vector or channel-wise) and how it writes (additive vs delta-rule overwrite).

## The standing limitation: a fixed state can't recall everything

Every primary source reports the same weakness:
- **Mamba-2:** associative recall and phonebook lookup are hard, and recall improves with state size N.
- **Gated DeltaNet:** pure recurrent models trail Transformers on real-world retrieval (30.6 vs 37.0).
- **Griffin:** perfect phonebook lookup only within its attention window. Hawk fails as length grows.
- **Jamba:** pure Mamba fails at in-context format-following.

**The field's answer is hybrids:**

| Hybrid | Recurrent : attention | Source |
|---|---|---|
| [[jamba]] | Mamba : attention = 7 : 1 (+ MoE) | [[jamba-hybrid-transformer-mamba-lieber-2024]] |
| Mamba-2-Attention | 58 SSD : 6 attention (~10%) | [[mamba-2-transformers-are-ssms-dao-2024]] |
| [[griffin]] | RG-LRU : local MQA = 2 : 1 | [[griffin-gated-linear-recurrences-de-2024]] |
| [[qwen3.8-flash-next\|Qwen3.8-Flash-Next]] | Gated DeltaNet : sparse attention = 3 : 1 | [[qwen3.8-flash-next-model-card-2026]] |
| [[kimi-linear-kimi-team-2025\|Kimi Linear]] | KDA : MLA = 3 : 1 | [[kimi-linear-kimi-team-2025]] |
| Nemotron 3 Super | hybrid Mamba-Transformer MoE | [[nemotron-3-super-nvidia-2026]] |

Other responses keep the pure recurrence but add exact memory: [[a-hippocampus-for-linear-attention-cui-2026|HOLA]]'s bounded KV buffer, or [[titans]]' test-time-trained memory.

## Memory at inference: state vs KV cache

The recurrent state is **fixed per layer**:
- N per channel for Mamba (N = 16; 64–256 for Mamba-2);
- a d_k × d_v matrix per head for Gated DeltaNet and RWKV-7 (64 × 64 per head; about 5.2M values for RWKV-7 2.9B).

That is why hybrids cut the KV cache in proportion to their attention ratio: Jamba needs 4 GB at 256K where Mixtral needs 32 GB ([[kv-cache-size-vs-active-parameters]]). The catch on hardware is that recurrent decode is memory-bound, because each step reads and writes the whole state ([[persistent-state-dataflow-accelerator-linear-attention-decode-gupta-2026]]).

## Related

[[linear-attention]] · [[gated-deltanet]] · [[mamba]] · [[rwkv]] · [[griffin]] · [[xlstm]] · [[recurrent-neural-network]] · [[lstm]] · [[recurrence]] · [[titans]] · [[test-time-memory]] · [[transformer-architecture]] · [[mixture-of-experts]]

## References

- [[s4-structured-state-spaces-gu-2021]] · [[mamba-selective-state-spaces-gu-2023]] · [[mamba-2-transformers-are-ssms-dao-2024]] · [[gated-delta-networks-yang-2024]] · [[rwkv-7-goose-peng-2025]] · [[griffin-gated-linear-recurrences-de-2024]] · [[xlstm-extended-lstm-beck-2024]] · [[titans-behrouz-2025]]

## Related 2026 sources

Surfaced via newsletter ingests; see [[index]].

- [[a-hippocampus-for-linear-attention-cui-2026]] — pairs a fixed-size recurrent state with a bounded *exact* KV cache for what the state overwrites; −16.1% ppl
- [[linear-attention]] — the adjacent family; KDA's gated delta rule is the recurrent-state design taken in an attention-shaped direction

## Lineage sources

- [[jamba-hybrid-transformer-mamba-lieber-2024]] — [[jamba]]: 1:7 attention:Mamba with MoE. 4 GB of KV at 256K against Mixtral's 32 GB. Pure Mamba fails at in-context format-following; the hybrid fixes it.

## Newer evidence (2026-11-10 sweep)

- Proposes "semantic state space models" that keep a readable text state between turns instead of a hidden state (not trained). ([[you-dont-need-long-context-belcak-2026]])
- In linear-recurrence + attention hybrids, recurrent layers learn retrieval first and starve attention; freezing them early (BLD) fixes it (RULER 32.75 → 36.04 at 1B). ([[competing-learning-in-hybrid-architectures-shen-2026]])
