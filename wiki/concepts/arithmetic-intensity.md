---
title: Arithmetic Intensity
type: concept
created: 2026-09-09
updated: 2026-09-10
sources:
  - llm-inference-roofline-yuan-2024.md
  - persistent-state-dataflow-accelerator-linear-attention-decode-gupta-2026.md
  - quantization-performance.md
status: complete
importance: high
---

# Arithmetic Intensity

`I = FLOPs / bytes moved`. The single number that decides whether a kernel is limited by compute or by memory bandwidth, and therefore which optimizations can possibly help it.

A kernel is **compute-bound** iff `I` exceeds the hardware's **ridge point** `I_ridge = peak_FLOPS / peak_bandwidth` — roughly **200 OPs/byte** for an A6000 at 155 TFLOPS / 768 GB/s. Below the ridge you are waiting on memory and adding FLOPs is free; above it you are waiting on arithmetic and moving fewer bytes buys nothing.

Nearly every performance result in this wiki reduces to which side of that line a phase sits on.

## The measured split in LLM inference

Per-sub-step on Llama-2-7B ([[llm-inference-roofline-yuan-2024]]):

| Phase | Sub-step | `I` | Bound |
|---|---|---|---|
| **Prefill** | FFN (gate/up/down) | **1,215** | compute |
| **Prefill** | q/k/v/o projections | **1,024** | compute |
| **Prefill** | `qk_matmul` | 114 | **memory** |
| **Decode** | all projections | **~1** | memory |
| **Decode** | attention ops | 0.99 | memory |

Decode sits **~200× below** the ridge; prefill sits far above it. That ~1000× gap is why prefill and decode need opposite optimizations, and why advice for one is usually useless for the other.

Note the third row: `qk_matmul` is memory-bound *even during prefill*, which is exactly the gap [[flash-attention]] closes.

## Consequences that recur across the wiki

**Weight-only quantization helps decode and can hurt prefill.** W4A16 shrinks bytes but leaves FLOPs unchanged, so it raises `I`. Below the ridge that is pure win; above it, dequantization is added work on the critical path. Measured on Apple Silicon: 4-bit makes decode **2.3–2.9× faster** and prefill **2–12% slower** (see [[quantization-performance]]).

**Batching moves you rightward across the ridge.** [[marlin-frantar-2024|Marlin]] quantifies the crossover — ~3.9× at batch ≤16–32, decaying to 1.5× at batch 128 — with the rule that W4A16 wins while you do fewer than ~25–50 MACs per 4-bit weight.

**Attention's share grows with context, so the right lever changes with N.** [[sparse-prefill]] records the climb: attention is ~¼ of prefill at 10K, 70–80% at 128K, >90% past 300K. Below ~64K, shrinking the sequence (which also shrinks the FFN) beats sparsifying attention; past 300K the reverse.

**A capacity win can be an intensity loss.** [[persistent-state-dataflow-accelerator-linear-attention-decode-gupta-2026|Gupta et al.]] make this unusually clear for [[linear-attention]]: replacing a growing [[kv-cache]] with a fixed-size recurrent state is a memory-*capacity* win, but every layer must read and write its **entire** state matrix on every token regardless of sequence length. Short sequences therefore pay full state traffic for very little compute — low intensity, hard against the bandwidth wall. The fixed state is not free; it relocates the cost.

**Prefill cost tracks *active* parameters — on both sides of a speculative pair.** Field measurement on an M4 Air, cache-defeated: Qwen3.6-35B-A3B prefills at ~380–400 tok/s to ~12K, then declines to 125 tok/s at 35K. (A first version of this measurement showed the curve *climbing* to 749; it was the prefix cache, and is retracted on [[methods-to-speed-up-prompt-processing]].) The consequence for [[speculative-prefill]]: a dense 4B draft costs 0.74–1.61× the 3B-active target's prefill, so at keep 0.20 the pair yields 0.75–1.06× — a draft is only cheap relative to a target's *active* parameters, and a sparse target has few.

## Reading hardware claims with it

Intensity is also the tool for auditing accelerator results. An FPGA or ASIC that reports a large speedup on one memory-bound operation has usually improved *that operation's* effective bandwidth, which says nothing about end-to-end throughput if the rest of the model still runs elsewhere. See [[hardware-accelerator]] for the pattern and the specific cases in this wiki.

## Related

- [[llm-inference-roofline-yuan-2024]] — the per-sub-step measurement this page rests on.
- [[quantization-performance]] — the clearest worked consequence.
- [[sparse-prefill]] · [[sparse-attention]] · [[token-pruning]] — the context-dependent lever choice.
- [[flash-attention]] — raises intensity by keeping the N×N matrix out of HBM.
- [[kv-cache]] · [[linear-attention]] · [[grouped-query-attention]] — what moves bytes at decode.
- [[hardware-accelerator]] · [[methods-to-speed-up-prompt-processing]]

## References

- [[llm-inference-roofline-yuan-2024]] · [[quantization-performance]] · [[persistent-state-dataflow-accelerator-linear-attention-decode-gupta-2026]]

## Related 2026 sources

- [[weight-streaming]] — SSD as the tier below DRAM — 20–100× slower per byte than unified memory at sequential rating and 50–600× at the random-read granularity streaming actually uses, so viable only when the per-token working set is a small, *pre-known* fraction of the model
