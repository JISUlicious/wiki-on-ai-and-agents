---
title: "RWKV-7 \"Goose\" with Expressive Dynamic State Evolution"
type: source
created: 2026-10-27
updated: 2026-10-27
sources:
  - rwkv-7-goose-peng-2025.md
tags:
  - 2025
  - paper
status: complete
importance: medium
authors:
  - Bo Peng
  - Ruichong Zhang
  - Daniel Goldstein
  - Eric Alcaide
  - William Merrill
introduces:
  - [[rwkv]]
year: 2025
venue: arXiv preprint
arxiv_id: "2503.14456"
---

# RWKV-7 "Goose"

[arXiv:2503.14456](https://arxiv.org/abs/2503.14456) (v2, 2025-03-30), from the RWKV Project (Linux Foundation AI & Data), EleutherAI, Tsinghua and others. There are 18 authors; Peng, Zhang and Goldstein share first authorship. Weights, data list and code are released under **Apache-2.0**.

## What it is

[[rwkv|RWKV-7]] is a recurrent LM with constant memory and constant time per token. It **generalizes the delta rule** ([[gated-deltanet|Gated DeltaNet]]) in three ways:
- a **vector-valued decay** `w_t`, one per channel rather than per head;
- a **vector-valued in-context learning rate** `a_t`;
- **separate keys for removing from and adding to** the state.

```
wkv_t = wkv_{t-1} · (diag(w_t) − κ̂_tᵀ (a_t ⊙ κ̂_t)) + v_tᵀ · k̃_t
w_t ∈ (≈0.545, 1)   (decay)     a_t = σ(·) ∈ (0,1)   (in-context learning rate)
```

- **Transition shape:** a diagonal-plus-rank-one transition, about a scaled Householder reflection (eigenvalues in [−1, 1]), so training stays parallelizable.
- **State size:** 64×64 per head. For the 2.9B model that is about 5.2M WKV state values, against a [[kv-cache]] that grows with context.

## Expressivity claim

- **Theorem 2:** RWKV-7 can solve an **NC¹-complete** problem (S5 swap tracking).
- **Theorem 3:** a 4-layer RWKV-7 can recognize **any regular language**.

The paper presents this as beyond TC⁰, which bounds Transformers and diagonal SSMs, assuming TC⁰ ≠ NC¹.

**Caveat:** the pretrained models allow only part of the negative-eigenvalue range, because of training instability.

## Results

| Model | Training tokens | English avg | MMLU | Multilingual avg |
|---|---|---|---|---|
| **RWKV7-World3-2.9B** | **5.6T** | **71.5** | 55.0 | **61.1** |
| Qwen2.5-3B | 18T | 71.4 | 65.7 | 55.6 |
| Llama-3.2-3B | 15T | 67.8 | — | 58.1 |

- **Headline claim:** it matches Qwen2.5's English average "with less than one third as many training tokens". The World models were **continued from RWKV-5/6 checkpoints**, not trained from scratch, on at most 96 H800 GPUs.
- **Recall:**
  - MQAR recall is 72.93% at 256 key-value pairs.
  - Pass-key retrieval is perfect to 35k tokens at 2.9B (pretrained at 4k context), and degrades near 50k after fine-tuning at 128k.
- **Speed:** at 16k tokens, the kernel's forward pass takes 7.9 ms against 33.9 ms for FlashAttention-3 (H100).

## Limitations

- The kernel is precision-sensitive (fp32 was used for the World models).
- Base models only, no instruction tuning.
- Prompt-format sensitivity.
- Loss rises beyond 10k tokens for the 4k-trained World models.

## Related

[[rwkv]] · [[state-space-model]] · [[linear-attention]] · [[gated-delta-networks-yang-2024]] · [[mamba-2-transformers-are-ssms-dao-2024]]
