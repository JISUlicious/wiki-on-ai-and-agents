---
title: "Self-Consistency Improves Chain of Thought Reasoning in Language Models"
type: source
created: 2026-05-14
updated: 2026-05-14
sources:
  - wang-self-consistency-2022.md
arxiv_id: "2203.11171"
venue: ICLR 2023
authors:
  - Xuezhi Wang
  - "[[jason-wei]]"
  - Dale Schuurmans
  - "[[quoc-le]]"
  - Ed H. Chi
  - Sharan Narang
  - Aakanksha Chowdhery
  - Denny Zhou
first_author: Xuezhi Wang
introduces:
  - "[[self-consistency]]"
year: 2022
tags:
  - 2022
status: complete
importance: high
---

# Self-Consistency Improves Chain of Thought Reasoning in Language Models

**Source**: `sources/wang-self-consistency-2022.md` (extracted from `wang-self-consistency-2022.pdf`)
**arXiv**: [2203.11171](https://arxiv.org/abs/2203.11171)
**Venue**: ICLR 2023
**Authors**: Xuezhi Wang (first), [[jason-wei]], Dale Schuurmans, [[quoc-le]], Ed H. Chi, Sharan Narang, Aakanksha Chowdhery, Denny Zhou ([[google-research|Google Brain]])

## Summary

This paper replaces the greedy decoding used in [[chain-of-thought-prompting|chain-of-thought prompting]] with a **sample-and-marginalize** decoding strategy called **self-consistency**. The procedure:

1. Sample many (~40) diverse reasoning paths from the LLM at non-zero temperature, all from the same CoT prompt.
2. Each path produces an answer (the final numeric / categorical / symbolic answer).
3. Take the **majority vote** across answers — the answer that the most reasoning paths converged on.

The intuition: a complex reasoning problem typically admits multiple valid reasoning paths leading to the same correct answer. If the model is competent but noisy, the correct answer should be the modal outcome across diverse samples. If the model is incompetent, the answers will be all over the place.

Results on InstructGPT and PaLM with 8-shot CoT prompts:
- **GSM8K**: +17.9%
- **SVAMP**: +11.0%
- **AQUA-RAT**: +12.2%
- **StrategyQA**: +6.4%
- **ARC-Challenge**: +3.9%

Self-consistency was one of the **earliest, most general inference-time-scaling techniques** — pay more inference compute (more samples) for more accuracy. It's the conceptual ancestor of every later test-time-compute method: best-of-N sampling, process reward models, the OpenAI o1 reasoning models (Sep 2024), Anthropic's reasoning Claude variants, and DeepSeek's R1 reasoning model.

Crucially, self-consistency doesn't require any model changes, fine-tuning, or auxiliary models. It's a pure decoding-time technique that wraps existing CoT-prompted models.

## Key Points

- **Sample N diverse reasoning paths** at temperature > 0 (typically T=0.7, N=40).
- **Marginalize** by majority-voting the final answers.
- **Self-consistent**: the answer is the one that most paths agree on, regardless of which intermediate steps got there.
- **Plug-in compatible**: wraps any CoT prompt; no model changes needed.
- **Test-time compute trade-off**: roughly N× the inference cost of greedy CoT, with substantial accuracy gains for hard reasoning tasks.
- **Stronger at scale**: gains grow with both N (samples) and model size. On GSM8K, PaLM 540B + self-consistency exceeds the model's previously-reported few-shot greedy CoT score by ~18 points.

## Entities Mentioned

- Xuezhi Wang — first author
- [[jason-wei]] — co-author (also [[chain-of-thought-wei-2022|CoT paper]])
- [[quoc-le]] — senior author
- Denny Zhou — senior author (CoT and reasoning research lead at Google)
- [[google-research]] — author affiliation

## Concepts Discussed

- [[self-consistency]] — the technique
- [[chain-of-thought-prompting]] — what it wraps
- Inference-time scaling / test-time compute

## Notable Quotes

> "We propose a new decoding strategy, self-consistency, to replace the naive greedy decoding used in chain-of-thought prompting. It first samples a diverse set of reasoning paths instead of only taking the greedy one, and then selects the most consistent answer by marginalizing out the sampled reasoning paths." — Abstract

## References

_Original source: `sources/wang-self-consistency-2022.md`_
