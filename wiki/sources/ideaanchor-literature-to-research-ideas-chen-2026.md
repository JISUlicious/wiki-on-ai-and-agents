---
title: "IdeaAnchor: Teaching LLMs to Turn Literature into Research Ideas"
type: source
created: 2026-10-29
updated: 2026-10-29
sources:
  - ideaanchor-literature-to-research-ideas-chen-2026.md
arxiv_id: "2610.08781"
authors:
  - Ziyu Chen
  - Yilun Zhao
  - Jiashuo Sun
  - Yiling Ma
  - Manasi Patwardhan
  - Arman Cohan
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# IdeaAnchor: Teaching LLMs to Turn Literature into Research Ideas

**Ziyu Chen, Yilun Zhao, Jiashuo Sun, Yiling Ma, Manasi Patwardhan, Arman Cohan** (Yale University; University of Chicago; [[uiuc|UIUC]]; Tata Consultancy Services). [arXiv:2610.08781](https://arxiv.org/abs/2610.08781), v1, cs.CL, 6 Oct 2026. Preprint; no venue.

## Summary

The task: given 5–7 related papers (title plus summary), produce a thinking trace, a research motivation (the gap) and a research plan. An **IdeaAnchor** is a structured specification mined backwards from a published paper. It gives:

- **the role each input paper played**: direct predecessor, inspiration source, gap and motivation, or methodological ingredient;
- **how the papers relate** to each other and to the result;
- **checkable criteria** for the motivation, the method and the overall idea.

The anchor is *privileged information*: available in training, absent at deployment. It is used three ways: to steer a teacher (GPT-5.4-mini) for SFT demonstrations; for self-distillation; and as the reward rubric for [[grpo|GRPO]]. Retrieval of full-text passages is added at inference.

## Key points

- **Corpus.** 7,494 ICLR/ICML/NeurIPS papers (2023–25) and 6,689 Nature Communications papers. **Author check:** authors of 20 papers replied, and 16 were satisfied. The mined sets cover 93.7% of the works authors credit, and only 3 of 134 mined works (2.2%) were flagged as unimportant.
- **Benchmark.** 924 ICLR 2026 papers (all 224 orals included), temporally held out. Experts refined the criteria. The metric is the criteria satisfaction rate (CSR), judged by GPT-5.4.
- **Results, Qwen3-8B:** base 10.9% → self-distillation 16.3% → SFT 21.7% → RL 24.6%.
- **Results, Qwen3.5-9B:** 31.1% → SFT 48.7% → RL 54.2%, above the GPT-5.5 (51.0%) and GPT-5.4 (50.5%) references.
- All three paradigms win >75% of pairwise comparisons against the base (LLM judge and human experts).
- **Retrieval mostly adds detail.** RL with full-text retrieval reaches only 25.6% CSR (vs 24.6%), but is strongly preferred pairwise. Manual inspection of 100 pairs says retrieval adds methodological specifics without changing which gap is targeted. The authors' reading: gap-finding has to be trained in; detail can be retrieved.
- Training only on natural-science anchors still raises ML-benchmark CSR.

## What it does not show

- **Matching, not discovering.** The criteria come from the actual published paper, so CSR rewards reconstructing the idea that was published. It does not reward a novel or better idea.
- **Judge and reward overlap.** The RL reward is a GPT-5.4-mini judge scored against anchor criteria; the evaluation is a GPT-5.4 judge on the same style of criteria. "RL surpassing GPT-5.5" is on that rubric only.
- No ideas were executed. The from-scratch study used six topics and three raters.

## Concepts & entities

- [[priced-guidance-future-research-ideas-wen-2026]]: a companion question, scored by information cost instead of rubric match.
- [[ai-research-preference-models-foster-2026]] · [[scientisttwo-autonomous-ai-human-knowledge-frontier-nam-2026]]: other 2026 work on AI research ideation.
- [[grpo]] · [[knowledge-distillation]] · [[supervised-fine-tuning]] · [[llm-as-a-judge]] · [[retrieval-augmented-generation]]
- [[qwen]] · [[uiuc]]

## References

- [arXiv:2610.08781](https://arxiv.org/abs/2610.08781)
