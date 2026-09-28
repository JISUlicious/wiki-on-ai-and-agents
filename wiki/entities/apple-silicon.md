---
title: Apple Silicon
type: entity
created: 2026-09-29
updated: 2026-09-29
sources:
  - above-the-inner-loop-m1-amx-prefill-gemm-bhan-2026.md
  - rigel-metal-tensor-compute-path-m4-max-kumaresan-2026.md
  - exploring-llms-mlx-m5-neural-accelerators-apple-2025.md
  - basert-native-metal-inference-apple-silicon-rathnayaka-2026.md
  - omlx-specprefill-github-2026.md
status: draft
importance: medium
tags:
  - product
---

# Apple Silicon

[[apple|Apple]]'s M-series system-on-chip line (M1 through M5) used in Macs. It is the main **local-inference platform** in this wiki. Two properties shape every page that touches it: **unified memory** shared by CPU and GPU with no host-to-device copy, and memory bandwidth that varies widely across tiers. Bandwidth sets decode speed; matrix throughput sets prefill speed.

## Three compute paths

- **GPU (Metal).** This is what [[mlx]] and [[llama-cpp]]'s Metal backend use, so nearly every Apple Silicon prefill/decode table here is a GPU number ([[quantization-performance]], [[methods-to-speed-up-prompt-processing]]).
  - On M4, [[rigel-metal-tensor-compute-path-m4-max-kumaresan-2026|Rigel]] shows Metal 4.1's `matmul2d` lowers onto the older `simdgroup_matrix` path, with FP8 matmul *emulated* at 0.87–0.94× FP16.
  - On M5, [[exploring-llms-mlx-m5-neural-accelerators-apple-2025|Apple's M5 benchmark]] reports GPU Neural Accelerators giving 3.3–4.1× faster time-to-first-token over M4, while token generation improves only about 1.2×.
- **CPU matrix coprocessor.** Accelerate dispatches fp32 GEMM to AMX on M1–M3 and Arm SME on M4+. [[above-the-inner-loop-m1-amx-prefill-gemm-bhan-2026|Above the Inner Loop]] finds M1 AMX throughput bounded by load issue, not FMA rate.
- **No external accelerator path.** There is no FPGA path and no practical eGPU path today (see [[hardware-accelerator]]). For models larger than RAM, streaming MoE experts from the internal SSD is the path that works ([[weight-streaming]]).

## Runtimes in this wiki

[[mlx]] (Apple's framework) and [[llama-cpp]] are the two incumbents. [[basert-native-metal-inference-apple-silicon-rathnayaka-2026|BaseRT]] is a native-Metal challenger that is competitive on decode but trails both on dense prefill. [[omlx-specprefill-github-2026|oMLX]] is a menu-bar inference server with SpecPrefill, mixed-precision quantization and an SSD-spilled [[kv-cache]]. See [[edge-inference]] for the broader picture.

## Related

- [[apple]] · [[mlx]] · [[llama-cpp]] · [[edge-inference]] · [[weight-streaming]] · [[quantization-performance]] · [[hardware-accelerator]] · [[arithmetic-intensity]]

## References

- [[above-the-inner-loop-m1-amx-prefill-gemm-bhan-2026]] · [[rigel-metal-tensor-compute-path-m4-max-kumaresan-2026]] · [[exploring-llms-mlx-m5-neural-accelerators-apple-2025]] · [[basert-native-metal-inference-apple-silicon-rathnayaka-2026]] · [[omlx-specprefill-github-2026]]
