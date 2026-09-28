---
title: Groq
type: entity
created: 2026-09-29
updated: 2026-09-29
sources:
  - prefill-decode-aware-evaluation-ai-accelerators-usami-2026.md
  - hbm-is-not-all-you-need-wei-2026.md
status: draft
importance: low
tags:
  - org
---

# Groq

AI-inference chip company that builds the **LPU**, an SRAM-based accelerator sold as GroqCard / GroqRack systems and through its own inference cloud. In this wiki it is the main example of the **SRAM-based** class of [[hardware-accelerator]]s: enormous on-chip bandwidth, very little capacity.

## In this wiki

- **[[hbm-is-not-all-you-need-wei-2026|HBM Is Not All You Need]]** (2026) sorts accelerators into three memory classes. The Groq LPU and Cerebras WSE-3 are **SRAM-based**: hundreds of TB/s over only tens of GB. That makes them strong at decode *if the model fits*, while a 70B model needs sharding that adds inter-chip cost.
- **[[prefill-decode-aware-evaluation-ai-accelerators-usami-2026|Prefill/Decode-Aware Evaluation]]** (2026) measured a GroqRack (72 GroqCards across 9 nodes, 15.42 GiB SRAM) against one A100 on Llama2-7B FP16 ([[llama]]). Findings:
  - **Decode:** Groq wins by about 4× on per-token latency, and TPOT is nearly constant with sequence length. That part is architectural: every weight and the [[kv-cache]] sit in explicitly managed SRAM on a static schedule.
  - **Prefill:** Groq loses by up to 39×. That part is an implementation artifact. The LPU has no branch instructions, and the supplied binary runs one token per invocation at batch 1, so prefill runs like decode. [[chunked-prefill]] is the obvious missing lever.
  - **Split serving:** prefill on the GPU and decode on Groq wins unless the input is roughly 50× longer than the output.

The paper reads Groq's design as the same answer as [[persistent-state-dataflow-accelerator-linear-attention-decode-gupta-2026|persistent on-chip state]] at a different scale: keep decode's working set out of the memory hierarchy.

## Related

- [[hardware-accelerator]] · [[kv-cache]] · [[chunked-prefill]] · [[arithmetic-intensity]] · [[nvidia]] · [[fpga]]

## References

- [[prefill-decode-aware-evaluation-ai-accelerators-usami-2026]] · [[hbm-is-not-all-you-need-wei-2026]]
- [groq.com/about-us](https://groq.com/about-us/) — company self-description (LPU, inference cloud)
