---
title: "Training Software Engineering Agents and Verifiers with SWE-Gym"
type: source
created: 2026-05-17
updated: 2026-05-17
sources:
  - swe-gym-pan-2024.md
arxiv_id: "2412.21139"
venue: ICML 2025
authors:
  - Jiayi Pan
  - Xingyao Wang
  - "[[graham-neubig]]"
  - Navdeep Jaitly
  - Heng Ji
  - Alane Suhr
  - Yizhe Zhang
first_author: Jiayi Pan
year: 2024
introduces:
  - "[[swe-gym]]"
tags:
  - 2024
status: complete
importance: high
---

# Training Software Engineering Agents and Verifiers with SWE-Gym

[arXiv:2412.21139](https://arxiv.org/abs/2412.21139) — Jiayi Pan, Xingyao Wang, [[graham-neubig]] et al. (UC Berkeley, UIUC, CMU, Apple). Accepted to ICML 2025.

## Summary

Pan et al. introduce **[[swe-gym]]**, the first publicly available training environment for software-engineering agents. It packages **2,438 real-world Python task instances** sourced from GitHub issues across **11 popular repositories** (pandas, MONAI, moto, mypy, dvc, dask, modin, pydantic, conan, hydra, bokeh) — repos held disjoint from SWE-Bench to avoid contamination. Each task ships with a natural-language issue description, a snapshot of the repository, a pre-built Docker runtime with pinned dependencies, and an expert-validated unit-test suite that doubles as a programmatic reward signal. Construction cost ~200 author annotation hours and ~10,000 CPU core hours; the released Docker image set totals 6 TB. A 230-task **SWE-Gym Lite** subset mirrors SWE-Bench Lite for fast iteration, and a 64,689-instance **SWE-Gym Raw** dump (without environments, spanning 358 repos) is released for synthetic-task work.

Using SWE-Gym the authors run two complementary scaling axes. **Training-time scaling**: fine-tune Qwen-2.5-Coder-32B on just 491 successful rollouts (collected by rejection sampling from GPT-4o and Claude 3.5 Sonnet under the OpenHands CodeAct scaffold), yielding absolute gains of +12.3% on SWE-Bench Lite (3.0 → 15.3) and +13.6% on SWE-Bench Verified (7.0 → 20.6); the curve is still rising at 491 trajectories, suggesting compute, not task count, is the bottleneck. **Inference-time scaling**: train an outcome-supervised **[[verifier]]** on SWE-Gym trajectories (success/failure labels from the unit tests), then sample multiple agent rollouts per test task and pick the verifier-best — log-linear gains push SWE-Bench Verified to **32.0%** and Lite to **26.0%**, a new **open-weight state-of-the-art at release**. SWE-Gym is the first **scalable training environment** for SWE agents (vs. eval-only SWE-Bench, synthetic R2E, or imitation-only SWE-Bench train), and serves as the blueprint that R2E-Gym and SWE-smith later extend.

## Key Points

- **2,438 executable Python tasks across 11 repos** — every instance has a Docker environment with installed dependencies + expert-curated `fail-to-pass` unit tests (avg 10 fail-to-pass tests out of 760 total per task). Disjoint from SWE-Bench repos.
- **Task difficulty exceeds SWE-Bench**: gold patches average 69.8 lines / 2.5 files / 4.1 functions edited (vs. 32.8 / 1.7 / 3.0 on SWE-Bench), partly because sophisticated repos like pandas and MONAI are included.
- **Two ingredients for agent improvement**: (1) **agent SFT** on rejection-sampled successful trajectories, (2) **verifier-based test-time best-of-N selection** ([[outcome-based-reward]] model trained on trajectory success labels).
- **Reproducible environments**: 6 TB of pre-built Docker images released so external researchers can collect their own trajectories. Versions for each instance auto-resolved from `pyproject.toml` / git tags / CI scripts.
- **Effective across scaffolds**: works with general-purpose OpenHands CodeAct *and* specialized-workflow MoatlessTools (where self-improvement via rejection-sampling FT hits 19.7% on SWE-Bench Lite).
- **Fixes stuck-in-loop pathology**: zero-shot Qwen-2.5-Coder-32B loops on 29.4% of SWE-Bench Verified tasks; SWE-Gym fine-tuning cuts this by 4.6–18.6 pp across model sizes.
- **Scaling laws**: both training-time (491 trajectories, no saturation) and inference-time (1 → 16 rollouts, log-linear) curves climb monotonically — performance is compute-limited, not data-limited.
- **Open-weight SOTA at release**: 32.0% Verified / 26.0% Lite, beating prior open-weight systems and approaching proprietary scaffolds built on GPT-4 / Claude at that time.
- **Released artifacts**: SWE-Gym (envs + tests), SWE-Gym Lite (230), SWE-Gym Raw (64,689), fine-tuned agent + verifier weights, and 491 expert trajectories.

## Entities

- [[uc-berkeley]] — Pan and Suhr's affiliation; lead institution.
- [[cmu]] — [[graham-neubig]]'s affiliation.
- [[graham-neubig]] — co-author; long-standing CMU NLP / agents researcher.
- UIUC (Xingyao Wang, Heng Ji) and Apple (Navdeep Jaitly, Yizhe Zhang) also contributed.

## Concepts

- [[swe-gym]] — the environment introduced by this paper.
- [[swe-bench]] — primary evaluation benchmark (Verified + Lite); SWE-Gym's repo set is held disjoint from it.
- [[llm-agent]] — SWE agents fit the broader LM-agent paradigm: tool use, multi-turn planning, environment feedback.
- [[agentic-rl]] — the paper is a canonical example of moving SWE agents from prompting-only into a training-loop regime with executable reward.
- [[verifier]] / [[outcome-based-reward]] — outcome-supervised reward model trained on trajectory pass/fail; powers test-time best-of-N.
- Related but not page-worthy here: [[rejection-sampling-fine-tuning]], [[inference-time-scaling]], OpenHands CodeAct scaffold, MoatlessTools scaffold.

## Quotes

> "We present SWE-Gym, the first environment for training software engineering (SWE) agents. SWE-Gym contains 2,438 real-world task instances, each comprising a Python codebase with an executable runtime environment, unit tests, and a task specified in natural language."

> "When combined with our fine-tuned SWE agents, we achieve 32.0% and 26.0% on SWE-Bench Verified and Lite, respectively, reflecting a new state-of-the-art for open-weight SWE agents."

> "Performance is currently limited by the compute budget for sampling rather than the number of tasks in SWE-Gym."

> "Fine-tuning on trajectories from SWE-Gym consistently reduces the stuck-in-loop rate by 4.6–18.6% across both SWE-Bench Lite and Verified tasks."

## References

- Jimenez et al. 2024 — **SWE-Bench**: the eval benchmark SWE-Gym targets and extends.
- Wang et al. 2024c — **OpenHands** CodeAct agent scaffold.
- Örwall 2024 — **MoatlessTools** specialized-workflow scaffold.
- Jain et al. 2024 — **R2E** (synthetic SWE eval, 246 instances) — contrasted as not realistic enough.
- Cobbe et al. 2021 — outcome-supervised verifier formulation (originally for GSM-8K) adopted here.
- Ouyang et al. 2022 — RLHF as the post-training precedent SWE-Gym extends from chatbots into agents.
- Hui et al. 2024 — **Qwen-2.5-Coder** base model used for fine-tuning.
- Hendrycks et al. 2021 (APPS, MATH), Chen et al. 2021 (HumanEval), Zheng et al. 2024 (CodeFeedback) — code/reasoning training-data baselines compared in Tab. 1.
