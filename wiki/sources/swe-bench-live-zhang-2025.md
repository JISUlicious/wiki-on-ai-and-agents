---
title: "SWE-bench Goes Live!"
type: source
created: 2026-05-17
updated: 2026-05-17
sources:
  - swe-bench-live-zhang-2025.md
arxiv_id: "2505.23419"
authors:
  - Linghao Zhang
  - Shilin He
  - Chaoyun Zhang
  - Yu Kang
  - Bowen Li
  - Chengxing Xie
  - Junhao Wang
  - Maoquan Wang
  - Yufan Huang
  - Shengyu Fu
  - Elsie Nallipogu
  - Qingwei Lin
  - Yingnong Dang
  - Saravan Rajmohan
  - Dongmei Zhang
first_author: Linghao Zhang
year: 2025
introduces:
  - "[[swe-bench-live]]"
tags:
  - 2025
status: complete
importance: high
---

# SWE-bench Goes Live!

**Source file**: [[swe-bench-live-zhang-2025]]
**arXiv**: [arXiv:2505.23419](https://arxiv.org/abs/2505.23419)
**Authors**: Linghao Zhang, Shilin He, Chaoyun Zhang, Yu Kang, Bowen Li, et al. (Microsoft)
**Year**: 2025 (May)

## Summary

SWE-bench-Live is a **live, monthly-refreshed successor to SWE-bench** for evaluating LLM-based coding agents on real-world GitHub issue resolution. The initial release contains **1,319 task instances** drawn from real issues filed between January 2024 and April 2025 across **93 Python repositories**, each packaged with a dedicated Docker image so that evaluation is reproducible end-to-end. The core technical contribution is **RepoLaunch**, an agentic pipeline that takes a repository snapshot and autonomously identifies relevant config files, picks a base image, installs dependencies (with a "time-machine" pip proxy that pins versions to the commit timestamp to avoid dependency drift), and validates the test suite — eliminating the months of manual environment setup that bottlenecked SWE-bench, SWE-Gym, and Multi-SWE-bench. The benchmark is designed to be updated automatically on a monthly cadence.

The motivation is explicitly **contamination resistance**: because static benchmarks like SWE-bench (released 2023) have not been refreshed, their tasks are at risk of leaking into pretraining corpora. The paper empirically supports this concern. Running the same OpenHands + Claude 3.7 Sonnet configuration on both benchmarks under identical settings, resolved rate drops from **43.20% on SWE-bench Verified to 19.25% on SWE-bench-Live**. Stratifying SWE-bench-Live by repository origin further shows that instances from repos that overlap with the original SWE-bench score notably higher (22.96%) than instances from previously-unused repos (18.89%) — consistent with implicit overfitting to the SWE-bench corpus. Together with the broader 93-repo coverage and automated curation, the work positions SWE-bench-Live as the de-facto contamination-resistant evaluation for long-horizon coding agents.

## Key Points

- **Live, monthly-refreshed benchmark**: First open, continuously-updated benchmark targeting repository-level issue resolution (vs. LiveCodeBench, which streams algorithmic problems).
- **1,319 tasks across 93 Python repositories**, restricted to issues created since January 2024 to limit pretraining contamination. The distribution across months from 2024-01 through 2025-04 is roughly uniform.
- **RepoLaunch pipeline**: Five-stage agentic workflow (relevant-file ID → base-image selection → interactive ReAct-style setup → verification → finalization) that auto-produces an instance-level Docker image per snapshot. Replaces the ~200+ hours of manual environment setup reported by SWE-Gym.
- **"Time-machine" pip proxy**: Forces dependency resolution to only consider package versions released at-or-before the commit's timestamp, addressing the "dependency version drift" failure mode where modern pip resolves to incompatible future versions.
- **Validation via FAIL_TO_PASS / PASS_TO_PASS transitions** (inherited from SWE-bench): each task must have at least one F2P transition and consistent results across repeated runs to filter flaky tests.
- **Lite subset**: 300 instances (50/month × 6 months from 2024-10 through 2025-03) for cheaper iteration.
- **Repository diversity**: AI/ML (26), DevOps (23), Web (17), Scientific (8), Database (8), CLI (4), Misc (3), Cloud (2), Desktop (2). Average repo: 85k LoC, 423 files. Average gold patch: 3.3 files, 9 hunks, 102.6 lines.
- **Headline result — scores drop sharply vs. SWE-bench**: Best agent (OpenHands + Claude 3.7 Sonnet) reaches only **19.25% resolved on the full SWE-bench-Live** vs. ~43% on SWE-bench Verified with the same setup (and >60% on the public leaderboard with heavier rollouts). SWE-agent + GPT-4.1 (18.57%) and SWE-agent + Claude 3.7 Sonnet (17.13%) round out the top three.
- **Evidence of overfitting to SWE-bench repos**: Resolved rate is 22.96% on the 216 instances from repos that appear in original SWE-bench vs. 18.89% on the 1,103 instances from novel repos — despite the novel-repo set being structurally simpler.
- **Recency does not predict difficulty**: Resolved rate is roughly flat across 2024Q1–2025Q1 issue creation quarters, suggesting the benchmark provides a stable signal as it refreshes.
- **Patch-difficulty heatmap**: Single-file patches under 5 lines are solved ~48% of the time; success falls below 10% once a patch touches 3+ files or >100 lines.
- **Three agents evaluated**: [[openhands]] (with CodeAct, 60-iter cap), [[swe-agent]] (100 LLM-call cap), [[agentless]] (localize-then-repair, single sample without reranking).
- **Four models evaluated**: GPT-4o (`gpt-4o-2024-11-20`), GPT-4.1 (`gpt-4.1-2025-04-14`), Claude 3.7 Sonnet (`claude-3-7-sonnet-20250219`), DeepSeek V3 (`DeepSeek-V3-0324`).

## Entities Mentioned

- [[microsoft]] — affiliation of all listed authors.
- [[microsoft-research]] — the research arm producing the benchmark.

## Concepts Discussed

- [[swe-bench-live]] — the benchmark introduced by this paper.
- [[swe-bench]] — the static predecessor that motivates the work.
- [[swe-gym]] — referenced as a manual-curation comparison (200+ hours of env setup).
- [[llm-agent]] — the system class under evaluation (OpenHands, SWE-agent, Agentless).
- [[benchmark-contamination]] — the central concern the design targets.
- [[docker]] — per-instance reproducible execution environments.
- [[react-prompting]] — RepoLaunch's setup agent follows a Thought→Action→Observation loop.
- Benchmarks compared: [[swe-bench-verified]], [[multi-swe-bench]], [[swe-smith]], [[livecodebench]].

## Notable Quotes

> "SWE-bench and its derivatives have not been updated since their initial releases, making them static benchmarks. Because LLMs are trained on massive inscrutable corpora, these static datasets are at risk of data contamination... This raises concerns about whether newer models are making truly generalizable progress or merely memorizing benchmark content." (§1)

> "The highest resolved rate on SWE-bench-Live is only 19.25%. ... We specifically re-ran the best performing combination, OpenHands with Claude 3.7 Sonnet, on the SWE-bench verified subset using the exact same setups as in our experiments. The resulting resolved rate reached 43.20%, more than twice the score achieved on SWE-bench-Live." (§4.2)

> "Although the Non-SWE-bench repositories are generally simpler with fewer files and lower code volume, the best-performing agent–model pair achieves a higher resolved rate of 22.96% on SWE-bench Instances, compared to only 18.89% on the Non-SWE-bench ones. This reinforces the hypothesis that existing agents may be overfit or implicitly optimized for the SWE-bench repositories." (§4.2)

> "SWE-bench-Live can be maintained with minimal–ideally zero–manual effort. We plan to update SWE-bench-Live on a monthly basis, continually providing the community with an up-to-date evaluation dataset." (§3)

## References

- arXiv: https://arxiv.org/abs/2505.23419
- Homepage: https://swe-bench-live.github.io/
- Code: https://github.com/SWE-bench-Live
- Dataset: https://huggingface.co/SWE-bench-Live
- Predecessor: SWE-bench (Jimenez et al., 2024) and SWE-bench Verified (OpenAI, 2024).
- Compared benchmarks: SWE-Gym, Multi-SWE-bench, SWE-smith, LiveCodeBench.
- Agents evaluated: OpenHands (Wang et al.), SWE-agent (Yang et al.), Agentless (Xia et al.).
