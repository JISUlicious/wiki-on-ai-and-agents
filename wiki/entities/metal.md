---
title: Metal
type: entity
created: 2026-09-30
updated: 2026-09-30
sources:
  - basert-native-metal-inference-apple-silicon-rathnayaka-2026.md
  - rigel-metal-tensor-compute-path-m4-max-kumaresan-2026.md
  - above-the-inner-loop-m1-amx-prefill-gemm-bhan-2026.md
status: draft
importance: medium
tags:
  - product
---

# Metal

[[apple|Apple]]'s low-level GPU programming API, and the layer every serious LLM runtime on [[apple-silicon|Apple Silicon]] ultimately runs on. In this wiki nearly every Mac prefill and decode number is a Metal GPU number, because both incumbents use it: [[mlx]] directly, and [[llama-cpp]] through its Metal backend. See [[quantization-performance]] and [[methods-to-speed-up-prompt-processing]].

## How it figures in this wiki

- **Native-Metal runtimes versus the incumbents.** [[basert-native-metal-inference-apple-silicon-rathnayaka-2026|BaseRT]] is a C++ runtime written directly on Metal, with hand-written shaders and separate decode (GEMV) and prefill (GEMM) kernels. Its thesis is that MLX and llama.cpp carry framework overhead a purpose-built Metal runtime avoids. It is competitive on decode but trails both on dense prefill.
- **What the newest Metal features actually compile to.** [[rigel-metal-tensor-compute-path-m4-max-kumaresan-2026|Rigel]] shows that on M4, Metal 4.1's `matmul2d` lowers onto the older `simdgroup_matrix` path, and FP8 matmul is *emulated*, at 0.87–0.94× of FP16. New API surface does not mean new hardware paths on older chips.
- **Beyond the GPU.** [[above-the-inner-loop-m1-amx-prefill-gemm-bhan-2026|Above the Inner Loop]] looks at prefill GEMM on the CPU-side matrix coprocessor (AMX) instead. AMX is a separate compute unit from the GPU that Metal programs, so it is a different optimization target from the Metal runtimes above.

## Related

- [[apple-silicon]] · [[apple]] · [[mlx]] · [[llama-cpp]] · [[edge-inference]] · [[hardware-accelerator]] · [[quantization-performance]]

## References

- [[basert-native-metal-inference-apple-silicon-rathnayaka-2026]] · [[rigel-metal-tensor-compute-path-m4-max-kumaresan-2026]] · [[above-the-inner-loop-m1-amx-prefill-gemm-bhan-2026]]
