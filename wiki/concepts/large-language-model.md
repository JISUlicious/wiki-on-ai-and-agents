---
title: Large Language Model (LLM)
type: concept
created: 2026-10-06
updated: 2026-10-06
sources:
  - ai-ml-dl-difference-nvidia-2016.md
  - ai-ml-dl-nn-ibm.md
  - pre-train-prompt-predict-liu-2021.md
status: draft
importance: high
---

# Large Language Model (LLM)

A [[deep-learning]] model, almost always a [[transformer|Transformer]] and usually [[decoder-only-transformer|decoder-only]], trained on large text corpora to predict the next token, then usually aligned for instruction following. It is the innermost circle of the field map: [[artificial-intelligence|AI]] ⊃ [[machine-learning|ML]] ⊃ [[deep-learning|DL]] ⊃ **LLM**.

> [!note] Origin of the term "large language model"
> **Not covered.** The wiki has no primary source for the first use of "LLM". The 2016 NVIDIA and IBM field-map sources predate LLMs or do not name them; IBM's page speaks only of "foundation models" for generative AI. By the early 2020s the term was standard. [[ai-chains-wu-2021|AI Chains]] (Oct 2021) already writes "large language models (LLMs)".

## Lineage anchors (dates from wiki source pages)

| When | Step |
|---|---|
| 2017-06 | [[transformer]] (1706.03762) |
| 2018 | [[gpt-1]] and [[bert]]: pre-train + fine-tune ([[pre-training]]) |
| 2019 | [[gpt-2]]: zero-shot transfer |
| 2020-01 | [[scaling-laws]] (Kaplan, 2001.08361) |
| 2020-05 | [[gpt-3]]: 175B parameters; [[in-context-learning]]; the prompting era ([[prompt-engineering]]) |
| 2021-07 | Prompting formalized as NLP's fourth paradigm ([[pre-train-prompt-predict-liu-2021]]) |
| 2022-03 | [[instructgpt]]: SFT + [[rlhf]] → ChatGPT (Nov 2022) |
| 2022-03 | Chinchilla compute-optimal scaling ([[chinchilla-hoffmann-2022]]) |
| 2022-10 → | LLM agents ([[react]], [[llm-agent]]) |
| 2023–2024 | Long context ([[long-context-llm]]), tools ([[function-calling]]), open weights ([[llama]]) |
| 2024-09 → | Reasoning models ([[o1]], [[deepseek-r1]], [[large-reasoning-model]]) |

## Hubs

- **Families:** [[gpt-family]], [[claude]], [[llama]], [[gemma]], [[gemini-1-5]]
- **Training:** [[pre-training]], [[fine-tuning]], [[instruction-tuning]], [[rlhf]], [[scaling-laws]]
- **Use:** [[in-context-learning]], [[prompt-engineering]], [[context-engineering]], [[retrieval-augmented-generation]], [[llm-agent]]
- **Overview:** [[from-transformer-to-modern-agents]], with nine evolutionary tracks

## Related

[[deep-learning]] · [[transformer]] · [[gpt-family]] · [[llm-agent]] · [[artificial-intelligence]]

## References

- [[ai-ml-dl-difference-nvidia-2016]]
- [[ai-vs-ml-vs-dl-vs-nn-ibm]]
- [[pre-train-prompt-predict-liu-2021]]
