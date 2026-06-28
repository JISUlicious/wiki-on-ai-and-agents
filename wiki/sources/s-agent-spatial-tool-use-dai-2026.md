---
title: "S-Agent: Spatial Tool-Use Elicits Reasoning for Spatial Intelligence"
type: source
created: 2026-06-29
updated: 2026-06-29
sources:
  - s-agent-spatial-tool-use-dai-2026.pdf
arxiv_id: "2606.20515"
authors:
  - Yalun Dai
  - Hao Li
  - Fangzhou Hong
  - et al.
first_author: Yalun Dai
year: 2026
venue: NTU / THU / ByteDance / NWPU (preprint)
tags: [2026]
status: complete
importance: medium
---

# S-Agent: Spatial Tool-Use Elicits Reasoning for Spatial Intelligence

Surfaced via the alphaXiv weekly digest (2026-06-24).

## Summary

S-Agent is a spatial tool-use agentic paradigm for reasoning over a continuous, evolving 3D world from streams of multi-view images and video. It reframes spatial reasoning as active spatio-temporal evidence accumulation rather than single-shot inference over static 2D frames. A VLM acts as a semantic planner that decides what evidence is needed; a hierarchy of spatial tools and experts then grounds objects in 2D, lifts them into 3D geometric evidence, and aggregates that into high-level spatial knowledge. A dual-memory system makes the process stateful across frames and reasoning steps: Scene Memory tracks grounded entities to preserve object identity and suppress duplicate evidence, while Agent Memory accumulates reasoning context. The framework is training-free at inference, but its zero-shot trajectories also distill a compact agent (S-Agent-8B) via a synthetic instruction set (S-300K).

## Key points

- VLM-as-planner orchestrates hierarchical 2D-to-3D spatial tools (grounding, depth/geometric lifting, aggregation) plus a dual-memory system.
- Dual memory: Scene Memory (evolving scene state, dedup of entities across frames) + Agent Memory (reasoning history).
- Training-free zero-shot gains: improves GPT-5.4 by 4.5% on MMSI-Bench; boosts Gemini-3-Pro (SOTA) up to 1.2% and Qwen-VL-8B up to 10.5% on MMSI-Bench.
- Distilled S-Agent-8B lifts MMSI-Bench accuracy from 31.1% to 41.6% (+10.5 pts) via S-300K trajectories, rivaling much larger models.
- Evaluated across multi-image (MMSI-Bench, ViewSpatial-Bench) and video (ReVSI, VSI-SUPER) spatial benchmarks; consistent gains for both open- and closed-source VLMs.
- Motivation: real 3D world is hidden and evolving, only partially observed per 2D frame, so persistent cross-frame/temporal evidence integration is needed.

## Concepts & entities

- [[computer-use-agent]] — S-Agent is a tool-orchestrating agent; a VLM planner drives an action loop over external spatial tools.
- [[memory-management]] — the dual-memory (Scene + Agent) design enabling stateful, persistent spatio-temporal reasoning.

## References

- arXiv: [arXiv:2606.20515](https://arxiv.org/abs/2606.20515)
- Local source: `s-agent-spatial-tool-use-dai-2026.pdf`
