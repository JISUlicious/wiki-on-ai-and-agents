---
title: "From Foundation to Application: Improving VLA Models in Practice"
type: source
created: 2026-07-22
updated: 2026-07-22
sources:
  - from-foundation-to-application-vla-wu-2026.md
arxiv_id: "2607.06403"
year: 2026
authors:
  - Wei Wu
  - Fangjing Wang
  - Fan Lu
  - Yujun Shen
  - Kecheng Zheng
first_author: Wei Wu
venue: arXiv (cs.RO) preprint
tags:
  - 2026
  - paper
status: complete
importance: low
---

# From Foundation to Application: Improving VLA Models in Practice

**Wei Wu, Fangjing Wang, Fan Lu, … Yujun Shen, Kecheng Zheng** (Robbyant) — [arXiv:2607.06403](https://arxiv.org/abs/2607.06403) (v1, 7 Jul 2026, cs.RO). Preprint; no venue listed.

## Summary

Released as **LingBot-VLA 2.0**, this is a vision-language-action model aimed squarely at closing the lab-to-deployment gap rather than at a new architecture. Three changes: a revamped data pipeline for cross-embodiment generalization, an action space extended past the standard dual-arm setup, and predictive dynamics modeling as an auxiliary objective. The last of these formulates future prediction as a proxy task, drawing semantic priors from a video representation model and geometric cues from a depth estimation model.

Peripheral to this wiki's LLM/agent focus, but notable as an open-weights embodied model release with a large disclosed data budget.

## Key points

- **~60,000 hours of pretraining data**: 50,000 hours of robot trajectories spanning **20 robot configurations**, plus 10,000 hours of egocentric human video.
- **Whole-body action space** — degrees of freedom for heads, waists, mobile bases, and dexterous hands, not just dual arms.
- **GM-100 benchmark, generalist (single policy jointly trained on all nine tasks)**: on Agilex Cobot Magic, 66.2 progress / 34.4 success, beating LingBot-VLA-1.0 by 8.0 / 4.4 and π0.5 by 7.1 / 2.2. On Galaxea R1 Pro, 34.6 / 15.6, beating π0.5 by 7.2 / 6.7. GR00T N1.7 is the weakest baseline (36.3 / 17.8 on Cobot Magic).
- **Long-horizon mobile manipulation**, 15 trials per task-setting pair: in-domain 77.1 / 60.0 on refrigerator sorting (Astribot S1) and 84.3 / 66.7 on stove cleaning (Cobot Magic-ARX X5). Out-of-distribution (±10 cm initial-pose perturbation, unseen object categories) drops to 37.0 / 13.3 and 67.5 / 40.0 — still ahead of π0.5 by roughly 5-7 points on both axes.
- Uses a **sparse MoE** action expert; the paper reports it gives more efficient scaling with reduced training loss and validation error.
- Open release: code, website, and HuggingFace checkpoint collection.

> [!warning] Newsletter discrepancy
> A newsletter summary attributed to this paper an inference latency of **~130 ms on a single RTX 4090D**. **The paper contains no such claim** — the strings "4090", "RTX", and any per-step latency figure do not appear anywhere in the text. The 60,000-hour / 50k+10k / 20-configuration data figures *are* accurate. Do not propagate the latency number.

## Concepts & entities

- [[mixture-of-experts]] — sparse MoE is used to scale action-modeling capacity, with a load-balancing bias update.
- [[world-model]] — the predictive dynamics proxy task is a world-model-flavored auxiliary objective rather than a full generative world model.
- [[magma-yang-2025]] — earlier multimodal foundation-model-for-agents work in the same VLA lineage.
- [[code-as-policies-liang-2022]] — contrasting earlier approach to robot control, generating policy code from language instead of learning end-to-end action outputs.

## References

- [arXiv:2607.06403](https://arxiv.org/abs/2607.06403)
- Original source text: `sources/from-foundation-to-application-vla-wu-2026.md`
