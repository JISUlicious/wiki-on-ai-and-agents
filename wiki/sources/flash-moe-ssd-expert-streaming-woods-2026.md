---
title: "flash-moe: SSD Expert Streaming for a 397B MoE on a 48 GB MacBook"
type: source
created: 2026-09-10
updated: 2026-09-10
sources: []
authors:
  - Dan Woods
year: 2026
venue: "GitHub danveloper/flash-moe; written up by Simon Willison, 2026-03-18"
status: complete
importance: medium
tags:
  - 2026
---

# flash-moe: SSD Expert Streaming for a 397B MoE on a 48 GB MacBook

**Repository**: [danveloper/flash-moe](https://github.com/danveloper/flash-moe) — *"Running a big model on a small laptop."* Pure C / Objective-C / Metal, no Python and no framework. Created **2026-03-18**, last pushed **2026-03-19**, 4,090 stars at ingest. Written up the same day by [Simon Willison](https://simonwillison.net/2026/Mar/18/llm-in-a-flash/). The most-cited fork is [Anemll/flash-moe](https://github.com/Anemll/flash-moe) (222 stars, active to 2026-03-24), which added the M5 Max numbers below.

This is the artifact that turned Apple's 2023 [[llm-in-a-flash-alizadeh-2023|LLM in a flash]] from a paper into a Mac practice. It is not itself an [[mlx]] project, but every [[mlx]]-family expert streamer that followed cites it — see the survey on [[weight-streaming]].

## What it does

Runs **Qwen3.5-397B-A17B** — a 397-billion-parameter [[mixture-of-experts]] model with 17B active per token — on a **48 GB M3 Max MacBook Pro**, a machine that cannot hold a tenth of it.

The split is the one the 2023 paper prescribes, applied to MoE routing instead of ReLU sparsity:

- **Resident (~5.5 GB)**: embeddings, attention, router matrices and other shared weights, at original precision.
- **On NVMe (209 GB at 4-bit, 120 GB at 2-bit)**: the routed experts. Each 4-bit expert is ~6.75 MB.
- **Per token**: the router's chosen experts for each layer are read on demand with parallel `pread()` calls under GCD dispatch groups. Only those experts are dequantized and multiplied.

Two implementation details the author highlights: experts per token were **cut from Qwen's 10 to 4** to shrink the per-token read, and the dequantization kernel was rearranged from `(nibble × scale + bias) × x` to `fma(nibble, scale·x, bias·x)`, which the README reports as 12% faster.

## Measured throughput (README self-reports)

| Machine | Expert format | Decode | Source |
|---|---|---|---|
| M3 Max, 48 GB | 2-bit (initial) | 5.5+ tok/s | Willison write-up |
| M3 Max, 48 GB | 4-bit | **4.36 tok/s** | original repo |
| M5 Max, 128 GB | 4-bit MLX experts | 9.5 tok/s | Anemll fork |
| M5 Max, 128 GB | Q3 GGUF + `cache-io-split 4` (recommended) | 12.9 tok/s | Anemll fork |
| M5 Max, 128 GB | 2-bit MLX experts | **14.5 tok/s** | Anemll fork |

The M5 Max runs the same code **3.3× faster** than the M3 Max: a faster SSD plus 80 GB more RAM for the expert cache. That ratio is the clearest available evidence that the bottleneck is the expert cache hit rate, not compute.

## How it was built

Woods used [[claude-code]] in an *autoresearch* loop — the model proposes an experiment, runs it, reads the result, proposes the next — for roughly **90 experiments in about 24 hours**, generating the Objective-C and Metal itself. The 2-bit version came first; when it broke tool calling, the loop moved to 4-bit experts.

> [!warning] What is not measured
> - **No SSD bandwidth figure** anywhere, so the 4.36 tok/s cannot be decomposed into read time and compute time.
> - **The top-k cut from 10 to 4 is a model change, not an engineering choice**, and no accuracy delta is reported for it. Whatever quality the model has at k=4 is not the quality Qwen shipped.
> - Quality evaluation is thin. Willison notes the claim that 2-bit was "indistinguishable from 4-bit" came from the agent and was not backed by a described evaluation; the 2-bit build then failed at tool calling.
> - **No baseline**: no comparison against [[llama-cpp]] with `mmap`, or against [[mlx]]-lm on a machine that fits the model.
> - **Prompt processing speed is not reported.** Prefill touches nearly every expert in every layer, so it is the phase SSD streaming hurts most; see [[weight-streaming]].

## What it set off

Within 48 hours: `matt-k-wong/mlx-flash` (2026-03-20, dense-model streaming for MLX) and the Anemll fork (2026-03-20). Then `jundot/omlx` issue #986 (2026-04-28), titled *"Add Flash-MoE-style SSD-backed expert streaming"*, and `ml-explore/mlx-lm` issue #1438 (2026-06-27). The full tree, with each project's own numbers, is on [[weight-streaming]]. As of 2026-09-10 none of it has been merged upstream into MLX or mlx-lm.

## Related

- [[weight-streaming]] — the concept, the three sparsity sources, and the implementation survey.
- [[llm-in-a-flash-alizadeh-2023]] — the paper this applies.
- [[mixture-of-experts]] — why routing gives exact sparsity for free.
- [[omlx-specprefill-github-2026]] — the server whose expert-streaming PR is still open.
- [[arithmetic-intensity]] · [[quantization-performance]] · [[mlx]] · [[claude-code]]

## References

- [danveloper/flash-moe](https://github.com/danveloper/flash-moe) · [Anemll/flash-moe](https://github.com/Anemll/flash-moe) · [Simon Willison, "Autoresearching Apple's LLM in a Flash to run Qwen 397B locally"](https://simonwillison.net/2026/Mar/18/llm-in-a-flash/)
- [[llm-in-a-flash-alizadeh-2023]]
