---
title: "Magma: A Foundation Model for Multimodal AI Agents"
type: source
created: 2026-05-17
updated: 2026-05-17
sources:
  - magma-yang-2025.md
arxiv_id: "2502.13130"
venue: CVPR 2025
authors:
  - Jianwei Yang
  - Reuben Tan
  - Qianhui Wu
  - Ruijie Zheng
  - Baolin Peng
  - Yongyuan Liang
  - Yu Gu
  - Mu Cai
  - Seonghyeon Ye
  - Joel Jang
  - Yuquan Deng
  - Lars Liden
  - Jianfeng Gao
first_author: Jianwei Yang
year: 2025
introduces:
  - "[[magma]]"
  - "[[vision-language-action-model]]"
  - "[[set-of-mark]]"
  - "[[trace-of-mark]]"
tags:
  - 2025
status: complete
importance: high
---

# Magma: A Foundation Model for Multimodal AI Agents

[arXiv:2502.13130](https://arxiv.org/abs/2502.13130) — CVPR 2025 — Microsoft Research, University of Maryland, University of Wisconsin–Madison, KAIST, University of Washington — [project page](https://microsoft.github.io/Magma)

## Summary

Magma is the first foundation model that unifies multimodal understanding with agentic action prediction across **both digital and physical environments** from a single set of weights. Prior [[vision-language-action-model]]s such as OpenVLA and RT-2 specialize in robotic manipulation, while UI-only models like SeeClick, Pix2Act, and Ferret-UI specialize in GUI navigation; each sacrifices generic VL understanding for a task-specific action policy. Magma instead trains one decoder-only LLM (LLaMA-3-8B backbone with a ConvNeXt-XXL vision encoder) on a single agentic pretraining mix that spans UI screenshots, robotic manipulation trajectories, instructional videos, and image-text pairs — roughly **39M samples in total**.

The key insight is that combining 2D pixel actions (clicks), 7-DoF robot actions, and verbal VL tasks naively does not transfer; the input/output gap between verbal and spatial-temporal supervision is too large. Magma bridges that gap with two surrogate pretraining tasks: **[[set-of-mark]] (SoM)** overlays numerical labels on actionable elements (clickable buttons in UI, robot end-effectors and objects in robotics frames, human-hand contact points in instructional videos) so the model predicts which mark to act on, unifying action grounding into a shared selection-and-coordinate format. **[[trace-of-mark]] (ToM)** extends marks along the time axis using CoTracker point tracks, asking the model to predict future trajectories of marked points — a compact way to "look ahead" and turn unlabeled video into action supervision without needing to predict full future frames. SoM applies to all data types; ToM is skipped for UI (discrete screenshots). Camera/global motion is removed via homography transformation before extracting traces from ego-centric footage.

Magma-8B sets new state-of-the-art results on **both UI navigation** (ScreenSpot, VisualWebBench, Mind2Web, AITW) and **robotic manipulation** (SimplerEnv Bridge and Google Robot, LIBERO) — nearly doubling OpenVLA's average success rate on Google Robot (+19.6%) and beating GPT-4V+OmniParser on UI grounding zero-shot — while remaining competitive with general LMMs (LLaVA-1.5, Qwen-VL) on VQAv2, TextVQA, POPE, BLINK, and video QA. This is the first credible "one model for [[gui-agent]] + [[embodied-agent]]" demonstration, bridging the [[computer-use-agent]] and robotics literatures that had previously diverged.

## Key Points

- **Unified VLA architecture**: single decoder-only LLM (LLaMA-3-8B) with ConvNeXt-XXL vision encoder handles both 2D UI clicks and 7-DoF robot actions; all outputs (verbal, spatial coordinates, robot actions) are mapped to text tokens (last 256 LLM tokens reused for discretized robot actions, following OpenVLA).
- **[[set-of-mark]] for action grounding**: candidate actionable regions are overlaid with numerical labels at pretraining time, so the model predicts a mark ID rather than raw (x,y) coordinates. Used across UI, robotics, and video data. Bounding boxes come from DOM/HTML for web, Android view hierarchies for mobile, and OmniParser for zero-shot eval.
- **[[trace-of-mark]] for action planning**: uses CoTracker dense point tracking on a 15×15 grid to extract future trajectories; foreground/background traces separated by motion magnitude; K-means clustering selects representative traces. Forces the model to "look ahead" and converts unlabeled video into action supervision. Not applied to UI (discrete screenshots).
- **~39M pretraining samples**: SeeClick + Vision2UI UI screenshots (~2.7M), Open-X-Embodiment robotics (970K trajectories → 9.4M frame-action triplets), instructional video (Ego4D 10.2M, Epic-Kitchens, Something-Something v2 4.5M; ~25M video samples from ~4M shot-consistent clips), plus ShareGPT4V + LLaVA-1.5 + OCR data (Magma-SFT 820K).
- **Camera-motion handling**: homography transformation removes global motion from ego-centric footage (Ego4D) before trace extraction; PySceneDetect splits videos into shot-consistent segments; CLIP score < 0.25 filters off-topic clips. CoTracker trace precision validated at 0.89 on YouCook2-BB.
- **SOTA on UI navigation**: beats GPT-4V + OmniParser on ScreenSpot zero-shot (Mobile 60.4/58.5, Desktop 75.3/52.9, Web 69.1/52.0 vs. 22.6/24.5, 20.2/11.8, 9.2/8.8); 96.3 on VWB-Ele-G vs. GPT-4V's 67.5.
- **SOTA on robotic manipulation from the same backbone**: SimplerEnv Google Robot 52.3% vs. OpenVLA 34.2%; SimplerEnv Bridge 35.4% vs. OpenVLA 14.5% (+19.6% avg improvement). Magma is resilient to the real-to-sim domain gap that hurts OpenVLA (which trained on real-robot trajectories).
- **Generic VL not sacrificed**: VQAv2 80.0, TextVQA 66.5, POPE 87.4 — competitive with LLaVA-Next 7B (81.8 / 64.9 / 86.5). Magma also reaches SOTA on BLINK without instruction fine-tuning.
- **One model for digital + physical agents**: first foundation model to convincingly handle [[gui-agent]] tasks AND [[embodied-agent]] / robotics tasks from a single set of weights, while retaining LMM-grade verbal intelligence.
- **Ablations**: simply mixing UI + OXE without SoM/ToM is *worse* than either alone (task conflict). SoM + ToM resolves the conflict and yields the gains.

## Entities

- [[microsoft]]
- [[microsoft-research]]

## Concepts

- [[magma]]
- [[vision-language-action-model]]
- [[computer-use-agent]]
- [[gui-agent]]
- [[embodied-agent]]
- [[set-of-mark]]
- [[trace-of-mark]]

## Quotes

> "We present Magma, a foundation model that serves multimodal AI agentic tasks in both the digital and physical worlds. Magma is a significant extension of vision-language (VL) models in that it not only retains the VL understanding ability (verbal intelligence) of the latter, but is also equipped with the ability to plan and act in the visual-spatial world (spatial-temporal intelligence) and complete agentic tasks ranging from UI navigation to robot manipulation." — Abstract

> "Simply combining those datasets does not bring benefits to the foundation model, due to the significant gap between multimodal understanding which is mostly verbal (i.e., textual descriptions for images and videos) and the action-taking tasks which are mostly spatial (i.e., 2D coordinates for UI or 7-DoF for robot arm)." — §1, motivating SoM/ToM

> "To the best of our knowledge, Magma is the first foundation model for multimodal AI agents that can understand multimodal inputs, perform action grounding and planning for the future, and finally adapt to downstream (unseen) agentic tasks in both the digital and physical environments." — §1

> "Predicting traces uses much fewer tokens to capture much longer temporal horizon and action-related object dynamics, while disregarding ambient contents." — §3.2.2, on why ToM beats next-frame prediction

> "Magma surpasses the second-place OpenVLA by 19.6%, nearly doubling the average success rate. On those challenging tasks such as 'Put Object in Drawer' and 'Put Carrot on Plate', Magma achieves a remarkable success rate while most baselines fail entirely." — §5.1.1

## References

- Source PDF: `sources/magma-yang-2025.md`
- arXiv: <https://arxiv.org/abs/2502.13130>
- Project page: <https://microsoft.github.io/Magma>
- Related work referenced in the paper: OpenVLA (Kim et al. 2024), RT-2 (Brohan et al. 2023), SeeClick (Cheng et al. 2024), Vision2UI, Ferret-UI, Pix2Act, WebGUM, OmniParser, CoTracker (Karaev et al.), LLaVA-1.5, ShareGPT4V, Ego4D, Epic-Kitchens, Something-Something v2, Open-X-Embodiment, SimplerEnv, Mind2Web, ScreenSpot, VisualWebBench, LIBERO, BLINK.
