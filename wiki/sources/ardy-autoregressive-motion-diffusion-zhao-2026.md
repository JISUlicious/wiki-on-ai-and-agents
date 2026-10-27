---
title: "ARDY: Autoregressive Diffusion with Hybrid Representation for Interactive Human Motion Generation"
type: source
created: 2026-10-24
updated: 2026-10-24
sources:
  - ardy-autoregressive-motion-diffusion-zhao-2026.md
tags:
  - 2026
  - paper
status: complete
importance: medium
authors:
  - Kaifeng Zhao
  - Mathis Petrovich
  - Haotian Zhang
  - Tingwu Wang
  - Siyu Tang
  - Davis Rempe
introduces:
  - [[ardy]]
year: 2026
venue: ACM TOG 45(4) / SIGGRAPH 2026
arxiv_id: "2607.08741"
---

# ARDY: Autoregressive Diffusion for Interactive Motion

[arXiv:2607.08741](https://arxiv.org/abs/2607.08741), 2026-07-09, published in ACM Transactions on Graphics 45(4) (SIGGRAPH 2026). Authors from [[nvidia]] and ETH Zürich.

## What it is

[[ardy|ARDY]] is the **real-time, streaming** companion to [[kimodo|Kimodo]]. It keeps Kimodo-style text and kinematic control, adds **mouse and keyboard velocity control**, and generates motion autoregressively, a window at a time.
- **Why:** offline models such as Kimodo and MaskControl are too slow for interaction, while earlier online models (A-MDM, CAMDM, DartControl, DiP, MotionStreamer) give up text or constraints, or have short context.

## How it works

- **Hybrid representation:**
  - an explicit **root** (position plus heading);
  - **latent body tokens** from a causal transformer tokenizer: patches of 4 frames, FSQ quantization, 8+8 layers. The decoder converts the global root to a local one, which cuts foot skate.
- **Denoiser:** two-stage, root then body (Kimodo's design), about 156M parameters, with an LLM2Vec text encoder.
- **Windows:** 40 frames (2 s at 20 fps) are generated at a time, with up to 8 s of history. **Goals beyond the window** enter as extra tokens.
- **Steps:** **10 diffusion steps** by default, or **4** for the fast model.
- **Latency:** **33 ms** (4-step) and **63 ms** (10-step) per generation on an RTX 4090. Latency-aware replanning uses a frame buffer.

## Results

- **Ablations on Rigplay:**
  - The hybrid representation clearly beats a purely explicit one: R-precision 65.47 vs 53.90, waypoint error 0.024 m vs 0.203 m.
  - A one-stage denoiser is much worse on constraints (keyframe error 0.101 m vs 0.023 m).
  - 1 step collapses, 4 steps is acceptable.
- **HumanML3D vs MaskControl:** constraint error 4.15 cm vs 46.18 cm without optimization, at 0.15 s vs 0.46 s per generation.
- **Long-horizon goals vs DiP:** error 2.92 cm vs 17.64 cm on goals beyond the window.
- **Perceptual study:** ARDY preferred 64.6–67.5% of the time, DiP 4.2–9.2%.

## Limitations

- Attends to the full history, which is inefficient for very long sessions.
- Multi-step diffusion cost remains.
- Purely kinematic: foot skate and jitter remain.

## Related

[[ardy]] · [[kimodo-kinematic-motion-diffusion-nvidia-2026]] (Kimodo-SOMA-RP-ArdyRep-v1.2 uses ARDY's representation) · [[text-to-motion]] · [[sonic-supersizing-motion-tracking-luo-2025]]
