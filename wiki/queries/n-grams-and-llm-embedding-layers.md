---
title: "What is an n-gram, and why use n-gram embeddings in an LLM's encoding layer?"
type: query
created: 2026-11-02
updated: 2026-11-02
sources:
  - deepseek-v4.1-flash-kv-cache-compression-deepseek-2026.md
  - memory-attention-kang-2026.md
status: complete
importance: medium
---

# What is an n-gram, and why use n-gram embeddings in an LLM's encoding layer?

An **n-gram** is a run of n consecutive items, usually characters, words or tokens. For the tokens `New`, `York`, `City`, the bigrams are `New York` and `York City`, and the trigram is `New York City`.

In a modern LLM, n-gram embeddings add a large lookup table to the usual token embedding layer. The main advantage is that the model can store more knowledge without much extra compute. "Encoding layer" is read here as the input embedding layer and the early layers just after it.

> [!note] Source status
> The Engram figures come from reading [arXiv:2601.07372](https://arxiv.org/abs/2601.07372) (v2) directly. That paper has no source page in this wiki yet. They are the authors' own results; no independent replication was found.

## How it works

A normal embedding layer gives each token one vector, so `York` looks the same at input whatever came before it. The attention and feed-forward layers then have to work out that `New York` is one entity.

Engram (Cheng et al., [[deepseek|DeepSeek]] and Peking University) is the clearest design. For each token, it:

1. Takes the last 2 or 3 tokens, after merging equivalent forms such as `Apple` and ` apple`. This cuts the effective vocabulary of a 128k tokenizer by 23%.
2. Hashes them to rows in large embedding tables, using several hash functions to limit collisions.
3. Uses the current hidden state to decide how much of the result to let through.
4. Adds the result to the model's hidden state through a residual connection.

## Advantages

These compare a 27B-parameter model with a [[mixture-of-experts|MoE]] baseline of the same size and compute, except where noted.

| Advantage | Reported evidence |
|---|---|
| **Adds capacity at constant compute** | Validation loss kept improving as memory slots grew, following a power law. |
| **Beats spending all sparse parameters on experts** | Moving about 20–25% of those parameters to Engram gave the best result (loss 1.7248 → 1.7109 in the 10B setup). The allocation curve is U-shaped. |
| **Frees early layers** | Representations at Engram's layer 5 best matched the baseline's layer 12, so the model behaves as if it were deeper. |
| **Improves more than recall** | MMLU +3.4, but also BBH +5.0, ARC-Challenge +3.7 and HumanEval +3.0. |
| **Helps long-context retrieval** | Multi-query [[needle-in-a-haystack]] accuracy rose from 84.2 to 97.0, at matched pre-training loss. |
| **Keeps the table off the GPU** | The lookup is known from token IDs, so it can be prefetched. A 100B table in host memory cost at most 2.8% throughput on dense 4B and 8B backbones. See [[weight-streaming]]. |

N-grams also help at the word-vector level: FastText, covered in [[word-embeddings]], uses character n-grams to handle out-of-vocabulary words and morphology.

## Limits

- **Hash collisions and context:** the table is static and rows can collide. In ablations, removing the context gate was among the changes that hurt most.
- **Higher orders:** adding 4-grams was slightly worse with a 1.6B budget, though the authors don't rule them out at larger scale.
- **Memory:** training cost is high enough that DeepSeek-V4.1-Flash replaced Adam with a lighter optimizer for these tables.
- **Placement:** early injection offloads more work, but the gate has less context to work with. Layer 2 was the best single position in the paper's sweep.

> [!warning] Contradiction
> The Engram paper's abstract gives MMLU +3.4, while a later section gives +3.0. The table above uses the abstract's figure.

## Use in shipped models

- **[[deepseek-v4.1-flash-kv-cache-compression-deepseek-2026|DeepSeek-V4.1-Flash]]:** has 196B Engram parameters across layers 1 and 14, using orders 2, 3 and 4. It dropped the original design's short convolution because the gain didn't justify the complexity.
- **[[qwen3.8-flash-next|Qwen3.8-Flash-Next]]:** has a 51B table with 20M bigram and trigram entries, indexed at layer 2. Its entity page says Qwen chose it as cheaper to compute and easier to offload than more experts. The raw model card isn't in `sources/`, so this wasn't rechecked here.
- **[[memory-attention-kang-2026|Memory Attention]]:** a related idea using per-layer tables indexed by a single token, not n-grams. Its gains aren't parameter-matched, so they don't isolate the benefit of the structure.

## Gaps

The wiki has no source page for the Engram paper, and no concept page for Engram or n-gram embeddings.

## References

- [Conditional Memory via Scalable Lookup (arXiv:2601.07372)](https://arxiv.org/abs/2601.07372)
- [[deepseek-v4.1-flash-kv-cache-compression-deepseek-2026]]
- [[memory-attention-kang-2026]]
- [[qwen3.8-flash-next-model-card-2026]]
