---
title: "H2O: Heavy-Hitter Oracle for Efficient Generative Inference of Large Language Models"
type: source
created: 2026-10-23
updated: 2026-10-23
sources:
  - h2o-heavy-hitter-oracle-zhang-2023.md
tags:
  - 2023
  - paper
status: complete
importance: medium
authors:
  - Zhenyu Zhang
  - Ying Sheng
  - Tianyi Zhou
  - Tianlong Chen
  - Lianmin Zheng
  - Ruisi Cai
  - Zhao Song
  - Yuandong Tian
  - Christopher Ré
  - Clark Barrett
  - Zhangyang Wang
  - Beidi Chen
introduces:
  - [[kv-cache-eviction]]
year: 2023
venue: NeurIPS 2023
arxiv_id: "2306.14048"
---

# H2O: Heavy-Hitter Oracle

[arXiv:2306.14048](https://arxiv.org/abs/2306.14048), **NeurIPS 2023**.

| | |
|---|---|
| **Authors** | Zhang, Sheng, Zhou, Chen, Zheng, Cai, Song, Tian, Ré, Barrett, Wang, Chen |
| **Affiliations** | UT Austin, Stanford, UCSD, Berkeley, Adobe, Meta, CMU |
| **Code** | github.com/FMInference/H2O |

## Idea

- **Attention is sparse.** It is "over 95% sparse at inference time", which suggests up to a 20× smaller cache.
- **A few tokens dominate.** Accumulated attention scores follow a **power law**, so a small set of **heavy hitters** carries most of the value. Masking them breaks accuracy.
- **The policy.** H2O keeps a fixed cache budget split **evenly between heavy hitters and recent tokens** ([[kv-cache-eviction]]).
  - It evicts one token per step: the one with the least accumulated attention, scored from past queries only.
  - Framed as dynamic submodular maximization, it comes with a near-optimality bound.
- **Motivating number:** a 30B model at batch 128 and sequence 1,024 needs **180 GB of KV**.

## Results

- **Budget.** At a **20% budget** (5× smaller cache), quality matches the full cache.
- **Throughput.** Up to **29× / 29× / 3×** over DeepSpeed ZeRO-Inference, HF Accelerate and FlexGen (OPT-6.7B and 30B).
  - On an A100 (6.7B, 2K + 2K tokens): 918.9 vs 494.1 tok/s.
  - At batch 64 FlexGen runs out of memory while H2O reaches 1,161 tok/s.
- **Heavy hitters do the work.** Recent tokens alone collapse COPA to 48. Adding heavy hitters restores it to 84, and they also rescue strided and fixed sparse attention.
- **Combinations.** It works with KV quantization. Combined with sink-style position rolling, it streams 4M tokens with lower perplexity than [[streamingllm-attention-sinks-xiao-2023|StreamingLLM]].

## Limitations

- **Eviction is irreversible:** "evicted KV embeddings become inaccessible in the subsequent decoding steps". Dropping critical KV "can cause a severe functional collapse".
- **Bias toward old tokens.** Accumulated scores favour the least recent tokens, and averaging them instead hurt accuracy.
- **Generation phase only.**
- **Retrieval not tested.** Needle-retrieval and multi-turn behaviour are not evaluated. A later survey reports −5.11% for H2O on MMLU, CS-QA and HumanEval ([[key-value-compress-kv-cache-compression-survey-2025]]).

## Related

[[kv-cache-eviction]] · [[kv-cache]] · [[attention-sink]] · [[snapkv-li-2024]] · [[token-pruning]] · [[sparse-attention]]
