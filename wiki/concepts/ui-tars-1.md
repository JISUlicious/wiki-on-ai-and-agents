---
title: UI-TARS (v1)
type: concept
created: 2026-05-21
updated: 2026-05-21
sources:
  - ui-tars-1-qin-2025.md
status: complete
importance: medium
tags:
  - 2025
---

# UI-TARS (v1)

**UI-TARS** ("**T**owards **A**utomated **R**easoning **S**ystem") is [[bytedance|ByteDance Seed]]'s first-generation native [[gui-agent|GUI agent]], introduced by Qin et al. in January 2025 ([[ui-tars-1-qin-2025]]). End-to-end model that perceives the screen and emits actions directly — no DOM parsing, raw pixels.

## Core mechanism

- **Native end-to-end** GUI agent: VLM that takes a screenshot + task and emits clicks / keystrokes.
- **Base**: Qwen-2-VL 7B/72B, continually trained on ~50B tokens.
- **Four-component framework**: **perception** (visual encoding), **reasoning** (System-2 chain-of-thought), **memory** (short-term context + long-term knowledge), **action** (unified action space across web / mobile / desktop).
- **Iterative training with reflective online traces**, then refined via [[dpo|DPO]].

## What it advances

- Establishes the "**native GUI agent**" architectural class — no intermediate parsing, no script generation; the model is the entire stack.
- Headline benchmarks at release: **OSWorld 24.6** (15-step) / 22.7 (50-step), **AndroidWorld 46.6**, **ScreenSpot Pro 38.1**, **VisualWebBench 82.8**, **ScreenSpot-v2 91.6**.
- Direct predecessor of [[ui-tars-2]] (Sep 2025), which adds multi-turn RL and lifts OSWorld to 47.5.

## Relation to other concepts

- [[ui-tars-2]] — direct successor with multi-turn RL.
- [[computer-use-agent]] / [[gui-agent]] — the broader category.
- [[magma]] — contemporaneous unified VLA alternative (covers GUI + robotics).
- [[osworld]] — the canonical evaluation benchmark.

## References

- [[ui-tars-1-qin-2025]] — primary source.
- [[ui-tars-2-wang-2025]] — successor paper.
