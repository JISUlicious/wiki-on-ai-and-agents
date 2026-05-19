---
title: SWE-bench-Live
type: concept
created: 2026-05-17
updated: 2026-05-17
sources:
  - swe-bench-live-zhang-2025.md
status: complete
importance: high
tags:
  - 2025
---

**SWE-bench-Live** is a monthly-refreshed, contamination-resistant successor to [[swe-bench]] for evaluating LLM-based coding agents on real-world GitHub issue resolution. Introduced by Zhang et al. ([[microsoft]], 2025), the initial release contains **1,319 task instances across 93 Python repositories**, restricted to issues filed between January 2024 and April 2025 to stay outside known LLM pretraining cutoffs, and packaged with per-instance Docker images for reproducible end-to-end evaluation. The work's core engineering contribution is **RepoLaunch**, an agentic pipeline that auto-Dockerizes arbitrary repositories without the 200+ hours of manual environment setup that bottlenecked SWE-bench, SWE-Gym, and Multi-SWE-bench. The benchmark is designed to be regenerated on a monthly cadence with minimal human effort.

## Core mechanism

- **Monthly refresh of post-2024 GitHub issues**: 1,319 tasks drawn from issues created since January 2024, distributed roughly uniformly across months from 2024-01 through 2025-04. A Lite subset of 300 instances (50/month × 6 months) is provided for cheaper iteration.
- **RepoLaunch auto-Dockerization**: five-stage [[react-prompting]]-style agentic workflow — relevant-file identification → base-image selection → interactive setup loop → verification → finalization — that produces an instance-level Docker image per snapshot. Includes a "time-machine" pip proxy that pins package resolution to versions released at-or-before the commit's timestamp, eliminating dependency-version drift.
- **Validation via FAIL_TO_PASS / PASS_TO_PASS transitions** (inherited from SWE-bench): every accepted task must have at least one F2P transition and consistent results across repeated runs to filter flaky tests.
- **Headline contamination evidence**: running the same OpenHands + Claude 3.7 Sonnet configuration under identical settings yields **43.20% on SWE-bench Verified vs. 19.25% on SWE-bench-Live** — a 2.2× gap on otherwise-matched conditions. Stratifying further: instances from repos that overlap with the original SWE-bench score 22.96%, vs. 18.89% on novel repos that are structurally simpler — consistent with implicit overfitting to the SWE-bench corpus.
- **Broader repository diversity**: 93 repos spanning AI/ML, DevOps, Web, Scientific, Database, CLI, Cloud, Desktop, and miscellaneous; average repo is 85k LoC across 423 files. Resolved rate is roughly flat across 2024Q1–2025Q1 issue quarters, suggesting the signal is stable as the benchmark refreshes.

## What it advances

SWE-bench-Live establishes the monthly-refresh template as the practical answer to [[benchmark-contamination]] for long-horizon coding-agent evaluation, succeeding SWE-bench Verified as the de-facto leaderboard for measuring genuine progress (vs. memorization or implicit overfitting). The RepoLaunch pipeline makes the cost of refresh tractable — what previously required hundreds of hours of manual environment engineering per benchmark release now runs automatically — which is the structural precondition for any refresh-based benchmark to actually keep refreshing.

## Relation to other concepts

Direct successor to [[swe-bench]] and SWE-bench Verified; complement to [[swe-gym]] (training data for SWE-agents) and Multi-SWE-bench (multilingual variant). Central motivation is [[benchmark-contamination]]: the concern that static benchmarks released years ago have leaked into LLM pretraining corpora. Targets evaluation of [[llm-agent]] systems (OpenHands, SWE-agent, Agentless were the three top-line agents evaluated). Compare to LiveCodeBench, which streams algorithmic problems rather than repository-level issues; SWE-bench-Live is to long-horizon coding agents what LiveCodeBench is to competitive programming.

## References

- [[swe-bench-live-zhang-2025]] — primary source ([arXiv:2505.23419](https://arxiv.org/abs/2505.23419)).
- Homepage: <https://swe-bench-live.github.io/>
- Code: <https://github.com/SWE-bench-Live>
- Dataset: <https://huggingface.co/SWE-bench-Live>
- Predecessor: SWE-bench (Jimenez et al. 2024); SWE-bench Verified (OpenAI 2024).
