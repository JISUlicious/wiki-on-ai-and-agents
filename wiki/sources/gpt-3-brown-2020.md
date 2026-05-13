---
title: "Language Models are Few-Shot Learners (GPT-3)"
type: source
created: 2026-05-13
updated: 2026-05-13
sources:
  - gpt-3-brown-2020.md
arxiv_id: "2005.14165"
venue: NeurIPS 2020 (Best Paper)
authors:
  - Tom B. Brown
  - Benjamin Mann
  - Nick Ryder
  - Melanie Subbiah
  - Jared Kaplan
  - Prafulla Dhariwal
  - Arvind Neelakantan
  - Pranav Shyam
  - Girish Sastry
  - Amanda Askell
  - Sandhini Agarwal
  - Ariel Herbert-Voss
  - Gretchen Krueger
  - Tom Henighan
  - Rewon Child
  - Aditya Ramesh
  - Daniel M. Ziegler
  - Jeffrey Wu
  - Clemens Winter
  - Christopher Hesse
  - Mark Chen
  - Eric Sigler
  - Mateusz Litwin
  - Scott Gray
  - Benjamin Chess
  - Jack Clark
  - Christopher Berner
  - Sam McCandlish
  - Alec Radford
  - Ilya Sutskever
  - Dario Amodei
year: 2020
tags:
  - 2020
status: complete
importance: high
---

# Language Models are Few-Shot Learners (GPT-3)

**Source**: `sources/gpt-3-brown-2020.md` (extracted from `GPT-3 Brown 2020.pdf`)
**arXiv**: [2005.14165](https://arxiv.org/abs/2005.14165)
**Venue**: NeurIPS 2020 (Best Paper Award)
**Authors**: [[tom-brown]] (first), Benjamin Mann, Nick Ryder, Melanie Subbiah, [[jared-kaplan]], Prafulla Dhariwal, ..., [[alec-radford]], [[ilya-sutskever]], [[dario-amodei]] ([[openai|OpenAI]]; Kaplan also at Johns Hopkins)

## Summary

This is the GPT-3 paper. The headline result is that a 175-billion-parameter [[decoder-only-transformer|decoder-only Transformer]] (10× larger than any previous non-sparse LM) becomes a **competent few-shot learner**: tasks can be specified with just a natural-language description plus a handful of `(input, output)` demonstrations in the prompt — no gradient updates, no fine-tuning — and the model performs at levels approaching or exceeding fine-tuned task-specific systems on many NLP benchmarks.

The paper introduces the term **[[in-context-learning|in-context learning]]** to describe this inner-loop adaptation: the model's pre-training acts as an outer-loop meta-learning over many implicit tasks, and at inference time the prompt's demonstrations select and refine an in-context task representation. Crucially, **few-shot performance improves much faster with scale than zero-shot performance does** — larger models are not just better at language modeling, they are better at *learning from context*.

GPT-3 is trained on a curated mixture of Common Crawl (heavily filtered), WebText2, Books1, Books2, and Wikipedia — roughly 300 billion tokens. The paper evaluates 8 model sizes from 125M to 175B parameters across 24+ NLP datasets in zero/one/few-shot settings, plus novel synthetic tasks (word unscrambling, novel-word usage, 3-digit arithmetic). It also includes substantial discussion of broader impacts — misuse, fairness, energy — and a fairly direct admission that human evaluators cannot reliably distinguish GPT-3-generated news articles from human-written ones.

This paper is a watershed: it shifted the field's center of gravity from fine-tuning to prompting, motivated the [[scaling-laws|scaling laws]] research agenda, and was the technical foundation for the subsequent product wave (Codex, GitHub Copilot, ChatGPT).

## Key Points

- **Scale**: 175B parameters; 8 model sizes from 125M up to 175B trained for direct scaling analysis.
- **Architecture**: standard GPT-style [[decoder-only-transformer|decoder-only Transformer]] (same as [[gpt-2]] but with alternating dense and locally-banded sparse attention patterns, à la Sparse Transformer). 96 layers, dmodel = 12288, 96 heads, dhead = 128. Context length 2048.
- **Training corpus**: ~300B tokens. Common Crawl (filtered + dedup + classifier-curated, ~410B tokens, sampled at 60%), WebText2 (22%), Books1 (8%), Books2 (8%), Wikipedia (3%).
- **Compute**: largest model trained on ~3640 PetaFLOP/s-days. Largest training run to that point in NLP.
- **[[in-context-learning|In-context learning]]**: tasks specified in the prompt with zero, one, or "few" (typically 10–100) demonstrations.
- **Findings**:
  - Few-shot ≫ zero-shot, and the gap widens with model size — confirming the meta-learning hypothesis.
  - SOTA or near-SOTA on translation (especially X→English), Cloze, completion, common-sense reasoning, reading comprehension, [[squad|SQuAD]], LAMBADA.
  - **3-digit arithmetic emerges around 13B** — a striking demonstration of capability emergence with scale.
  - Struggles on natural language inference, some reading comprehension (RTE-like), bidirectional tasks (where masking helps).
- **Limitations explicit**: methodological issues with benchmark contamination via Common Crawl; weaknesses in NLI; unidirectional context limitations; the model "loses coherence" on long generations; world-knowledge biases.
- **Broader impacts section**: misuse potential (disinformation, spam, fraud), bias (gender, race, religion), energy footprint. First major LM paper to discuss this at length.

## Entities Mentioned

- [[tom-brown]] — first author
- [[jared-kaplan]] — co-author; lead of the [[scaling-laws]] paper (Kaplan et al. 2020)
- [[alec-radford]], [[ilya-sutskever]], [[dario-amodei]] — OpenAI senior authors
- [[openai]] — origin org
- [[gpt-3]] — the model

## Concepts Discussed

- [[in-context-learning]] — coined here
- [[few-shot-learning]] / [[zero-shot-learning]] — the evaluation regime
- [[scaling-laws]] — both implicitly (via the 8-size sweep) and explicitly (citing Kaplan et al.)
- [[autoregressive-language-modeling]] — the only training objective
- [[transformer-architecture]] — standard with sparse-attention variant
- [[hallucination]] — implicitly discussed in "loses coherence" and bias sections

## Notable Quotes

> "Here we show that scaling up language models greatly improves task-agnostic, few-shot performance, sometimes even reaching competitiveness with prior state-of-the-art fine-tuning approaches." — Abstract

> "Larger models make increasingly efficient use of in-context information." — Figure 1.2 caption

> "We use the term 'in-context learning' to describe the inner loop of this [meta-learning] process, which occurs within the forward-pass upon each sequence." — §1

## References

_Original source: `sources/gpt-3-brown-2020.md`_
_OpenAI blog: https://openai.com/blog/gpt-3-apps/ (the paper effectively launched the OpenAI API)_
