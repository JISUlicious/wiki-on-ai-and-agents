---
title: "ASPIRE: Agentic Skill Programming through Iterative Robot Exploration"
type: source
created: 2026-07-06
updated: 2026-07-06
sources:
  - aspire-agentic-skill-programming-robotics-lu-2026.pdf
arxiv_id: "2607.00272"
authors:
  - Runyu Lu
  - Yubo Wu
  - Ethan Kou
  - Linxi "Jim" Fan
  - Guanzhi Wang
first_author: Runyu Lu
year: 2026
tags: [2026]
status: complete
importance: medium
---

# ASPIRE: Agentic Skill Programming through Iterative Robot Exploration

Surfaced via the NLP newsletter (Top AI Papers of the Week, 2026-06-28–07-05).

## Summary

ASPIRE (from an NVIDIA-led team with UMich, UIUC, UC Berkeley, and CMU) reframes **robot programming as continual code-as-policy learning** that compounds experience into a reusable skill library. Rather than fixed, human-engineered pipelines, it runs an open-ended learning loop in which an agent autonomously writes and refines robot control programs, diagnoses failures, synthesizes repairs, and validates outcomes. Discovered skills persist across tasks, across simulation and real-world settings, and across different embodiments, so adaptation gets faster as the library grows.

## Key points

- Three components: (1) a **closed-loop robot execution engine** exposing fine-grained multimodal traces (perception overlays, grasp candidates, motion trajectories, collision feedback); (2) a continually expanding **skill library** distilling validated fixes into reusable, transferable knowledge; (3) an **evolutionary search** generating diverse task sequences and control programs, systematically debugging beyond single-trajectory refinement.
- Surpasses prior methods by **up to 77%** on manipulation under perturbation (LIBERO-Pro) and **72%** on Robosuite's bimanual handover.
- Up to **32%** improvement on long-horizon household tasks (BEHAVIOR-1K).
- Strong zero-shot generalization: **31% success** on unseen long-horizon tasks (LIBERO-Pro Long) vs. **4%** for prior methods that rely heavily on test-time reasoning and retries.
- Skills discovered in simulation give initial evidence of **sim-to-real transfer**, cutting real-robot programming effort despite different embodiments and robot APIs.

## Concepts & entities

[[code-as-harness]] · [[skill-acquisition]] · [[self-improving-agent]]

## References

- [arXiv:2607.00272](https://arxiv.org/abs/2607.00272)
