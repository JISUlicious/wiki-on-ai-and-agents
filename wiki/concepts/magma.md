---
title: Magma
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

# Magma

**Magma** is [[microsoft]] Research's unified [[vision-language-action-model]] foundation model that handles both GUI navigation and robotic manipulation from a single set of weights. It is the first credible "one model for digital + physical agents" demonstration, bridging the previously divergent [[computer-use-agent]] and embodied-robotics literatures while retaining generic vision-language understanding competitive with LLaVA-class LMMs.

## Core mechanism

- **LLaMA-3-8B + ConvNeXt-XXL backbone** — a decoder-only LLM with a ConvNeXt-XXL vision encoder; all outputs (text, 2D click coordinates, 7-DoF robot actions) are mapped into the text-token space, with the last 256 LLM tokens repurposed for discretized robot actions (following OpenVLA).
- **~39M pretraining samples** — a single agentic pretraining mix spanning UI screenshots (SeeClick, Vision2UI ~2.7M), Open-X-Embodiment robotics (~9.4M frame-action triplets), instructional video (Ego4D, Epic-Kitchens, Something-Something v2; ~25M video samples), and standard VL data (ShareGPT4V, LLaVA-1.5, Magma-SFT 820K).
- **[[set-of-mark]] for action grounding** — overlays numerical labels on actionable elements (clickable UI buttons, robot end-effectors, hand-contact points) so the model predicts a mark ID rather than raw coordinates, unifying action grounding into a shared selection format across domains.
- **[[trace-of-mark]] for action planning** — uses CoTracker dense point tracking to extract future trajectories of marked points along the time axis, turning unlabeled video into action supervision while skipping ambient pixel-level next-frame prediction.
- **Zero-shot UI grounding beats GPT-4V + OmniParser** — on ScreenSpot Magma scores 60.4/58.5 (Mobile), 75.3/52.9 (Desktop), 69.1/52.0 (Web) versus GPT-4V+OmniParser's much lower numbers; also nearly doubles OpenVLA on SimplerEnv Google Robot (+19.6%).

## What it advances

Magma is the empirical proof that a single foundation model can act competently in both pixel-space GUIs and continuous robot control without sacrificing VL understanding — previous [[vision-language-action-model]]s like OpenVLA and RT-2 specialized in manipulation, while UI-specialist models (SeeClick, Ferret-UI, Pix2Act) sacrificed generic VL ability. The ablation result that naively mixing UI + OXE data is *worse* than either alone, but that SoM + ToM resolves the conflict, establishes a recipe for cross-domain agentic pretraining.

## Relation to other concepts

- Instance of the [[vision-language-action-model]] paradigm; extends VLM with an action output head.
- Targets [[computer-use-agent]] / [[gui-agent]] tasks (ScreenSpot, Mind2Web, AITW) alongside embodied-agent benchmarks (SimplerEnv, LIBERO).
- Introduces the [[set-of-mark]] and [[trace-of-mark]] grounding tricks that other 2025 VLA work has begun adopting.
- Produced by [[microsoft]] Research; positioned against OpenVLA (Stanford) and RT-2 ([[google-deepmind]]).
- Complements [[ui-tars-2]] in the GUI-agent space, though UI-TARS-2 is GUI-specialist and RL-trained while Magma is cross-domain and pretrained.

## References

- [[magma-yang-2025]] — CVPR 2025 paper introducing Magma; [arXiv:2502.13130](https://arxiv.org/abs/2502.13130)
- Project page: <https://microsoft.github.io/Magma>
