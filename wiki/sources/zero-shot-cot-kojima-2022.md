---
title: "Large Language Models are Zero-Shot Reasoners (Zero-shot CoT)"
type: source
created: 2026-05-14
updated: 2026-05-14
sources:
  - kojima-zeroshot-cot-2022.md
arxiv_id: "2205.11916"
venue: NeurIPS 2022
authors:
  - Takeshi Kojima
  - Shixiang Shane Gu
  - Machel Reid
  - Yutaka Matsuo
  - Yusuke Iwasawa
first_author: Takeshi Kojima
introduces:
  - "[[zero-shot-cot]]"
year: 2022
tags:
  - 2022
status: complete
importance: high
---

# Large Language Models are Zero-Shot Reasoners (Zero-shot CoT)

**Source**: `sources/kojima-zeroshot-cot-2022.md` (extracted from `kojima-zeroshot-cot-2022.pdf`)
**arXiv**: [2205.11916](https://arxiv.org/abs/2205.11916)
**Venue**: NeurIPS 2022
**Authors**: Takeshi Kojima (University of Tokyo), Shixiang Shane Gu ([[google-research|Google Research]]), Machel Reid, Yutaka Matsuo, Yusuke Iwasawa (UTokyo)

## Summary

A famously short and surprising paper. Where [[chain-of-thought-wei-2022|Wei et al. (2022)]] showed that **few-shot** CoT prompting dramatically improves reasoning, this paper shows that **simply appending the phrase "Let's think step by step"** to a zero-shot prompt elicits comparable chain-of-thought reasoning — without any in-context examples.

Concretely, the technique is a two-stage prompt:

1. **Reasoning extraction**: `Q: ... A: Let's think step by step.` The LLM produces a chain of reasoning.
2. **Answer extraction**: feed the reasoning back into the model with `Therefore, the answer (arabic numerals) is ` and let it finish.

This trivially-simple prompt template lifts InstructGPT (text-davinci-002) from **17.7% to 78.7%** on MultiArith and from **10.4% to 40.7%** on GSM8K. Similar gains on AQUA-RAT, SVAMP, Last Letter, Coin Flip, Date Understanding, Tracking Shuffled Objects. Comparable gains on 540B [[palm|PaLM]].

The paper's broader claim: large LMs harbor **latent zero-shot reasoning capabilities** that the standard "Q: ... A:" prompt format fails to elicit. A single trigger phrase ("Let's think step by step") flips a switch. Versatility across very diverse reasoning tasks with one template hints at "high-level, multi-task broad cognitive capabilities extractable by simple prompting."

Zero-shot CoT became the default productionized version of chain-of-thought reasoning — no exemplars to curate, just a magic phrase. The technique is built into most modern chat LLMs' default behavior. The success of this dramatically simple prompt motivated a huge wave of "what other latent capabilities are buried in trigger phrases" research.

## Key Points

- **The trigger phrase**: `"Let's think step by step"` — appended to a zero-shot Q&A prompt.
- **Two-stage prompt structure**: (1) reasoning extraction with the trigger, (2) answer extraction with `"Therefore, the answer is "`. The second stage is needed because raw chain-of-thought outputs don't always end with a clean answer.
- **Result**: MultiArith 17.7% → 78.7%; GSM8K 10.4% → 40.7% on InstructGPT.
- **Tested**: 12 reasoning datasets across arithmetic, symbolic, and logical reasoning.
- **Scale dependence**: works best on large models (>~100B). Smaller models don't benefit; some get hurt.
- **Versatility**: a single prompt template generalizes across many task types — unusual for CoT-era techniques.

## Entities Mentioned

- Takeshi Kojima — first author (UTokyo PhD)
- Shixiang Shane Gu — co-author (Google Brain at the time, later DeepMind)
- [[google-research]] — co-author affiliation

## Concepts Discussed

- [[zero-shot-cot]] — the technique
- [[chain-of-thought-prompting]] — what this generalizes to zero-shot
- [[zero-shot-learning]] — the evaluation regime
- [[emergent-abilities]] — zero-shot CoT helps only at scale

## Notable Quotes

> "We show that LLMs are decent zero-shot reasoners by simply adding 'Let's think step by step' before each answer." — Abstract

> "The versatility of this single prompt across very diverse reasoning tasks hints at untapped and understudied fundamental zero-shot capabilities of LLMs, suggesting high-level, multi-task broad cognitive capabilities may be extracted by simple prompting." — Abstract

## References

_Original source: `sources/kojima-zeroshot-cot-2022.md`_
