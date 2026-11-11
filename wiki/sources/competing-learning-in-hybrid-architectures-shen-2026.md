---
title: "Competing Learning in Hybrid Architectures"
type: source
created: 2026-11-10
updated: 2026-11-10
sources:
  - competing-learning-in-hybrid-architectures-shen-2026.md
arxiv_id: "2610.26830"
venue: arXiv preprint (cs.CL)
authors:
  - Xingyu Shen
  - Yingfa Chen
  - Ziqing Qiao
  - Chaojun Xiao
  - Xu Han
  - Zhiyuan Liu
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# Competing Learning in Hybrid Architectures

**Source:** Xingyu Shen, Yingfa Chen, Ziqing Qiao, Chaojun Xiao, Xu Han, Zhiyuan Liu ([[tsinghua-university|Tsinghua]]) — [arXiv:2610.26830](https://arxiv.org/abs/2610.26830), cs.CL, v1 19 Oct 2026. Code: github.com/thunlp/competing-learning.

## Summary

Hybrid LMs that mix [[linear-attention|linear recurrence]] layers with softmax attention (Kimi K3, GLM-5.3-Flash, [[qwen3.8-flash-next|Qwen3.8-Flash-Next]], Nemotron-3-Ultra) are meant to split the work: attention does long-range in-context retrieval and recurrence does cheap short-range mixing. This paper finds that standard pre-training undermines that split. **Competing Learning**: both mixer types can learn retrieval, and whichever learns it first reduces the retrieval error and so starves the other of gradient signal. Natural text has more short- than long-distance patterns, and Kimi Delta Attention (KDA) layers start with a short-range bias (low forget-gate init), so recurrence usually wins and attention under-learns retrieval. The fix, **Better Layer Division (BLD)**, freezes the recurrent layers for an initial stage so attention gets a head start.

## Key points

- **Diagnostic.** A synthetic "Spaced Retrieval" copy task (gap g = 64 or 8192) and a per-layer *retrieval ability score*. The score is computed by patching a layer's mixer output with the output from an input that has a different source digit and measuring the drop in the correct-answer logit margin. This is close to [[mechanistic-interpretability|activation patching]].
- **Evidence.** Freezing KDA mid-training lets attention start learning retrieval immediately, and attention gradient norms rise to ~10× the baseline. Unfreezing KDA early enough lets it take retrieval back. Unfreezing it after attention has a big enough lead makes the shift stick. The data mix decides the winner: more long-gap examples favour attention.
- **BLD setup.** KDA–attention hybrids from 180M to 1B, with a 3:1 KDA:attention pattern and NoPE. Trained on DCLM at Chinchilla-style budgets ([[chinchilla]]) with 16K context. KDA is frozen for about 1/10 of the later unfrozen phase (AdamW, then Muon). The architecture is unchanged.
- **Results at 1B** (vanilla → BLD): loss 2.874 → 2.868; commonsense avg 48.69 → 49.45; recall suite 36.50 → 38.14; [[ruler|RULER]] 32.75 → 36.04; LongPPL 7.69 → 6.50. Retrieval gains are large at every scale (e.g. RULER +4.2 at 510M), while CSR gains are small and slightly negative at 510M. Fitted [[scaling-laws]] suggest the loss advantage persists at larger scale.
- **Transfers** to [[gated-deltanet|Gated DeltaNet]] and GLA hybrids.
- **Mechanism check.** DuoAttention-style α analysis shows BLD produces more retrieval heads (3 critical heads vs 1 in vanilla).
- The authors call BLD a preliminary fix. It was tested only up to 1B, and no long-context continual-training or post-training interaction was studied.

## Concepts & entities

- [[linear-attention]] · [[gated-deltanet]] · [[state-space-model]] · [[recurrent-neural-network]] · [[self-attention]] · [[attention-mechanism]]
- [[long-context-llm]] · [[ruler]] · [[needle-in-a-haystack]] · [[in-context-learning]] · [[pre-training]] · [[scaling-laws]]
- Hybrid-model entities: [[qwen3.8-flash-next]] · [[jamba]] · [[griffin]] · [[mamba]]
- Org: [[tsinghua-university]]

## References

- [arXiv:2610.26830](https://arxiv.org/abs/2610.26830)
