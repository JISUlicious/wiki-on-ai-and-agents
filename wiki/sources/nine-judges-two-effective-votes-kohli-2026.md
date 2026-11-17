---
title: "Nine Judges, Two Effective Votes: Correlated Errors Undermine LLM Evaluation Panels"
type: source
created: 2026-11-17
updated: 2026-11-17
sources:
  - nine-judges-two-effective-votes-kohli-2026.md
arxiv_id: "2605.29800"
venue: arXiv preprint (cs.CL)
authors:
  - Guneet Kohli
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# Nine Judges, Two Effective Votes: Correlated Errors Undermine LLM Evaluation Panels

**Source:** Guneet Kohli ([[apple|Apple]]) — [arXiv:2605.29800](https://arxiv.org/abs/2605.29800), cs.CL, v1 28 May 2026. Single-author preprint.

## Summary

Panels of [[llm-as-a-judge|LLM judges]] from different providers are meant to approximate independent votes. By the logic of the Condorcet Jury Theorem, adding judges should then push majority-vote accuracy toward certainty. This paper measures how much independence such a panel actually has.

The panel has 9 judges from 7 families: GPT-4o, GPT-4o-mini, Claude Sonnet 4.5, Gemini 2.5 Pro, Llama 4 Maverick and Scout, Qwen3-32B, Mistral Large 3 and DeepSeek-V3. They judge 1,000 items each from the ChaosNLI versions of MNLI, SNLI and AlphaNLI, where every item has 100 human labels. Two tools do the measuring. The **Kish effective sample size** n_eff = k / (1 + (k − 1)·φ̄) is computed from pairwise correlations of the judges' errors. A **Condorcet null model** simulates what majority vote would score if the same judges voted independently, controlling for item difficulty.

## Key points

| Metric | MNLI | SNLI | AlphaNLI |
|---|---|---|---|
| n_eff (Kish) | 2.18 | 2.35 | 2.48 |
| Mean error correlation φ̄ | 0.391 | 0.354 | 0.328 |
| Panel accuracy | 72.0% | 77.7% | 88.7% |
| Best single judge | 71.8% | 84.2% | 91.2% |
| Condorcet gap | 22.0 pp | 14.0 pp | 7.6 pp |

- **About two votes, not nine.** The panel's independence ratio is 24.2% (MNLI). Roughly three quarters of its nominal independence is lost because the judges make the same mistakes on the same items.
- **The best single judge matches or beats the panel.** The panel's lift is +0.2 pp on MNLI (within noise), −6.5 pp on SNLI and −2.5 pp on AlphaNLI.
- **The errors pile up at the extremes.** On MNLI, 290 items have all 9 judges right and 51 have all 9 wrong, against fewer than 1 expected under independence. Even on unanimous items, accuracy is 90.9%, not the ~99.99% Condorcet predicts.
- **Item difficulty is not the explanation.** Shared difficulty accounts for only 6.8% of the MNLI gap, or 13.5% with 10 bins. A stratified permutation test gives p < 10⁻⁴ (z = 65.6).
- **More judges don't help.** n_eff levels off at about 2.3–3.1 as judges are added, with little benefit beyond 5.
- **Better aggregation doesn't help either.** Dawid–Skene EM and accuracy-weighted voting close at most 11% of the Condorcet gap, even with oracle access to gold labels.
- **The result is robust.** It holds across prompt variants, temperatures and chain-of-thought, and on a pairwise preference task (RewardBench, n_eff = 2.0).

## Why it matters

Adding more judges from similar training pipelines buys little extra information, and a unanimous verdict is weaker evidence than it looks. The author argues the fix is judges that genuinely reason differently, not larger panels or smarter vote-weighting. This finding complements PoLL, which compared a panel only against the *average* single judge.

## Concepts & entities

- [[llm-as-a-judge]] · [[agent-evaluation]] · [[calibration]].
- [[multi-agent]] — the "diverse ensemble" assumption fails when model errors are correlated.
- Related sources: [[judging-llm-as-a-judge-zheng-2023]] · [[reliability-without-validity-llm-as-a-judge-norman-2026]].
- [[apple]] · [[claude]] · [[qwen]] · [[deepseek-v3]].

## References

- [arXiv:2605.29800](https://arxiv.org/abs/2605.29800)
