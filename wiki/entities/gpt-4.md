---
title: GPT-4
type: entity
created: 2026-05-13
updated: 2026-05-13
sources:
  - gpt-4-openai-2023.md
family: gpt
predecessor: "[[gpt-3]]"
successors: []
parameters: "undisclosed"
release_date: 2023-03-14
status: complete
importance: high
tags:
  - model
---

# GPT-4

GPT-4 is [[openai|OpenAI]]'s 2023 flagship LLM. It accepts text and image inputs and produces text output, and was introduced in the "[[gpt-4-openai-2023|GPT-4 Technical Report]]" (OpenAI, March 2023).

## What is known

- **Multimodal input**: text + images; text-only output.
- **[[transformer-architecture|Transformer]]-based**, pre-trained on next-token prediction over web text + licensed data.
- **Post-trained** with [[rlhf|RLHF]] plus rule-based reward models (RBRMs).
- **Context window**: 8k tokens initially; a 32k variant followed shortly. (Subsequent GPT-4 Turbo and GPT-4o variants extended to 128k.)
- **Predictable scaling**: built on infrastructure that predicted final loss within ~0.5% and HumanEval pass rate accurately from runs 1,000–10,000× smaller.

## What is not disclosed

The technical report explicitly withholds:

- Parameter count.
- Architecture details beyond "Transformer-based" and "decoder-only" (implied).
- Training compute.
- Training data composition.
- Training algorithm.

This marked a sharp public departure from earlier GPT papers — see the [[gpt-4-openai-2023|technical report]] for the official rationale (competitive landscape + safety). Various leaks and analyses have speculated about size (often quoted as a Mixture-of-Experts in the 1.7T-total-parameter range), but none are officially confirmed.

## Lineage

| Predecessor | Successor |
|---|---|
| [[gpt-3]] (175B) → GPT-3.5 (intermediate; powers original ChatGPT) | GPT-4 Turbo (Nov 2023), GPT-4o (May 2024, fully multimodal), o-series reasoning models (2024–), GPT-5 (2025–) |

## Capability highlights (from the technical report)

- **Simulated bar exam**: top 10% (vs. GPT-3.5's bottom 10%).
- **SAT**: 1410/1600.
- **LSAT**: 88th percentile.
- **USMLE Step 1**: ~75%.
- **[[mmlu|MMLU]]**: 86.4% (5-shot).
- **[[humaneval|HumanEval]]**: 67% pass@1.
- **AP Calculus BC**: 4/5.
- **Codeforces**: ~5th percentile (notable weak point at competitive programming, c. 2023).

## Acknowledged limitations

- [[hallucination|Hallucinates]] confidently.
- Limited context window relative to many tasks.
- Does not learn from experience (each interaction starts fresh).
- Sensitive to prompt phrasing.

## References

- [[gpt-4-openai-2023]]
