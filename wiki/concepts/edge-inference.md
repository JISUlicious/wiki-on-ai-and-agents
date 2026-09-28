---
title: Edge Inference
type: concept
created: 2026-09-10
updated: 2026-09-10
sources:
  - llm-in-a-flash-alizadeh-2023.md
  - neuroprefetcher-delta-prefetching-dhar-2026.md
  - persistent-state-dataflow-accelerator-linear-attention-decode-gupta-2026.md
  - ssd-offloading-moe-weights-considered-harmful-kyung-2025.md
status: draft
importance: medium
---

# Edge Inference

Running LLMs on a single consumer or embedded device — a laptop, a phone, a Jetson — where the binding constraints are DRAM capacity, memory bandwidth, and power, and where the alternative to a clever technique is usually that the model does not run at all. That last property is what separates edge results from datacenter ones: a 7× slowdown that would be absurd for a serving fleet is a win if it turns "does not fit" into "fits".

## The pages that live here

- **[[weight-streaming]]** — models larger than DRAM, read per token from SSD. The main edge technique of 2026; ten [[apple-silicon|Apple Silicon]] implementations surveyed.
- **[[llm-in-a-flash-alizadeh-2023]]** — the origin, on an M1 Max and an M2 Ultra. Candid that energy per token goes *up*.
- **[[neuroprefetcher-delta-prefetching-dhar-2026]]** — the same idea on an NVIDIA Jetson AGX Orin with 32 GiB, 7.9–12× over demand paging, at a measured accuracy cost.
- **[[persistent-state-dataflow-accelerator-linear-attention-decode-gupta-2026]]** — the ~10 W regime where a partial FPGA accelerator makes sense; not reachable from a Mac ([[hardware-accelerator]]).
- **[[ssd-offloading-moe-weights-considered-harmful-kyung-2025]]** — the energy objection, framed for the datacenter but naming mobile and edge as the domain where flash at ~10 pJ/bit would pay off first.
- **[[quantization-performance]]** — the Apple Silicon prefill/decode measurements every local-inference decision starts from.
- **[[hardware-accelerator]]** — why neither FPGA nor eGPU is a path on Apple Silicon today.

## Related

- [[apple]] · [[mlx]] · [[llama-cpp]] · [[arithmetic-intensity]] · [[kv-cache]]
