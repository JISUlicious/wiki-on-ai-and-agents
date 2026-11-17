---
title: "No Second Run: Does RSI Stop at Frontier Pre-training?"
type: source
created: 2026-11-17
updated: 2026-11-17
sources:
  - no-second-run-cai-2026.md
arxiv_id: "2611.10719"
venue: arXiv preprint (cs.AI)
authors:
  - Xiaojie Cai
  - Jiayi Chen
  - Junwei Chen
  - Pengfei Liu
  - Yu Qiao
  - Xipeng Qiu
  - et al.
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# No Second Run: Does RSI Stop at Frontier Pre-training?

**Source:** Xiaojie Cai, Jiayi Chen, Junwei Chen, … Haoyang Zou (59 authors, alphabetical) (Shanghai Innovation Institute / SII-GAIR; Xinxu AI) — [arXiv:2611.10719](https://arxiv.org/abs/2611.10719), cs.AI, v1 7 Nov 2026. Technical report for **801-30B-A3B-Base**; weights and SFT data are promised at GAIR-NLP/801-30B.

## Summary

Public [[recursive-self-improvement]] work loops propose → run → measure → update on things that are cheap to rerun and easy to score: harnesses, kernels, post-training. Frontier [[pre-training]] breaks all three conditions. A run costs hundreds of thousands of GPU-hours and cannot be repeated. Its real objective, headroom for post-training, cannot be read off any single score. And what must improve is a production system and the teams running it, not one model. The authors' answer is that RSI does not stop there, but it changes form. They split the unrepeatable run into smaller loops that can close:

- **Debugging loops** (infra and data) trace internal-metric or output anomalies to root causes.
- A **headroom loop** (the recipe loop) judges checkpoints by what they are worth after adaptation.
- **Experience-to-metric loops** (classifier development, site discovery, crawl coverage) turn data-team judgement into measurable workflows.

A shared context infrastructure ties them together. It acts as the organization's memory and keeps negative results and the reasons for them. The paper argues the bottleneck moves from compute to institutional memory. The authors present the model as the byproduct and this account as the contribution.

## Key points

- **The model.** 801-30B-A3B is a [[mixture-of-experts]] model on the Qwen3-30B-A3B architecture (30B total, 3B active, 48 MoE layers). It was trained from scratch on 11.8T tokens in three months by a new team, using Muon (AdamW for embeddings, head, norms and routers), three 4K-context sub-stages, then 32K and 256K extension. The data came from six pipelines with more than 30 stages, producing 1,053 subsets and 10.5T unique tokens.
- **Debugging pays off.** Five cross-stack diagnoses were resolved in about a month. Examples: expert over-balancing fixed by global-batch load balancing, Muon/AdamW module reassignment, and advertisement-like outputs traced to low-quality web data. At a matched 1T-token budget, the mean over 22 benchmarks rose from 47.78 (initial pilot) to 51.96 (revised pilot) to 56.83 (main run).
- **Base scores mislead.** Tiny-SFT (6.33B tokens) exposed "false stagnation": from 4.1T to 7T tokens the base average sat near 61 while post-SFT rose from 65.34 to 66.83. Base and post-SFT checkpoint rankings correlate only at Spearman 0.796 (Stage1-1) and 0.438 (later stages). Learning aptitude, the slope of score versus log SFT data, is added as a second headroom signal.
- **Agents produce training data.** 220K executable pull requests rolled out across nine harnesses gave more than 100B tokens of verified agentic trajectories (134B agentic tokens in all, about 91K tokens per trajectory), used in the long-context stage.
- **Base results.** The 13-benchmark average is 73.13: second of seven, behind Qwen3.5-35B-A3B (76.67) and just ahead of Nemotron-3.5-Lightning (73.07). It is the top score on GSM8K (92.80) and HumanEval (78.81).
- **After identical SFT** (6.33B tokens, no RL): AVG13 81.34, second to Qwen3.5 (83.76); AVG5 28.70 vs. 20.54 for Qwen3.5.
- **Agentic.** With only that light SFT, it averages 58.43 across SWE-bench Verified (68.95), SWE-bench Pro (48.29), CyberGym (38.90) and KernelBench (77.60). Fully post-trained baselines score lower, for example Flint-40B-A2.5B at 50.23 and Qwen3.6-35B-A3B at 44.59. The authors stress this is a headroom reference, not a like-for-like comparison.

## Caveats

- Only a base model and lightweight SFT are released now; there is no RL and no full post-training.
- The "loops" are largely human–agent organizational processes. There is no ablation showing how much each loop contributed versus team skill.
- SFT probes were not repeated across seeds, so small checkpoint differences are noisy.

## Concepts & entities

- [[recursive-self-improvement]] · [[self-improving-agent]] — RSI recast for an unrepeatable run.
- [[pre-training]] · [[mixture-of-experts]] · [[supervised-fine-tuning]] · [[scaling-laws]] — the training regime and why small proxies fall short.
- [[swe-bench-verified]] · [[swe-agent]] · [[gsm8k]] · [[humaneval]] — evaluation.
- [[qwen]] · [[andrej-karpathy]] (autoresearch, cited as a single-GPU RSI example).

## References

- [arXiv:2611.10719](https://arxiv.org/abs/2611.10719)
