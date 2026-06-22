---
title: "SpatialClaw: Rethinking Action Interface for Agentic Spatial Reasoning"
type: source
created: 2026-06-22
updated: 2026-06-22
sources:
  - spatialclaw-cho-2026.md
arxiv_id: "2606.13673"
authors:
  - Seokju Cho
  - et al.
first_author: Seokju Cho
year: 2026
venue: arXiv preprint
tags:
  - 2026
status: complete
importance: medium
---

# SpatialClaw: Rethinking Action Interface for Agentic Spatial Reasoning

Surfaced via the NLP newsletter (Top AI Papers of the Week, 2026-06-14–06-21).

## Summary

SpatialClaw is a training-free framework from [[nvidia]] for agentic spatial reasoning that treats **code as the action interface** ([[code-as-harness]]). A vision-language-model-backed agent reasons about 3D/4D scenes by writing one executable Python cell per step into a *stateful* Python kernel that is pre-loaded with the input frames plus a suite of perception and geometry primitives (e.g., SAM3 for segmentation, a 3D reconstruction/depth tool). Because the kernel persists state across steps, the agent conditions each new cell on all prior textual and visual outputs, letting it freely compose tools, inspect intermediate masks/point clouds, and revise its strategy mid-task.

The paper argues that prior spatial agents are bottlenecked by their interface: single-pass code commits to a full strategy before observing any result, while structured tool-call APIs constrain how operations can be composed. By contrast, SpatialClaw's open-ended code interface adapts the analysis to each problem. Across 20 spatial benchmarks spanning static and dynamic 3D/4D tasks, it reaches **59.9% average accuracy, +11.2 points over the recent spatial agent baseline**, with consistent gains across six VLM backbones from two model families and no per-benchmark or per-model adaptation.

## Key points

- **Code-as-action over a persistent Jupyter-style kernel:** the agent emits one Python cell per step; scene geometry, masks, and trajectories live as ordinary Python variables preserved across steps.
- **Perception/geometry primitives** (SAM3 segmentation, reconstruction/depth, centroid and distance helpers) are pre-loaded as callable tools the agent composes freely.
- **Iterative, feedback-driven reasoning:** the agent sees stdout, new variables, and rendered visual evidence, then self-corrects (e.g., swapping a median centroid for a KDTree nearest-point query).
- **59.9% average accuracy across 20 spatial benchmarks**, a **+11.2 point** improvement over the prior spatial agent.
- **Training-free and backbone-agnostic:** consistent gains across six VLM backbones from two model families with no fine-tuning or benchmark-specific adaptation.
- Contrasts code-as-interface against single-pass code execution and rigid structured tool-call APIs, attributing gains to interface flexibility.

## Concepts & entities

- [[nvidia]] — originating lab (work done during the first author's internship; first author affiliated with KAIST).
- [[code-as-harness]] — code as the agent's action interface over a stateful kernel.

## References

- [arXiv:2606.13673](https://arxiv.org/abs/2606.13673)
- Source PDF: `spatialclaw-cho-2026.pdf`
