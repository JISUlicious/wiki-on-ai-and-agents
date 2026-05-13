---
title: "Chain-of-Thought Prompting Elicits Reasoning in Large Language Models"
type: source
created: 2026-05-13
updated: 2026-05-13
sources:
  - chain-of-thought-wei-2022.md
arxiv_id: "2201.11903"
venue: NeurIPS 2022
authors:
  - Jason Wei
  - Xuezhi Wang
  - Dale Schuurmans
  - Maarten Bosma
  - Brian Ichter
  - Fei Xia
  - Ed H. Chi
  - Quoc V. Le
  - Denny Zhou
year: 2022
tags:
  - 2022
status: complete
importance: high
---

# Chain-of-Thought Prompting Elicits Reasoning in Large Language Models

**Source**: `sources/chain-of-thought-wei-2022.md` (extracted from `Chain-of-Thought Wei 2022 2201.11903.pdf`)
**arXiv**: [2201.11903](https://arxiv.org/abs/2201.11903)
**Venue**: NeurIPS 2022
**Authors**: [[jason-wei]] (first), Xuezhi Wang, Dale Schuurmans, Maarten Bosma, Brian Ichter, Fei Xia, Ed H. Chi, Quoc V. Le, Denny Zhou ([[google-research|Google Research, Brain Team]])

## Summary

This paper introduces **chain-of-thought (CoT) prompting**: a prompting technique in which few-shot exemplars in the prompt include not just `(input, output)` pairs but `(input, chain-of-thought, output)` *triples*, where the chain of thought is a natural-language sequence of intermediate reasoning steps. The model is then asked to produce a similar chain of thought before answering the test input. The effect on multi-step reasoning tasks is dramatic.

The paper's most-cited result: prompting [[palm|PaLM 540B]] with just **eight** CoT exemplars achieves 57% on the GSM8K math-word-problem benchmark — **better than fine-tuned [[gpt-3]] 175B with a separate verifier** (which scored 55%). The gains hold across arithmetic, commonsense (CSQA, StrategyQA), and symbolic reasoning (last-letter concatenation, coin flip) benchmarks, on three different LLMs (LaMDA 137B, GPT-3 175B, PaLM 540B).

Crucially, **CoT is an emergent ability of scale**: on smaller models (<~100B) CoT prompting often *hurts* performance because the model produces inconsistent or fabricated reasoning. On large-enough models, CoT pushes performance dramatically above standard prompting. This made CoT the canonical example of a "capability that emerges with scale."

After this paper, CoT became one of the most-used prompting techniques in production LLM applications. It also motivated a large line of follow-up work: zero-shot CoT ("Let's think step by step", Kojima et al. 2022), self-consistency (Wang et al. 2022), tree-of-thoughts, program-of-thoughts, and the broader "reasoning model" wave.

## Key Points

- **The technique**: few-shot prompt format becomes `(input, chain-of-thought, output)` triples instead of `(input, output)` pairs. No fine-tuning, no model changes.
- **Attractive properties** per the paper:
  1. Allocates *extra compute* to harder problems (longer chains).
  2. Provides an *interpretable* trace of the model's "reasoning" (caveat: not necessarily faithful to internal computation).
  3. Applies to any task expressible in natural language.
  4. Works "off the shelf" — sufficiently large pre-trained LMs do it without any fine-tuning.
- **Emergence with scale**: CoT helps for ≥~100B-parameter models; hurts smaller ones. This is the iconic emergent-capability example.
- **Best result**: PaLM 540B + 8-shot CoT achieves SOTA on GSM8K (57%), surpassing GPT-3 175B fine-tuned with a verifier.
- **Tasks tested**: GSM8K, MultiArith, ASDiv, AQUA, SVAMP, MAWPS (arithmetic); CSQA, StrategyQA (commonsense); last-letter, coin-flip (symbolic).
- **Models tested**: LaMDA (8B–137B), [[gpt-3]] (350M–175B), PaLM (8B–540B).

## Entities Mentioned

- [[jason-wei]] — first author, will later be a prominent voice on LLM capabilities
- [[google-research]] — Brain Team
- [[palm]] — Google's 540B-parameter model, used in the headline results
- [[gpt-3]] — compared against

## Concepts Discussed

- [[chain-of-thought-prompting]] — the technique
- [[in-context-learning]] / [[few-shot-learning]] — the broader prompting paradigm
- [[emergent-abilities]] — CoT as a canonical example
- [[scaling-laws]] — adjacent: capabilities emerge as scale grows

## Notable Quotes

> "We explore how generating a chain of thought—a series of intermediate reasoning steps—significantly improves the ability of large language models to perform complex reasoning." — Abstract

> "Chain-of-thought reasoning can be readily elicited in sufficiently large off-the-shelf language models simply by including examples of chain of thought sequences into the exemplars of few-shot prompting." — §2

> "Prompting a PaLM 540B with just eight chain-of-thought exemplars achieves state-of-the-art accuracy on the GSM8K benchmark of math word problems, surpassing even finetuned GPT-3 with a verifier." — Abstract

## References

_Original source: `sources/chain-of-thought-wei-2022.md`_
