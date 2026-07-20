---
title: "Does quantization affect models' performance on long-context tasks?"
type: source
created: 2026-07-07
updated: 2026-07-07
sources: []
arxiv_id: "2505.20276"
venue: EMNLP 2025
year: 2025
authors:
  - Anmol Mekala
tags:
  - 2025
  - paper
status: complete
importance: high
---

# Does Quantization Affect Models' Performance on Long-Context Tasks?

**Mekala et al.** — [arXiv:2505.20276](https://arxiv.org/abs/2505.20276), **EMNLP 2025**.

9.7K test examples; 5 methods; 5 models (Llama-3.1 8B/70B, Qwen-2.5 7B/32B/72B); RULER, ONERULER (26 languages), NOCHA, FACTSCORE, CS4.

## The headline: damage is context-length-dependent

RULER accuracy delta vs BF16:

| Method | 8K | 64K | 128K |
|---|---|---|---|
| FP8 | -1.9 | -1.3 | **-0.7** |
| GPTQ-int8 | -0.9 | -0.7 | -2.0 |
| AWQ-int4 | -0.6 | -6.4 | -5.8 |
| GPTQ-int4 | -0.2 | -4.5 | **-12 to -13** |
| BNB-nf4 | -0.2 | -0.9 | **-23** |

**Identical models and quantization: ~0% damage at 8K, up to -23 points at 128K.** Any evaluation done at short context will completely miss this.

Average across all tasks: FP8 -0.2%, GPTQ-int8 -0.8%, AWQ-int4 -1.8%, GPTQ-int4 -2.7%, BNB-nf4 **-6.9%**.

## Other findings

- **Non-English degrades up to 5x more than English.**
- Up to **59% loss** on ONERULER for Llama-3.1-70B under BNB-nf4.
- Strong model dependence: Qwen-2.5-72B is robust under BNB-nf4 while Llama-3.1-70B loses **32%** on the same task - so format recommendations do not transfer across model families.

## Concepts

- [[quantization-quality]] · [[quantization]] · [[long-context-llm]] · [[kv-cache]]

## References

- [arXiv:2505.20276](https://arxiv.org/abs/2505.20276)

