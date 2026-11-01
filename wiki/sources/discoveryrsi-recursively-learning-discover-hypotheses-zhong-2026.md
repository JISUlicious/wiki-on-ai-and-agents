---
title: "DiscoveryRSI: Recursively Learning to Discover Valuable Hypotheses"
type: source
created: 2026-10-29
updated: 2026-10-29
sources:
  - discoveryrsi-recursively-learning-discover-hypotheses-zhong-2026.md
arxiv_id: "2610.23839"
authors:
  - Jianyuan Zhong
  - Chenwei Wu
  - Zhaochen Yu
  - Xixian Jiang
  - Jiajun Fan
  - Leo Anthony Celi
  - Zhenfei Yin
  - Yingcheng Wu
  - Ling Yang
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# DiscoveryRSI: Recursively Learning to Discover Valuable Hypotheses

**Jianyuan Zhong, Chenwei Wu, Zhaochen Yu, et al.; corresponding authors Leo Anthony Celi, Zhenfei Yin, Yingcheng Wu, Ling Yang** (PhAI Labs, [[chinese-university-of-hong-kong|CUHK]], U. Michigan, NUS, [[fudan-university|Fudan]], [[peking-university|PKU]], [[stanford-university|Stanford]], HKUST, [[uc-santa-barbara|UCSB]], Harvard Medical School, [[uiuc|UIUC]], [[mit|MIT]], Oxford, [[princeton-university|Princeton]], and others; the author list above is abridged). [arXiv:2610.23839](https://arxiv.org/abs/2610.23839), v2, cs.CL, 20 Oct 2026.

## Summary

DiscoveryRSI defines a *valuable hypothesis* operationally: an evidence-grounded claim about a consequential uncertainty, together with a feasible investigation, decision criteria and alternative explanations. A **Lead / Generation / Evolution / Critique** multi-agent search grows **hypothesis trees** using evolve, repair, crossover and restart operators. Selected investigations are compiled into executable **Harbor tasks**. The system then improves itself recursively in three ways:
1. **Hypothesis credit optimization.** TrueSkill-style assessments are attributed back to the contributing agent actions, giving credit-weighted SFT.
2. **Harness replay.** With the model frozen, recurring failures (lost artifacts, premature closure, repeated exploration) are diagnosed and the workflow is edited.
3. **Rediscovery.** Accepted updates are applied to fresh questions.

The paper also introduces **DFM-Bench**, which scores hypotheses, revisions, development paths and runs.

## Key points

- **DiscoveryRSI-27B** is a full fine-tune of **Qwen3.8-27B** (256K context). It uses 2,397 of 15,792 trajectories (291M tokens, of which only 2.12% are directly supervised) for one epoch.
- **Automated quality** (GPT-5.6 Sol as judge; 241 questions in four life-science domains: therapeutic targets, population multi-omics, disease mechanisms, clinical & population health — datasets incl. HCC cohort, MIMIC-IV, eICU, UK Biobank, GEO) rises **4.86 → 5.18 → 5.39** over two RSI rounds. The second round adds less (+0.21 vs +0.32). Gains concentrate on falsifiability and data resolvability. Evolve/repair use rises from 77.5% to 91.7%.
- **Human review** covers 292 proposals by 27 biomedical raters, blinded. Mean change per revision is **+0.07** (DiscoveryRSI) vs +0.03 (GPT-5.6 Sol) and −0.01 (GPT-6 Luna). Absolute ratings are **mixed**: DiscoveryRSI wins on importance (3.10 vs 2.94) and evidence-bounded claims, ties on advance over known work (2.31), and loses on data fit, alternatives and informativeness.
- **"Inspiration" rate.** This is the share of hypothesis versions that a post-cutoff publication links to by supplying a design, comparator, measurement or evidence gap. DiscoveryRSI scores **6.3% (74/1,176)**, against DeepSeek 5.4% (14/257), GLM 2.8% (1/36), GPT 2.5% (59/2,341) and Claude 2.3% (3/131).

> [!warning] Newsletter vs paper
> - **"Matches post-cutoff research" overstates the measure.** The 6.3% counts hypotheses for which a *later paper supplies a usable design element or evidence gap*. It does not mean the hypotheses anticipated published findings.
> - **The comparison is very unbalanced.** The Claude figure rests on **131 hypotheses with 3 hits**, and GLM's on 36 with 1. Each model has a different temporal cutoff. Section 7.4 labels the rows by family ("GPT", "Claude"), while the abstract names "GPT-6" and "Claude Opus 5".
> - The 4.86 → 5.39 gain is **LLM-judged** (GPT-5.6 Sol), not human-rated.

## What it does not show

- **No wet-lab or executed validation** of the hypotheses. Harbor tasks measure execution readiness, not outcomes.
- **Same-family judge.** The automated judge (GPT-5.6 Sol) is from the same model family as one of the baselines. The human study is described as "preliminary".
- No ablation separates credit-weighted SFT from harness replay in the round-over-round gain.

## Concepts & entities

- [[self-improving-agent]] · [[recursive-agent-harnessing]]: model-side and harness-side updates in one loop. Compare [[meta-harness-lee-2026]] and [[harness-continual-learning-kang-2026]].
- [[multi-agent]] · [[quality-diversity]]: evolve, repair, crossover and restart operators over hypothesis trees.
- [[llm-as-a-judge]] · [[qwen]]
- Automated-science neighbours: [[agentic-scientist-orchestrating-coding-agents-lange-2026]], [[self-revising-discovery-systems-wang-2026]], [[scientisttwo-autonomous-ai-human-knowledge-frontier-nam-2026]]
- [[recursive-self-improvement]]: concept page for RSI definitions, autonomy levels and recurring caveats across papers.

## References

- [arXiv:2610.23839](https://arxiv.org/abs/2610.23839)
