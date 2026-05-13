---
title: GPT Family
type: concept
created: 2026-05-13
updated: 2026-05-13
sources:
  - gpt-1-radford-2018.md
status: complete
importance: high
---

# GPT Family

The GPT family is the line of [[decoder-only-transformer|decoder-only Transformer]] language models produced by [[openai|OpenAI]] starting in 2018. The defining recipe across the family: generative pre-training with [[autoregressive-language-modeling|next-token prediction]] on increasingly large corpora, with successive generations principally distinguished by **scale** (parameters, data, compute) and, in later generations, by post-training procedures like instruction tuning and [[rlhf|RLHF]].

## Lineage

| Model | Year | Params | Key contribution |
|---|---|---|---|
| [[gpt-1]] | 2018 | 117M | The recipe: generative pre-training + discriminative fine-tuning |
| [[gpt-2]] | 2019 | 1.5B | Scale + zero-shot transfer; "language models are unsupervised multi-task learners" |
| [[gpt-3]] | 2020 | 175B | [[in-context-learning|In-context learning]] / few-shot prompting; [[scaling-laws|scaling laws]] confirmed |
| [[gpt-4]] | 2023 | undisclosed | Multimodality (images), strong reasoning, post-trained for alignment |

ChatGPT (Nov 2022) is a deployed product based on a fine-tuned GPT-3.5 / GPT-4 — not itself a separate model family member, but the event that took LLMs mainstream.

## Common architectural features

- Decoder-only [[transformer-architecture|Transformer]] with masked self-attention.
- [[byte-pair-encoding|BPE]] tokenizer (byte-level in GPT-2+).
- Learned positional embeddings.
- Trained on web-scale text corpora (WebText for GPT-2; Common Crawl + Books + Wikipedia + WebText for GPT-3).
- Pre-norm in GPT-2+ (post-norm in GPT-1 followed the [[transformer]] original).

## Recurring research themes

- **Scaling**: each generation 10–100× the previous in parameters and/or training compute.
- **Capability emergence**: behaviors like [[in-context-learning|few-shot learning]], [[chain-of-thought-prompting|chain-of-thought]] reasoning, and tool use emerge at scale rather than being explicitly trained.
- **Alignment**: from GPT-3.5 onward, [[rlhf|RLHF]] and constitutional methods became central to making the base model usable as an assistant.

## References

- [[gpt-1-radford-2018]]
