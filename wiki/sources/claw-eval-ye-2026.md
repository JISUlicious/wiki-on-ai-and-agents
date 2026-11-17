---
title: "Claw-Eval: Towards Trustworthy Evaluation of Autonomous Agents"
type: source
created: 2026-11-17
updated: 2026-11-17
sources:
  - claw-eval-ye-2026.md
arxiv_id: "2604.06132"
venue: arXiv preprint (cs.AI)
authors:
  - Bowen Ye
  - Rang Li
  - Qibin Yang
  - Yuanxin Liu
  - Linli Yao
  - Hanglong Lv
  - Zhihui Xie
  - Chenxin An
  - Lei Li
  - Lingpeng Kong
  - Qi Liu
  - Zhifang Sui
  - Tong Yang
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# Claw-Eval: Towards Trustworthy Evaluation of Autonomous Agents

**Source:** Bowen Ye, Rang Li, Qibin Yang, Yuanxin Liu, Linli Yao, Hanglong Lv, Zhihui Xie, Chenxin An, Lei Li, Lingpeng Kong, Qi Liu, Zhifang Sui, Tong Yang ([[peking-university|Peking University]]; [[hku|The University of Hong Kong]]) — [arXiv:2604.06132](https://arxiv.org/abs/2604.06132), cs.AI, v1 7 Apr 2026. This page summarises v3 (7 May 2026). Leaderboard, data and code at claw-eval.github.io.

## Summary

Claw-Eval is an end-to-end agent benchmark built around one complaint: most benchmarks grade only the final output, so they cannot tell an agent that did the work from one that faked it or broke a rule on the way. It has **300 human-verified tasks** in three groups: General service orchestration (161), Multimodal perception and generation (101) and Multi-turn professional dialogue (38). Every run is recorded through three independent channels: the agent's execution trace, server-side audit logs, and environment snapshots after the run. These feed **2,159 rubric items** (7.2 per task). Deterministic checks handle objective conditions; an LLM judge handles open-ended ones.

Each attempt is scored as `safety × (α · completion + β · robustness)`. Safety is a multiplicative gate, so a policy violation cannot be bought back with high completion. Robustness is the share of tool types that recovered after an injected error. Each task runs three times, and the paper reports the average score, Pass@3 (any trial passes, threshold 0.75) and Pass^3 (all three pass).

## Key points

- **Output-only judging misses violations.** A vanilla [[llm-as-a-judge|LLM judge]] (Gemini 3 Flash) that sees the full transcript and the grader code, but not audit logs or snapshots, misses **12 of 27 (44%) safety violations** and 15 of 118 (13%) robustness failures that the hybrid grader catches. Sometimes it rationalizes the agent's behaviour.
- **Results on 14 models.** On General + Multi-turn combined, Claude Opus 4.6 leads Pass^3 at 70.4%, then Claude Sonnet 4.6 (67.8%) and GPT-5.4 (60.3%). Nemotron 3 Super is last at 5.5%.
- **Multimodal is much harder.** The best Multimodal Pass^3 is 25.7% (GPT-5.4), against 70.8% for Opus 4.6 on General. Rankings change: GPT-5.4 is third on General but first on Multimodal.
- **Errors hurt consistency, not peak skill.** With HTTP 429/500 errors and latency spikes injected into up to 60% of mock-service calls, Pass@3 stays nearly flat while Pass^3 drops by up to 24 pp. Opus 4.6 is the most resilient (56.5% Pass^3 at rate 0.6).
- **Difficulty spread.** Pass^3 on Easy tasks ranges from 14% to 75% across six models; Opus 4.6 keeps 65.1% on Hard.
- Trial-to-trial noise is small next to the gaps between models (median σ = 1.2% on General).

## Caveats

- Multi-turn tasks use Claude Opus 4.6 as both the simulated user and the judge, and Opus 4.6 is also the top-ranked model on that split.
- All models run in one shared scaffold, so the scores do not cover native harnesses.

## Concepts & entities

- [[agent-benchmark]] · [[agent-evaluation]] — trajectory-aware evaluation with safety and robustness built in.
- [[llm-as-a-judge]] · [[verifier]] — the hybrid rule-plus-judge grader.
- [[false-success]] · [[reward-hacking]] — what output-only grading fails to catch.
- [[openclaw]] · [[claude-code]] — the agent harnesses that motivate the benchmark.
- Used as an evaluation suite by [[evo-bench-can-language-models-improve-agent-harness-huang-2026]] and [[skill2env-capability-oriented-environment-synthesis-allspark-2026]].

## References

- [arXiv:2604.06132](https://arxiv.org/abs/2604.06132)
