---
title: "ARC-AGI-3: A New Challenge for Frontier Agentic Intelligence"
type: source
created: 2026-06-03
updated: 2026-06-03
sources:
  - arc-agi-3-arc-prize-foundation-2026.md
arxiv_id: "2603.24621"
authors:
  - ARC Prize Foundation
  - François Chollet
  - Mike Knoop
  - Gregory Kamradt
first_author: ARC Prize Foundation
year: 2026
tags:
  - 2026
status: complete
importance: medium
---

# ARC-AGI-3: A New Challenge for Frontier Agentic Intelligence

Surfaced via the alphaXiv / NLP-newsletter digest.

[arXiv:2603.24621](https://arxiv.org/abs/2603.24621) — ARC Prize Foundation (François Chollet, Mike Knoop, Gregory Kamradt et al.), 2026.

## Summary

ARC-AGI-3 is an **interactive benchmark for agentic intelligence**: novel, abstract, turn-based environments in which an agent must explore, infer goals, build an internal model of environment dynamics, and plan effective action sequences — all **without explicit instructions**. Like ARC-AGI-1 and 2, it targets *fluid adaptive efficiency* on novel tasks while deliberately avoiding language and external knowledge, leaning only on **Core Knowledge priors** (objectness, basic geometry). Environments are difficulty-calibrated against human test-takers.

The headline gap: humans solve **100%** of the environments, while frontier AI systems as of March 2026 score **below 1%**. The paper presents the benchmark design, an **efficiency-based scoring framework grounded in human action baselines** (intelligence defined as action efficiency across exploration, goal inference, world-modeling, and planning), and the methodology used to construct, validate, and calibrate environments — including a private dataset for official scoring. It situates ARC-AGI-3 in the series lineage from "On the Measure of Intelligence" (ARC-AGI-1, 2019) and ARC-AGI-2 (2025).

## Key points

- Shifts ARC-AGI from **static grid puzzles to interactive, turn-based environments** for agentic evaluation.
- Agents must **explore, infer goals, model dynamics, and plan** with no explicit instructions.
- Uses only **Core Knowledge priors**; avoids language and external/world knowledge to resist memorization shortcuts.
- **Humans: 100% solve rate; frontier AI: <1%** (as of March 2026) — a stark capability gap.
- **Efficiency-based scoring**: single scalar **action efficiency** measure grounded in human action baselines.
- Private dataset kept inaccessible to model developers for official scoring and verification.

## Concepts & entities

- [[agent-benchmark]] — ARC-AGI-3 is an interactive agentic-intelligence benchmark (no `arc-agi` concept page yet).
- [[skill-acquisition]] — Intelligence framed as skill-acquisition / adaptive efficiency on novel tasks.
- [[llm-agent]] — Evaluates frontier agentic systems.
- [[latent-reasoning|reasoning]] — Requires multi-step planning and world-model inference.
- [[francois-chollet|François Chollet]] — Co-founder/benchmark designer (ARC Prize Foundation).

## References

- "On the Measure of Intelligence" (Chollet, 2019) — origin of ARC-AGI-1 and the efficiency framing.
- ARC-AGI-1 (2019), ARC-AGI-2 (March 2025) — static-task predecessors in the series.
