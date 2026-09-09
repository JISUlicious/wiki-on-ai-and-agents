---
title: Hardware Accelerator
type: concept
created: 2026-09-09
updated: 2026-09-09
sources:
  - fast-prefill-fpga-sparse-attention-jayanth-2026.md
  - persistent-state-dataflow-accelerator-linear-attention-decode-gupta-2026.md
  - prefill-decode-aware-evaluation-ai-accelerators-usami-2026.md
  - hbm-is-not-all-you-need-wei-2026.md
status: complete
importance: medium
---

# Hardware Accelerator

Non-GPU silicon for LLM inference — FPGAs, and fixed-function parts from Groq, Tenstorrent and others. This wiki's coverage is thin and deliberately sceptical, because the field's headline numbers are unusually hard to read.

## Why prefill and decode land on different hardware

The organizing fact is [[arithmetic-intensity]]. Prefill is compute-bound at ~1,024–1,215 OPs/byte; decode sits at ~1. So they want opposite machines, and datacenter GPUs are a compromise that fits neither perfectly — [[hbm-is-not-all-you-need-wei-2026|HBM bandwidth sits nearly idle during prefill]] while being the binding constraint at decode.

That gap is the whole opportunity, and it produces two design responses:

- **Disaggregate across heterogeneous memory** — cheap GDDR parts for prefill, HBM for decode. The memory-tier analogue of [[distserve-zhong-2024|DistServe]]'s phase split across GPUs.
- **Build fixed-function dataflow for one bottleneck** — hold state on-chip and never round-trip through HBM.

## Read every accelerator claim for scope

Two patterns account for most inflated numbers in this literature, and both appear in the papers here.

> [!warning] "4.5× faster than an H100" can mean one layer
> [[persistent-state-dataflow-accelerator-linear-attention-decode-gupta-2026|Gupta et al.]] report **63 μs/token, 4.5× faster than an H100 PCIe, ~60× energy efficiency, 9.96 W** on an Alveo U55C. Genuine, and the mechanism is elegant — a Qwen3-Next-style Gated DeltaNet state is `32 × 128 × 128 × 4 B = 2 MB`, which fits in on-chip BRAM, so the state update never touches HBM.
>
> But the board holds **only that state, for a single layer**. No Q/K/V projections, no output projection, no FFN/MoE, no softmax-attention layers, no weights of any kind. [[qwen3.8-flash-next]] is 48 layers as 12 × (3 × Gated DeltaNet → 1 sparse attention). This is **a coprocessor for one operation, not an engine** — and the page notes plainly that a single H100 serves the whole model far faster end-to-end.

> [!warning] Check whether the host transfer is in the number
> [[fast-prefill-fpga-sparse-attention-jayanth-2026|FAST-Prefill]] (Alveo U280) reports up to 2.5× TTFT against an A5000. The paper **never names the host interconnect** — "PCIe" appears nowhere — and never quantifies DMA bandwidth or transfer latency, while stating that the host does tokenization and embedding lookup and DMAs embeddings across. So the FPGA figure is plausibly device-only against a host-observed GPU figure.
>
> It also states its headline **four mutually inconsistent ways** (abstract "up to 2.5×", body "1.5× to 2.x"), reports **no absolute latency or power anywhere**, and never compares against the BF16 GPU inference a practitioner would actually run.

## The Apple Silicon question

There is **no path to an FPGA accelerator on a Mac**, for three independent reasons, any one sufficient: Xilinx XRT's memory management lives in a Linux kernel driver with no macOS or Apple Silicon build; every accelerator above is a PCIe card and no Apple Silicon Mac exposes usable third-party PCIe; and Thunderbolt gives ~3–5 GB/s (TB4) or ~10–15 (TB5) against ~120 GB/s of unified memory.

The interconnect objection is decisive **for accelerators that do not hold the model**, because the link then carries traffic every step. It does *not* apply to an external GPU whose VRAM holds the whole model — there the link carries only prompts and tokens after a one-time load. As of 31 March 2026, Apple approved Tiny Corp's TinyGPU driver extension, making external Nvidia and AMD GPUs over Thunderbolt/USB4 the first officially sanctioned third-party discrete GPU path on Apple Silicon.

## Related

- [[arithmetic-intensity]] — the tool for auditing any accelerator claim.
- [[llm-inference-roofline-yuan-2024]] · [[quantization-performance]] — the roofline treatment.
- [[distserve-zhong-2024]] · [[splitwise-patel-2023]] — phase disaggregation, of which memory-tier disaggregation is the hardware analogue.
- [[sparse-prefill]] · [[sparse-attention]] · [[linear-attention]] · [[kv-cache]] — the software levers these papers implement in silicon.
- [[qwen3.8-flash-next]] · [[kimi-k3-moonshot-2026]] — the hybrid architectures the GDN accelerator targets.

## References

- [[fast-prefill-fpga-sparse-attention-jayanth-2026]] · [[persistent-state-dataflow-accelerator-linear-attention-decode-gupta-2026]] · [[prefill-decode-aware-evaluation-ai-accelerators-usami-2026]] · [[hbm-is-not-all-you-need-wei-2026]]
