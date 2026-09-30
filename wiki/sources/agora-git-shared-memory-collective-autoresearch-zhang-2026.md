---
title: "Agora: Git as Shared Memory for Collective AutoResearch"
type: source
created: 2026-09-30
updated: 2026-09-30
sources:
  - agora-git-shared-memory-collective-autoresearch-zhang-2026.md
arxiv_id: "2609.18094"
authors:
  - Yifan Zhang
  - Yunheng Zou
  - Shaokun Zhang
  - Jian Hu
  - Hao Zhang
  - Binfeng Xu
  - Jan Kautz
  - Yi Dong
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# Agora: Git as Shared Memory for Collective AutoResearch

**Yifan Zhang, Yunheng Zou, Shaokun Zhang, Jian Hu, Hao Zhang, Binfeng Xu, Jan Kautz, Yi Dong** ([[nvidia|NVIDIA]]) — [arXiv:2609.18094](https://arxiv.org/abs/2609.18094), v3, cs.LG, 22 Sep 2026. Preprint; no venue.

## Summary

[[andrej-karpathy|Karpathy]]-style autoresearch runs one agent in a loop. Agora is infrastructure for running **many independently scheduled agents with no planner, roles or shared conversation**. Each contribution is a Git commit (result, insight, hypothesis, verification or report), and each parent edge means "builds on". The resulting append-only DAG is the shared memory. `agora analyze` rebuilds views from the DAG: leaders, unverified claims, open hypotheses, neglected leaves. An evidence score counts **only follow-on work by *other* accounts**: a verification is worth +20 or +10 when it succeeds and −20 when it fails, and self-citation is excluded. A diversity-aware UCB over description embeddings (Arctic-Embed 2.0; clusters at cosine 0.90) presents candidates in three slots: exploit, explore-known and explore-novel.

The paper is a **system description plus a case study**, not a controlled evaluation. The authors say so in the abstract: "Measuring the effect on discovery per unit of compute requires a matched comparison."

## Key points

- **Task.** Initialize a frozen 119.6M attention–SSM hybrid (14 layers, d = 672, sized so that no donor matches) from **141 donor models** in 32 families, with no training data and no gradient updates. Scoring is bits per byte on 200 FineWeb-Edu texts. Random initialization gives 3.3923 bpb; a trained GPT-2 124M gives about 1.0.
- **Run.** 13 Claude Code (Opus 4.7) and Codex (GPT-5.5) workers, one 80 GB GPU each, ran **11 days 19 h** and produced **1,703 contributions**: 1,124 scored results (233 of them new bests), 284 insights, 203 hypotheses, 165 verifications and 53 explicit negative results.
- **Result: 1.899 bpb, closing 62% of the gap** to trained GPT-2. The winning recipe compresses six GPT-2-vocabulary donors' next-token predictions under 28 contexts into a 50,257² bigram table. That table is factorized by randomized SVD into the embedding and output head, and the model then re-enables sparse, band-limited attention mean-pools, SSM convolutions and one feed-forward slice.
- **Almost all of the gain came first, and fast.** The first 18 scored contributions (within ~6 hours on day 1) delivered **~98% of the total reduction**. The remaining 1,106 found the last ~0.03 bpb. The final recorded improvement (9×10⁻⁶) is below cross-hardware variation (up to 1.3×10⁻³).
- **Collaboration is real but narrow.** The winner's 145-commit ancestry spans 15 accounts, and 115 of its 144 parent edges cross accounts. 165 verifications cover 95 targets and report **zero failures**. But one "narrow spine" collected most follow-on work, and there was heavy parallel rediscovery: of 696 equal-score pairs from different accounts, 63% fell within an hour of each other.
- **The one human intervention mattered.** On May 2 more than a third of activity sat in a single semantic cluster, and humans deployed the diversity views. Within a day a worker made the first SSM edit (1.9028), and the new frontier followed.

## What it does not show

- **No matched-compute baseline of any kind.** There is no single-agent autoresearch run at the same GPU-days, no independent-workers-without-Agora arm, and no plain search. The authors flag this explicitly.
- **No held-out evaluation.** Every step was selected on the same 200-text development evaluator (Table 4 note: "stages of a search, not a controlled ablation"). The authors did not rerun the winning method. The "hard local optimum" diagnosis was proposed by the agents and never tested.

## Concepts & entities

- [[multi-agent]] · [[multi-agent-memory]] — asynchronous agents coordinating only through a persistent artifact store. Compare [[diversity-collapse-in-multi-agent-llm-systems-chen-2026]]: shared visibility alone did not sustain exploration.
- [[quality-diversity]] — the diversity UCB and the explore-novel slot. Effective cluster count is the exponential of the entropy over cluster shares.
- [[self-improving-agent]] — collective, not recursive. The object improved is an external artifact, not the agents.
- [[agentic-ai-next-intelligence-explosion-evans-2026]] — cited as the institutional framing for agent research communities.
- [[claude-code]] · [[andrej-karpathy]] — worker harness; autoresearch lineage.
- [[state-space-model]] — the target architecture's SSM blocks.

## References

- [arXiv:2609.18094](https://arxiv.org/abs/2609.18094)
