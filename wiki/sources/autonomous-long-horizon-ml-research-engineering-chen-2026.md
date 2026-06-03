---
title: "Toward Autonomous Long-Horizon Engineering for ML Research"
type: source
created: 2026-06-03
updated: 2026-06-03
sources:
  - autonomous-long-horizon-ml-research-engineering-chen-2026.md
arxiv_id: "2604.13018"
authors:
  - Guoxin Chen
  - Jie Chen
  - Lei Chen
  - Jiale Zhao
  - Fanzhe Meng
  - Wayne Xin Zhao
  - Ruihua Song
  - Ji-Rong Wen
  - Kai Jia
first_author: Guoxin Chen
year: 2026
tags:
  - 2026
status: complete
importance: medium
---

# Toward Autonomous Long-Horizon Engineering for ML Research

[arXiv:2604.13018](https://arxiv.org/abs/2604.13018) — Chen, Chen, Chen, Zhao, Meng, Zhao, Song, Wen, Jia (Gaoling School of AI, Renmin University of China; AweAI), 2026. Surfaced via the alphaXiv / NLP-newsletter digest.

## Summary

Agentic systems increasingly automate pieces of AI research, but turning underspecified research objectives into runnable, experimentally validated ML systems remains the bottleneck. The authors frame this as **long-horizon ML research engineering**: converting a research spec into a runnable system through repeated implementation, experimentation, and refinement, while sustaining cumulative progress across heterogeneous stages under delayed, confounded feedback. They introduce **AiScientist**, a multi-agent system built on the principle of **"thin control over thick state"**: a lightweight hierarchical research team coordinates through a **File-as-Bus** workspace that preserves decision-relevant artifacts across roles and invocations.

On **PaperBench**, AiScientist beats the strongest matched baselines by 9.92 / 11.15 points (Gemini-3-Flash / GLM-5). On **MLE-Bench Lite** it reaches **81.82 Any-Medal%** under both backbones (+4.55 / +16.67 over baselines, +13.64 over a Codex/GPT-5.5 xhigh frontier harness). Ablations show durable project state is central: removing File-as-Bus drops PaperBench by 6.41 points and MLE-Bench Lite Any-Medal% by 31.82 points. Conclusion: long-horizon AI research is not only a stronger-local-reasoning problem but a **systems problem** of maintaining cumulative, inspectable progress.

## Key points

- **Problem framing:** long-horizon ML research engineering — spec → runnable, validated ML system via iterative implement/experiment/refine under delayed feedback.
- **AiScientist:** multi-agent hierarchical research team built around "thin control over thick state."
- **File-as-Bus workspace:** shared file-based medium preserving decision-relevant artifacts across roles and invocations; the core durable-state mechanism.
- **PaperBench gains:** +9.92 (Gemini-3-Flash) and +11.15 (GLM-5) over strongest matched baselines.
- **MLE-Bench Lite:** 81.82 Any-Medal% under both backbones; +13.64 over a Codex/GPT-5.5 xhigh frontier harness.
- **Ablation:** removing File-as-Bus costs 6.41 PaperBench points and 31.82 MLE-Bench Lite Any-Medal% — durable state drives later-round refinement.

## Concepts & entities

- [[multi-agent]] — AiScientist is a hierarchical multi-agent research team.
- [[llm-agent]] — autonomous agent for end-to-end ML research engineering.
- [[agent-benchmark]] — evaluated on PaperBench and MLE-Bench Lite.
- [[code-generation]] — produces runnable, experimentally validated ML system code.
- [[context-assembly]] — File-as-Bus is a shared persistent-state / context mechanism across agent invocations.

## References

- [arXiv:2604.13018](https://arxiv.org/abs/2604.13018)
