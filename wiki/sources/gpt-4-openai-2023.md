---
title: "GPT-4 Technical Report"
type: source
created: 2026-05-13
updated: 2026-05-13
sources:
  - gpt-4-openai-2023.md
arxiv_id: "2303.08774"
venue: OpenAI technical report
authors:
  - OpenAI
year: 2023
tags:
  - 2023
status: complete
importance: high
---

# GPT-4 Technical Report

**Source**: `sources/gpt-4-openai-2023.md` (extracted from `GPT-4 Tech report 2303.08774.pdf`)
**arXiv**: [2303.08774](https://arxiv.org/abs/2303.08774)
**Venue**: [[openai|OpenAI]] technical report, March 2023
**Authors**: OpenAI (corporate authorship; individual contributors listed in appendix)

## Summary

This is the technical report accompanying the release of **[[gpt-4]]**, OpenAI's flagship multimodal LLM. GPT-4 accepts image and text inputs and produces text output, and is positioned as a substantial capability jump over GPT-3.5 — most viscerally illustrated by the simulated bar exam result: GPT-3.5 scores in the **bottom 10%**, GPT-4 in the **top 10%** of test takers.

The report is notable for what it **omits**. Citing "the competitive landscape and the safety implications of large-scale models," OpenAI declined to disclose architecture, parameter count, training compute, training data, or training method. This marked a sharp public departure from the open-ish era of [[gpt-2]] / [[gpt-3]] and set the closed-frontier-lab norm that subsequent releases (PaLM 2, Gemini, Claude 2+) largely followed.

What the report **does** cover in depth:

1. **Predictable scaling**: OpenAI built infrastructure that allowed them to predict GPT-4's final pre-training loss and even some downstream capabilities (HumanEval pass rate) from models trained with 1,000–10,000× less compute. The fitted scaling laws (loss as a power law in compute, plus an irreducible-loss term) accurately predicted the final run. This is a substantial methodological contribution and a sign of the engineering maturity of frontier training runs.
2. **Capability evaluations**: a broad battery of professional and academic exams (bar exam, LSAT, SAT, AP exams, GRE, USMLE, Codeforces, etc.), [[mmlu|MMLU]], [[humaneval|HumanEval]], and traditional NLP benchmarks. GPT-4 surpasses GPT-3.5 dramatically on most.
3. **Multilingual**: outperforms English-language SOTA on MMLU translated into 24 of 26 languages.
4. **Multimodal evals**: image-input performance on VQA-style benchmarks (preview; full image-input deployment came later in 2023).
5. **Post-training alignment**: [[rlhf|RLHF]] and "rule-based reward models" (RBRMs) are used to improve factuality and adherence to desired behavior.
6. **System card**: an extensive (~50-page) appendix covering bias, disinformation, over-reliance, privacy, cybersecurity, weapons proliferation risks, and the safety interventions OpenAI made — including pre-deployment red-teaming with domain experts (e.g., chemical and biological weapons specialists).

## Key Points

- **Multimodal input**: text + images; text-only output.
- **Training**: pre-training on next-token prediction + post-training [[rlhf|RLHF]] + rule-based reward models.
- **No architectural disclosure**: no parameter count, no architecture details, no training compute, no data composition. (Substantive details about subsequent models like GPT-4o, GPT-4 Turbo, and successors are similarly closed.)
- **Predictable scaling**: predicted final pre-training loss within ~0.5% and HumanEval pass-rate buckets accurately, from much smaller runs. Validates the operational use of [[scaling-laws|scaling laws]] for safety-relevant capability forecasting.
- **Exam performance**: bar exam top 10%; SAT 1410/1600; LSAT 88th percentile; USMLE Step 1 ~75%; AP Calculus BC 4/5; Codeforces ~5th percentile (still bad at competitive programming).
- **MMLU**: 86.4% (5-shot), substantial gain over GPT-3.5's 70%.
- **HumanEval**: 67% pass@1 (no CoT) — major jump in code generation.
- **Acknowledged limitations**: [[hallucination|hallucination]], limited context window (originally 8k; 32k variant came later), no learning from experience, sensitive to prompt phrasing.
- **Safety interventions**: model-assisted safety pipeline, RBRMs, extensive system card on real-world risk surfaces.

## Entities Mentioned

- [[openai]] — author / origin
- [[gpt-4]] — the model
- [[gpt-3]] (and GPT-3.5) — comparison baselines

## Concepts Discussed

- [[rlhf]] — central to post-training
- [[multimodal-llm]] — GPT-4 is the headline multimodal flagship of its era
- [[scaling-laws]] — predictable scaling section
- [[mmlu]], [[humaneval]] — benchmarks
- [[hallucination]] — explicitly acknowledged as ongoing limitation

## Notable Quotes

> "We report the development of GPT-4, a large-scale, multimodal model which can accept image and text inputs and produce text outputs." — Abstract

> "Given both the competitive landscape and the safety implications of large-scale models like GPT-4, this report contains no further details about the architecture (including model size), hardware, training compute, dataset construction, training method, or similar." — §2

> "These improvements allowed us to reliably predict some aspects of the performance of GPT-4 from smaller models trained using 1,000× – 10,000× less compute." — §3

## References

_Original source: `sources/gpt-4-openai-2023.md`_
