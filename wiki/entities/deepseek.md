---
title: DeepSeek
type: entity
created: 2026-05-17
updated: 2026-05-17
sources:
  - deepseek-r1-2025.md
status: draft
importance: medium
tags:
  - org
---

DeepSeek is a Chinese AI lab known for releasing open-weights frontier-class language models. Their lineage includes DeepSeek-V2/V3 base models and the [[deepseek-r1]] reasoning model. The lab also originated [[grpo]] (Group Relative Policy Optimization) via the DeepSeekMath paper, a now-widely-used RL algorithm for LLM post-training.

DeepSeek's open releases — particularly the R1 weights and recipe — substantially shifted the global research landscape on [[agentic-rl]] and verifier-driven reasoning, demonstrating that strong long-CoT behavior can be reached without proprietary RLHF stacks.

## Notable work

- [[deepseek-r1]] — reasoning-focused open-weights model trained with [[grpo]] and rule-based verifiable rewards (see [[deepseek-r1-2025]]).
- DeepSeekMath — original venue introducing [[grpo]].
- DeepSeek-V3 — large MoE base model underlying R1.
- Contributions to [[agentic-rl]] practice via published recipes.
- **DeepSeek Harness v0.1** (Aug 2026, developer preview, MIT) — an open-sourced agent [[code-as-harness|harness]] built on the Cordis meta-framework, where models, tools, [[agent-skills|skills]], sessions, sandboxes, storage, loops, scheduling and UI are all independent mountable plugins. Its distinguishing design choice is an **append-only session log**: everything the model sees is recorded, so sessions can be resumed, forked, searched and replayed rather than reconstructed from chat history. Four runtime modes (Standard / Code / Minimal / Creator).
- **DeepSeek-V4-Pro-0813** — GA release centred on agent workloads, with low/high/max reasoning-effort tiers ([[reasoning-effort]]) and a native [[responses-api|Responses API]]. Reported agentic scores: 87.9 [[terminal-bench|Terminal Bench 2.1]], 62.7 DeepSWE, 74.1 Toolathlon-Verified, 83.3 CyberGym, 31.8 public AutomationBench — all measured *through DeepSeek Harness in minimal mode*, which [[dataspace-benchmarking-data-agents-li-2026|harness-sensitivity results]] suggest is not a harness-neutral number.

> [!note] Product-news provenance
> The two entries above come from a newsletter digest (AI Agents Weekly, 2026-08-15), not a paper or a first-party technical report. Benchmark figures are vendor-reported and unverified here.

## References

- [[deepseek-r1-2025]]
