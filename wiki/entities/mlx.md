---
title: MLX
type: entity
created: 2026-09-10
updated: 2026-09-10
sources:
  - omlx-specprefill-github-2026.md
  - flash-moe-ssd-expert-streaming-woods-2026.md
status: draft
importance: medium
tags:
  - product
---

# MLX

[[apple|Apple]]'s open-source array framework for machine learning on [[apple-silicon|Apple Silicon]] ([ml-explore/mlx](https://github.com/ml-explore/mlx)). Its defining properties are **unified memory** — arrays live in memory the CPU and GPU both address, so there is no host↔device copy — and **lazy evaluation**, where operations build a graph that is materialized only when a value is needed. The GPU backend is Metal.

The LLM layer is **mlx-lm** (Python: `mlx_lm.load`, `generate`, an OpenAI-compatible server, quantization) and **mlx-swift-lm**. Converted checkpoints live under the `mlx-community` organization on [[huggingface|Hugging Face]].

## Facts this wiki has verified

- **`load(..., lazy=True)` memory-maps the safetensors and defers materialization.** This is the hook every SSD expert streamer on Apple Silicon uses: replace the MoE module (`SwitchGLU`) before the lazy state is materialized, and non-resident experts never enter RAM. See [[weight-streaming]].
- **Upstream has no expert streaming.** The mlx-lm feature request (#1438, opened 2026-06-27) had 32 comments and **zero maintainer replies**, no label, no assignee and no PR as of 2026-09-10. Everything in that space is third-party.
- **The CUDA backend does not run on macOS.** It exists in-tree, but per the project's own discussion (ml-explore/mlx #2422) it targets Linux servers and "does not add CUDA support on Mac". So MLX cannot drive an external GPU attached to a Mac; see [[hardware-accelerator]].

## Ecosystem pages in this wiki

- [[omlx-specprefill-github-2026]] — oMLX, the menu-bar inference server: continuous batching, SSD-tiered KV cache, positional SpecPrefill, MTP and DFlash [[speculative-decoding]], oQ quantization.
- [[weight-streaming]] — the SSD expert-streaming projects built on or beside MLX: SwiftLM, TurboFieldfare, mlx-moe, mlx-moe-offload, mlx-flash, TensorFold, and the oMLX PR.
- [[quantization-performance]] — Apple Silicon prefill/decode measurements.
- [[llama-cpp]] — the other Apple Silicon runtime; Metal backend, `mmap`-based loading.

## Related

- [[apple]] · [[hardware-accelerator]] · [[arithmetic-intensity]]

## Related 2026 sources

- [[exploring-llms-mlx-m5-neural-accelerators-apple-2025]] — Apple's M5 benchmark: 3.3–4.1× TTFT over M4 through MLX, automatic on macOS 26.2+
- [[basert-native-metal-inference-apple-silicon-rathnayaka-2026]] — a closed-core native-Metal runtime benchmarked against MLX v0.31.2 on M4 Pro: decode 0.90–1.35× (the 1.35× is one 0.6B cell); **dense prefill 0.82–0.90× MLX at 2,048 tokens**; MoE prefill 1.42–1.78× at 128 tokens decaying to ~1.0× at 2,048. No M5 / Neural Accelerator comparison
