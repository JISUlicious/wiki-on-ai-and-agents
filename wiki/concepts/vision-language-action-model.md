---
title: Vision-Language-Action Model
type: concept
created: 2026-05-17
updated: 2026-05-17
sources:
  - magma-yang-2025.md
status: complete
importance: high
tags:
  - 2025
---

# Vision-Language-Action Model (VLA)

A **vision-language-action model (VLA)** is a foundation model that extends a vision-language model with an *action output head*, emitting executable actions — UI clicks, keyboard input, 7-DoF robot joint commands, navigation primitives — conditioned on visual observations and natural-language instructions. VLAs are the standard architecture for both [[embodied-agent]]s and modern [[computer-use-agent]]s, replacing earlier modular planner-controller stacks with a single end-to-end policy.

## Core mechanism

- **VLM + action output head** — a decoder-only LLM backbone with a visual encoder, where action tokens (discretized robot joint deltas, click coordinates, button IDs) are folded into the text-token space. In OpenVLA and [[magma]] the last 256 LLM vocabulary tokens are repurposed for robot-action bins.
- **Lineage** — descends from RT-2 ([[deepmind]], 2023), OpenVLA (Stanford, 2024), and RoboFlamingo, which established the recipe of treating actions as just-another-modality emitted by an LLM trained on paired (observation, instruction, action) trajectories.
- **[[magma]] unifies GUI and robotic domains** — until 2025 the VLA literature was split into UI-specialists (SeeClick, Ferret-UI, Pix2Act) and manipulation-specialists (OpenVLA, RT-2); [[magma]] is the first to convincingly handle both from one set of weights.
- **[[set-of-mark]] / [[trace-of-mark]] grounding** — the [[magma]]-introduced surrogate pretraining tasks that bridge the input/output gap between verbal VL supervision and spatial-temporal action supervision; SoM overlays numerical labels on actionable regions, ToM extracts future trajectories of marked points.
- **Evaluation surface** — UI-side benchmarks (ScreenSpot, Mind2Web, AITW, VisualWebBench) and robotics benchmarks (SimplerEnv, LIBERO, Open-X-Embodiment subsets) are both standard for VLA evaluation.

## What it advances

VLAs collapse the perception → planning → control pipeline into a single learned policy, allowing the same scaling-law tailwinds that lifted LLMs to lift agents that *physically act*. They are the architectural substrate underneath the 2025 [[computer-use-agent]] wave and the modern robotics-foundation-model thrust ([[deepmind]] Gemini Robotics, Physical Intelligence π-series, Figure Helix).

## Relation to other concepts

- Architectural class to which [[magma]], OpenVLA, RT-2, and RoboFlamingo all belong.
- Powers both [[computer-use-agent]]s (when actions are clicks/keys) and [[embodied-agent]]s (when actions are joint commands).
- Built on top of vision-language models (LLaVA, Qwen-VL, PaLI, etc.) — a VLA is "a VLM that can also act."
- Often co-evolves with grounding techniques like [[set-of-mark]] and [[trace-of-mark]].

## References

- [[magma-yang-2025]] — current unified-VLA reference; [arXiv:2502.13130](https://arxiv.org/abs/2502.13130)
- Related prior work: OpenVLA (Kim et al. 2024), RT-2 (Brohan et al. 2023), RoboFlamingo.
