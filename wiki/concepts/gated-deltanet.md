---
title: Gated DeltaNet (gated delta rule)
type: concept
created: 2026-10-27
updated: 2026-10-27
sources:
  - gated-delta-networks-yang-2024.md
status: draft
importance: high
builds-on:
  - [[linear-attention]]
  - [[mamba]]
---

# Gated DeltaNet (gated delta rule)

A recurrent token mixer with a matrix state per head. Its update combines **Mamba-2's decay gate** (fast erasure) with **DeltaNet's delta rule** (targeted overwrite of one key's value):

```
S_t = S_{t−1} · α_t (I − β_t k_t k_tᵀ) + β_t v_t k_tᵀ,     o_t = S_t q_t
```

Read as learning, each token is one SGD step on `½‖S k_t − v_t‖²`, with learning rate β_t and weight decay α_t ([[test-time-memory]]). It trains chunkwise in parallel via the WY/UT transform ([[gated-delta-networks-yang-2024]], ICLR 2025).

## Where it sits

```
linear attention (add only) ─┬─ + gate (Mamba-2 / SSD) ─┐
                             └─ + delta rule (DeltaNet) ─┴─► Gated DeltaNet ─► KDA (channel-wise gate, Kimi)
                                                                            ╰► RWKV-7 (vector gate + vector learning rate; concurrent)
```

## Evidence and adoption

- **Benchmarks** (1.3B / 100B tokens): it beats Mamba-2 and DeltaNet on language modelling and on all three needle-in-a-haystack (S-NIAH) variants.
- **Retrieval needs a hybrid.** Pure Gated DeltaNet still trails Transformers on real retrieval (30.6 vs 37.0). The sliding-window-attention hybrids win (39.0–40.1).
- **Production:**
  - [[qwen3.8-flash-next|Qwen3.8-Flash-Next]] uses it in 36 of 48 layers;
  - Kimi Linear's KDA extends it ([[kimi-linear-kimi-team-2025]]);
  - HOLA adds an exact KV buffer ([[a-hippocampus-for-linear-attention-cui-2026]]);
  - an FPGA decode accelerator targets it ([[persistent-state-dataflow-accelerator-linear-attention-decode-gupta-2026]]).

## Related

[[linear-attention]] · [[state-space-model]] · [[mamba]] · [[rwkv]] · [[titans]]
