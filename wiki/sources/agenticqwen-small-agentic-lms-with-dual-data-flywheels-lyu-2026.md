---
title: "AgenticQwen: Small Agentic LMs with Dual Data Flywheels"
type: source
created: 2026-06-03
updated: 2026-06-03
sources:
  - agenticqwen-small-agentic-lms-with-dual-data-flywheels-lyu-2026.md
arxiv_id: "2604.21590"
authors:
  - Yuanjie Lyu
  - Chengyu Wang
  - Haonan Zheng
  - Yuanhao Yue
  - Junbing Yan
  - Ming Wang
  - Jun Huang
first_author: Yuanjie Lyu
year: 2026
tags:
  - 2026
status: complete
importance: medium
---

# AgenticQwen: Small Agentic LMs with Dual Data Flywheels

*Surfaced via the alphaXiv / NLP-newsletter digest.*

[arXiv:2604.21590](https://arxiv.org/abs/2604.21590) — Lyu et al. (Alibaba Group), 2026.

## Summary

Industrial agent applications need models that do multi-step reasoning and tool use under strict cost and latency constraints, making *small* agentic models highly desirable — yet deployed systems still lean on expensive frontier models (GPT-5, Claude) or prohibitively large open models (Qwen3-235B). **AgenticQwen** is a family of small agentic models trained via multi-round RL on synthetic plus limited open-source data. Its training framework combines reasoning RL and agentic RL with **dual data flywheels** that automatically generate increasingly challenging tasks: a *reasoning* flywheel that raises difficulty by learning from errors, and an *agentic* flywheel that expands linear workflows into multi-branch behavior trees reflecting real-world decision complexity.

## Key points

- Motivated by the cost/latency gap: small agentic models are desirable for standardized, high-frequency tool-use and search tasks where frontier models are overkill.
- Trained with **multi-round RL** on synthetic data plus a limited amount of open-source data.
- **Reasoning flywheel**: increases task difficulty by learning from the model's own errors.
- **Agentic flywheel**: expands linear workflows into multi-branch behavior trees that better capture real decision complexity.
- Achieves strong performance on multiple public agentic benchmarks.
- In an industrial agent system, closes the gap with much larger models on search and data-analysis tasks; checkpoints and partial data released on Hugging Face.

## Concepts & entities

- [[agentic-rl]]
- [[tool-use]]
- [[llm-agent]]

## References

- [arXiv:2604.21590](https://arxiv.org/abs/2604.21590)
- Models: https://huggingface.co/collections/alibaba-pai/agenticqwen
- Code: https://github.com/haruhi-sudo/data_synth_and_rl
- Local source: `sources/agenticqwen-small-agentic-lms-with-dual-data-flywheels-lyu-2026.md`
