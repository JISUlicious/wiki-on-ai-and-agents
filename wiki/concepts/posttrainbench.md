---
title: PostTrainBench
type: concept
created: 2026-11-10
updated: 2026-11-10
sources:
  - what-is-missing-from-ai-post-training-ai-lim-2026.md
  - training-llm-agents-to-automate-post-training-rsi-dojo-gao-2026.md
  - ai4ai-bench-algorithmic-design-chi-2026.md
status: draft
importance: medium
---

# PostTrainBench

A benchmark for "AI post-training AI": an agent such as [[claude-code]] is given a base model, a target benchmark, one H100 and a 10-hour budget, and must post-train the model end to end (write the pipeline, train, evaluate checkpoints, submit the best one). Introduced by Rank et al. (2026, arXiv:2603.08640). Promoted to a page on 2026-11-10 after three sources cited it.

## Setup (from the released trajectories)

- Target benchmarks: AIME 2025, ArenaHardWriting, BFCL, [[gpqa|GPQA]] Main, GSM8K, HealthBench, [[humaneval|HumanEval]].
- Base models: [[gemma|Gemma]]-3-4B-PT, Qwen3-1.7B-Base, Qwen3-4B-Base, SmolLM3-3B-Base.
- A Lite variant is used in the GPT-5.6 System Card setting.

## What sources report

- [[what-is-missing-from-ai-post-training-ai-lim-2026]] analyses 1,338 public trajectories (5,111 verified training runs) and separates execution from strategy failures.
- [[training-llm-agents-to-automate-post-training-rsi-dojo-gao-2026]] trains the skill into weights: RSI-Dojo-35B-A3B goes from 13.08 to 29.95 on v1.1 (GLM-5.2 29.52, GPT-5.5 27.80), and from 5.80 to 53.29 on Lite. Its setup differs from the official one (weaker GPUs, pre-downloaded data), so the numbers are not strictly comparable.
- [[ai4ai-bench-algorithmic-design-chi-2026]] positions itself against PostTrainBench among AI-for-AI benchmarks.

## Related

[[recursive-self-improvement]] · [[agent-benchmark]] · [[benchmark]]
