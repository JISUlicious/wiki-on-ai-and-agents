---
title: "JAXBench: Benchmarking Autonomous TPU Kernel Optimization"
type: source
created: 2026-08-05
updated: 2026-08-05
sources:
  - jaxbench-tpu-kernel-optimization-tschand-2026.md
arxiv_id: "2607.20466"
year: 2026
authors:
  - Arya Tschand
  - Charles Hong
  - Amir Yazdanbakhsh
  - Vijay Janapa Reddi
  - Sethu Sankaran
tags:
  - 2026
  - paper
status: complete
importance: medium
---

# JAXBench: Benchmarking Autonomous TPU Kernel Optimization

**Arya Tschand, Charles Hong, Julian Walker, Nina Cai et al.** (Harvard University, UC Berkeley, Google DeepMind, Google) — [arXiv:2607.20466](https://arxiv.org/abs/2607.20466).

## Summary

Rigorous benchmarks like KernelBench and TritonBench have driven fast progress on autonomous *GPU* kernel optimization, but no equivalent existed for TPUs. JAXBench is the TPU-native analogue: 50 JAX workloads on Google Cloud TPU v6e (Trillium), with candidate kernels written in **Pallas**, JAX's low-level kernel DSL that lowers to the Mosaic backend. The suite comprises 17 production ML operators extracted from architectures in the public MaxText library (Llama-3.1, DeepSeek-V3, Mixtral, Mamba-2, AlphaFold2) plus 33 operators translated from KernelBench L2 and re-sized so they actually saturate the MXU (60–95% utilization, vs. the small shapes used by prior multi-platform work where measured "speedup" is really launch-overhead bookkeeping). Eight of the 17 production operators ship with hand-optimized Pallas kernels from the public Tokamax library, block-size tuned to serve as an expert upper bound.

TPUs differ from GPUs in both hardware and software stack — sequential machines with wide SIMD vector registers (8×128 for 32-bit on v6e) and 256×256 systolic MXUs, rather than SIMT; programs compile JAX → XLA, and kernel authoring goes through Pallas rather than Triton. Pallas kernels must reason about VMEM/SMEM/HBM placement, software pipelining and prefetch scheduling, block-shape constraints, and Mosaic's enforced lexicographic grid traversal. Critically, **Pallas appears in LLM training data orders of magnitude less often than CUDA or even Triton**, so models fluent at GPU kernels routinely hallucinate Pallas APIs or emit memory-space annotations that don't type-check.

The paper's central finding is a clean lesson about context: **on a sparsely-documented DSL, target-specific context matters more than model scale, and compiler feedback alone is nearly useless.** Injecting curated TPU documentation into an otherwise unchanged iterative loop raises per-sample correctness from 5.8% to 37.3% — a much larger jump than upgrading Gemini 3 Flash to Gemini 3.1 Pro at fixed context. Once correctness is solved, *search structure* is what converts correctness into speed.

## Key points

**Cross-method comparison, full 50-benchmark suite, Gemini 3 Flash, 144-sample budget** (speedups floored at 1× before aggregating; incorrect benchmarks count as 1×):

| Method | Geomean speedup | Mean speedup | Per-benchmark correctness | fast1@N |
|---|---|---|---|---|
| Best-of-N | 1.01× | 1.02× | 13/50 | beats XLA on 1 benchmark |
| Iterative refinement | 1.18× | 1.50× | 32/50 | 18% |
| Iterative + context | 1.28× | 1.55× | **48/50** | 32% |
| Autocomp (beam search) | **1.36×** | **1.70×** | 45/50 | **76%** |

**The context lesson, with numbers:**
- Adding Autocomp's curated context (hardware-architecture summary, per-benchmark Pallas API reference, annotated code examples, correctness rules) to the iterative loop — **with no change to the search algorithm** — raises per-sample correctness from **5.8% → 37.3%**.
- Compiler/runtime/profiler feedback alone is insufficient: plain iterative refinement gets only **16.9%** per-sample correctness with Gemini 3.1 Pro on the 5-kernel subset, and **1.2%** with Flash. The Pallas constraints that govern correctness — lexicographic grid traversal, VMEM/SMEM/HBM placement, (8,128) block divisibility, prefetch scheduling — are simply absent from error messages, so the feedback loop has nothing to iterate against.
- Pallas API misuse dominates context-free failure: **99.7%** of best-of-N and **93.8%** of iterative samples fail at compile or first execution by hallucinating Pallas APIs or misusing `pallas_call` arguments and `BlockSpec` types. Even with context it remains the leading failure mode at 59.8% (iterative+ctx) and 55.8% (Autocomp).
- Model capacity helps *less* than context. On the 5-kernel subset, Flash → Pro moves iterative 1.07× → 2.43×, iterative+context 1.59× → 3.82×, Autocomp 2.35× → 3.79×. The authors' framing: *"for Pallas and other languages underrepresented in training data, the correctness bottleneck is information rather than reasoning, and written documentation supplies most of what is missing."*
- **Search structure matters once correctness is solved.** Iterative+context lands *more* correct kernels than Autocomp (48/50 vs. 45/50) but a *lower* geomean (1.28× vs. 1.36×) — it spends its budget debugging until nearly every chain is correct and rarely pushes past the first correct kernel, while Autocomp commits 4 translation beam-search cycles up front and then spends the rest on optimization. With Pro (strong enough to produce correct seeds without extensive debugging) the two converge: 3.82× vs. 3.79×.

**Against hand-tuned Pallas (8 priority kernels, Gemini 3 Flash, 144 samples), speedup over XLA:**

| Kernel | Hand-tuned (Tokamax) | Autocomp | Iter+ctx | Iter |
|---|---|---|---|---|
| Flash Attention | 3.91× | 2.67× | 0.46× | — |
| GQA Attention | 3.50× | 2.63× | 2.63× | — |
| MLA Attention | 0.94× | 0.85× | 1.54× | — |
| Sparse Attention | 0.86× | **2.81×** | 2.54× | — |
| Paged Attention | 2.29× | — | 0.61× | — |
| Ragged Paged Attention | 6.91× | — | — | — |
| GEMM | 0.96× | 0.75× | 0.97× | 0.59× |
| Megablox GMM | 1.62× | **2.21×** | 0.27× | 0.52× |
| **Geomean (floor 1×)** | **2.08×** | **1.60×** | 1.34× | 1.00× |

Autocomp reaches roughly **77% of the hand-tuned geomean**, exceeding Tokamax on 2 kernels and landing within 68–91% on 4 more — but fails correctness entirely on paged and ragged paged attention, exactly where hand-written scheduling matters most.

- Concurrent work MultiKernelBench extends KernelBench to Pallas but achieves only **8.4–10.5% Pass@1** on Pallas tasks; it evaluates on TPU v2-8 at problem sizes too small to saturate the MXU and has no production LLM operators or optimized references.
- Workload sizing example: the GEMM workload runs at 8192 × 8192 × 28672 and achieves ~79% MXU utilization; compute-bound workloads like GEMM show no improvement from tuning while attention and elementwise kernels remain memory-bound.
- **Limitations:** all 50 workloads run on a *single* TPU v6e chip — multi-chip sharding and collectives are out of scope. Future work targets tensor/pipeline/expert-parallel regimes on multi-chip pods, where the primitives (`shard_map`, `psum_scatter`, `all_to_all`) are even more sparsely represented in pretraining data than Pallas itself.
- Benchmark, evaluation harness, and baseline results are released at `github.com/AI-Hypercomputer/accelerator-agents`.

## Concepts & entities

- [[agent-evaluation]] — JAXBench is an agent benchmark: it evaluates LLM-driven optimization *methods* (best-of-N, iterative loops, beam search), not just single completions.
- [[agentic-harness-engineering]] — the context-vs-search-structure separation is a harness-design result: identical model, identical task, method structure determines outcome.
- [[flash-attention]] — Flash Attention is one of the 8 hand-tuned reference kernels (3.91× hand-tuned vs. 2.67× for the best agent).
- [[humaneval]] — JAXBench's correctness-plus-speedup protocol descends from the pass@k tradition, extended with `fast1@N` for performance breadth.
- [[distributed-training]] — the named future direction: extending JAXBench to multi-chip TPU pods with tensor-, pipeline-, and expert-parallel sharding and collective scheduling.

## References

- [arXiv:2607.20466](https://arxiv.org/abs/2607.20466)
