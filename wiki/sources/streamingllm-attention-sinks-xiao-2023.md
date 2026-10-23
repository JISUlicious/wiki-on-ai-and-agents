---
title: "Efficient Streaming Language Models with Attention Sinks (StreamingLLM)"
type: source
created: 2026-10-23
updated: 2026-10-23
sources:
  - streamingllm-attention-sinks-xiao-2023.md
tags:
  - 2023
  - paper
status: complete
importance: high
authors:
  - Guangxuan Xiao
  - Yuandong Tian
  - Beidi Chen
  - Song Han
  - Mike Lewis
introduces:
  - [[attention-sink]]
year: 2023
venue: ICLR 2024
arxiv_id: "2309.17453"
---

# StreamingLLM: Efficient Streaming Language Models with Attention Sinks

[arXiv:2309.17453](https://arxiv.org/abs/2309.17453), **ICLR 2024**.

| | |
|---|---|
| **Authors** | Xiao (MIT; part of the work done at Meta AI), Tian (Meta AI), Chen (CMU), Han (MIT and NVIDIA), Lewis (Meta AI) |
| **Code** | github.com/mit-han-lab/streaming-llm |

This is the **primary source for [[attention-sink|attention sinks]]**.

## Finding

Window attention collapses as soon as the first tokens' KV is evicted. LLMs put "a surprisingly large amount of attention score" on the initial tokens "irrespective of their relevance".
- **Why:** softmax must sum to one, so excess attention needs somewhere to go, and the first tokens are visible to every position.
- **Position, not meaning:** replacing the first 4 tokens with "\n" still restores perplexity (5.60 vs 5.40).
- **Size of the effect:** above the bottom two layers, the first token often takes more than half of all attention.

## Method

- **Cache layout:** keep **4 sink tokens + a rolling window** of recent KV ([[kv-cache-eviction]]). Llama-2-7B with a 4,096-token cache:

  | Sinks + window | Perplexity |
  |---|---|
  | 0 + 4096 | 3359.95 |
  | 1 + 4095 | 11.88 |
  | **4 + 4092** | **9.59** |
  | 8 + 4088 | 9.54 |

- **Positions inside the cache:** positions are assigned within the cache, not the original text. Keys are cached pre-RoPE and re-rotated each step. Works with RoPE and ALiBi.
- **No fine-tuning needed.**
- **Pre-train with a sink:** 160M models trained with **one learnable sink token** need only that one token (streaming PPL 18.01, against 18.49 vanilla with one sink). The authors "recommend training future LLMs with a sink token".

## Results

- **Length:** stable perplexity up to **4M tokens** on the Llama-2, Falcon, Pythia and MPT families.
- **Speed:** **up to 22.2× per-token speedup** over sliding window with recomputation, at similar memory.
- **Adoption:** TensorRT-LLM, HF Transformers, MLC LLM, Intel Extension for Transformers.

## Limitations (stated)

- **No longer memory:** it "does not extend the models' context window or enhance their long-term memory".
- **Not for whole-document tasks:** it is unsuitable for long-document QA and summarization.
- **Evicted tokens are gone for good.**
  - StreamEval accuracy "drops to zero" once the question-answer distance exceeds the cache.
  - On LongBench it loses to plain truncation (NarrativeQA 11.6 vs 18.7).

## Related

[[attention-sink]] · [[kv-cache-eviction]] · [[kv-cache]] · [[h2o-heavy-hitter-oracle-zhang-2023]] · [[sliding-window-attention]] · [[kvquant-10-million-context-kv-quantization-hooper-2024]] (keeps the sink token in fp16) · [[cracks-in-the-foundation-long-context-extension-bertsch-2026]]
