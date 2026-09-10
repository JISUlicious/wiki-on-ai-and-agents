---
title: Apple
type: entity
created: 2026-09-10
updated: 2026-09-10
sources:
  - llm-in-a-flash-alizadeh-2023.md
  - flash-moe-ssd-expert-streaming-woods-2026.md
status: draft
importance: medium
tags:
  - org
---

# Apple

Hardware and platform company whose relevance to this wiki is the **Apple Silicon** inference platform rather than frontier models. Two properties of the M-series chips shape every local-inference page here: **unified memory** shared by CPU and GPU with no host↔device copy, and memory bandwidth that ranges from ~120 GB/s on a base M4 to ~800 GB/s on an M2 Ultra — which makes the decode phase bandwidth-bound in a way the [[arithmetic-intensity]] and [[quantization-performance]] pages measure directly.

## Contributions this wiki tracks

- **[[mlx]]** — Apple's open-source array framework for Apple Silicon, and the mlx-lm / mlx-swift-lm LLM tooling on top of it.
- **[[llm-in-a-flash-alizadeh-2023|LLM in a flash]]** (2023) — Apple Machine Learning Research's paper on running models larger than DRAM by streaming weights from flash; the origin of [[weight-streaming]]. Apple has published no successor; the follow-on work is community-built.
- **TinyGPU approval** (2026-03-31) — the first officially sanctioned third-party discrete GPU driver on Apple Silicon, which nonetheless opens no inference-stack path today; see [[hardware-accelerator]].

## Related

- [[mlx]] · [[weight-streaming]] · [[hardware-accelerator]] · [[quantization-performance]] · [[llama-cpp]] · [[omlx-specprefill-github-2026]]

## Related 2026 sources

- [[exploring-llms-mlx-m5-neural-accelerators-apple-2025]] — the M5 Neural Accelerators measured: Qwen3-14B 4-bit 4.06× TTFT, 1.19–1.27× generation
