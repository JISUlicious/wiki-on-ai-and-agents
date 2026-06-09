---
title: "Evolving Excellence: Automated Optimization of LLM-based Agents"
type: source
created: 2026-06-09
updated: 2026-06-09
sources:
  - evolving-excellence-automated-optimization-brookes-2025.md
arxiv_id: "2512.09108"
authors:
  - Paul Brookes
  - et al.
first_author: Paul Brookes
year: 2025
tags:
  - 2025
status: complete
importance: medium
---

# Evolving Excellence: Automated Optimization of LLM-based Agents

Surfaced while researching [[agentic-harness-engineering]].

[arXiv:2512.09108](https://arxiv.org/abs/2512.09108) — Paul Brookes et al. (TurinTech AI, with UK universities), December 2025.

## Summary

LLM agents often underperform because of suboptimal configurations — poorly tuned prompts, tool descriptions, and parameters that normally take weeks of manual refinement. Existing optimizers are either too complex for general use or tune components in isolation, missing critical interdependencies among prompts, tool docs, and parameters. The paper presents **Artemis**, a no-code evolutionary optimization platform that *jointly* optimizes an agent's whole configuration via semantically-aware genetic operators. Given only a benchmark script and natural-language goals, Artemis auto-discovers configurable components, extracts performance signals from execution logs, and evolves configurations without architectural changes — treating the agent as a black box.

Across four agent systems it reports: +13.6% acceptance rate for the ALE Agent (AtCoder Heuristic Contest), a statistically significant +10.1% for Mini-SWE Agent on SWE-Perf, a 36.9% token reduction for a CrewAI agent on Math Odyssey, and +22% accuracy for a Qwen2.5-7B-powered MathTales-Teacher agent on GSM8K — showing it works on both commercial and local models. The authors position automated, no-code agent optimization as practical and accessible to practitioners without evolutionary-algorithm expertise.

## Key points

- Targets the agent *configuration* — prompts, tool descriptions, parameters — as a jointly optimized space rather than per-component tuning.
- Artemis: no-code evolutionary platform using semantically-aware mutation and crossover operators tailored to natural-language components.
- Inputs are minimal: a benchmark script plus natural-language goals; it auto-discovers configurable components and mines execution logs for fitness signals.
- Treats agents as black boxes — no architectural modifications required.
- Results across 4 systems: ALE Agent +13.6% acceptance; Mini-SWE Agent +10.1% on SWE-Perf; CrewAI -36.9% tokens on Math Odyssey; MathTales-Teacher (Qwen2.5-7B) +22% on GSM8K.
- Demonstrates optimization of both commercial and local-model agents, surfacing non-obvious cross-component optima.

## Concepts & entities

- [[agentic-harness-engineering]] — Artemis optimizes the configuration layer wrapping the model (the harness around it).
- [[skill-optimization]] — automated tuning of prompts/tool descriptions/parameters as the agent's skill configuration.
- [[self-improving-agent]] — evolutionary search drives automated, iterative agent improvement.
- [[llm-agent]] — the systems being optimized (ALE, Mini-SWE, CrewAI, MathTales).
- [[multi-agent]] — multi-component agent pipelines (e.g., CrewAI) among the targets.

## References

- [[agentic-harness-engineering]] — primary concept this source backs.
- [[skill-optimization]] — closely related optimization framing.
