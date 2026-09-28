---
title: FPGA
type: concept
created: 2026-09-29
updated: 2026-09-29
sources:
  - fast-prefill-fpga-sparse-attention-jayanth-2026.md
  - persistent-state-dataflow-accelerator-linear-attention-decode-gupta-2026.md
  - prefill-decode-aware-evaluation-ai-accelerators-usami-2026.md
  - neuroprefetcher-delta-prefetching-dhar-2026.md
status: draft
importance: low
---

# FPGA

A **field-programmable gate array (FPGA)** is a chip whose logic, on-chip memory (BRAM/URAM) and DSP blocks can be rewired after manufacture. For LLM inference this means much less peak arithmetic than a GPU, but a **memory hierarchy the designer can program**. That trade only pays off when a kernel is limited by memory bandwidth rather than compute (see [[arithmetic-intensity]]). The wider class of non-GPU inference silicon, and the wiki's sceptical reading of its headline numbers, is covered in [[hardware-accelerator]].

## In this wiki

The wiki's two FPGA sources both come from Viktor Prasanna's USC group. Each attacks a memory-bound operation.

- **Prefill.** [[fast-prefill-fpga-sparse-attention-jayanth-2026|FAST-Prefill]] (Alveo U280) puts FlexPrefill-style dynamic [[sparse-prefill|sparse attention]] into hardware. Its argument is that dynamic sparsity makes prefill memory-bound, so a 5.4-TOPS FPGA can beat a 222-TOPS GPU. The paper states its speedup in several ways, and it never names the host interconnect.
- **Decode.** [[persistent-state-dataflow-accelerator-linear-attention-decode-gupta-2026|Gupta et al.]] (Alveo U55C) pin the 2 MB recurrent state of one Gated DeltaNet layer in BRAM. They report about 4.5× over an H100 for that single operation. The board holds no model weights, so this is a coprocessor, not an inference engine.
- **The same answer at different scales.** [[prefill-decode-aware-evaluation-ai-accelerators-usami-2026|Usami et al.]] compare Groq putting everything in SRAM with the FPGA pinning the state in BRAM. [[neuroprefetcher-delta-prefetching-dhar-2026|NeuroPrefetcher]] contrasts both with a software approach on Jetson.
- **Not a Mac path.** [[hardware-accelerator]] and [[edge-inference]] record why an FPGA accelerator cannot be used from Apple Silicon today: no XRT driver, no usable PCIe, and too little Thunderbolt bandwidth.

## Related

- [[hardware-accelerator]] · [[arithmetic-intensity]] · [[edge-inference]] · [[sparse-prefill]] · [[linear-attention]] · [[groq]] · [[apple-silicon]]

## References

- [[fast-prefill-fpga-sparse-attention-jayanth-2026]] · [[persistent-state-dataflow-accelerator-linear-attention-decode-gupta-2026]]
