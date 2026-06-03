---
title: "Composer 2 Technical Report"
type: source
created: 2026-06-03
updated: 2026-06-03
sources:
  - composer-2-technical-report-cursor-2026.md
arxiv_id: "2603.24477"
authors:
  - Cursor Research Team
first_author: Cursor Research Team
year: 2026
tags:
  - 2026
status: complete
importance: medium
---

# Composer 2 Technical Report

Surfaced via the alphaXiv / NLP-newsletter digest.

[arXiv:2603.24477](https://arxiv.org/abs/2603.24477) — Cursor Research Team, 2026.

## Summary

Composer 2 is a model specialized for **agentic software engineering**, built by the Cursor Research Team to combine strong long-horizon planning and coding intelligence with fast, interactive use. It is trained in two phases on a 1.04T-parameter / 32B-active **Mixture-of-Experts** base: continued pretraining to improve latent coding ability, followed by large-scale **asynchronous reinforcement learning** to improve end-to-end coding through stronger reasoning, accurate multi-step execution, and long-horizon coherence.

A core tenet is minimizing train-test mismatch by training the model **inside the same Cursor harness** used in deployment — equivalent tools, structure, and environments matched to real problems. The team introduces **CursorBench**, a benchmark derived from real software-engineering problems in large codebases (including their own), and reports frontier-level results: 61.3 on CursorBench (a large jump over prior Composer models), 61.7 on Terminal-Bench, and 73.7 on SWE-bench Multilingual in their harness — competitive with state of the art while cheaper to serve due to coding specialization.

## Key points

- **Domain-specialized coding model**: trades general breadth for frontier coding/agentic performance at lower serving cost.
- Base is a **1.04T-param / 32B-active MoE**; multi-token-prediction (MTP) layers trained with self-distillation.
- Two-phase training: **continued pretraining** then **asynchronous RL** with multi-sample policy gradients; MoE-routing replay to reduce sampling/training policy mismatch.
- **Harness-matched training**: trains in the deployed Cursor harness with equivalent tools to minimize train-test mismatch.
- Introduces **CursorBench** from real large-codebase SWE tasks (less-specified than public benchmarks).
- Scores: CursorBench 61.3, Terminal-Bench 61.7, SWE-bench Multilingual 73.7 (in-harness) — competitive with SOTA.

## Concepts & entities

- [[mixture-of-experts]] — 1.04T/32B-active MoE backbone.
- [[agentic-rl]] — Large-scale asynchronous RL for end-to-end coding.
- [[code-as-harness]] — Trained inside the same Cursor agent harness used at deploy time.
- [[swe-bench]] — Reports SWE-bench Multilingual; also Terminal-Bench and CursorBench.
- [[cursor|Cursor]] — Authoring lab and product.
- [[llm-agent]] — Composer 2 powers software-engineering agents.

## References

- SWE-bench Multilingual [28]; Terminal-Bench [39] — public evaluation benchmarks.
- CursorBench — internal real-world SWE benchmark introduced in this report.
