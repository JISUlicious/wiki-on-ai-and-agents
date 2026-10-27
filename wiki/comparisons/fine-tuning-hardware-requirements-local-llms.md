---
title: "Hardware requirements for training and fine-tuning current local LLMs (2026)"
type: comparison
created: 2026-10-27
updated: 2026-10-27
sources:
  - unsloth-qwen3.5-fine-tune-guide-2026.md
  - unsloth-gemma-4-fine-tune-guide-2026.md
  - llama-factory-hardware-requirement-2026-10-27.md
  - mlx-lm-lora-docs-2026-10-27.md
  - mlx-lora-finetune-qwen3.5-sciences44-2026.md
  - dgx-spark-bf16-lora-qwen3.5-35b-a3b-forum-2026.md
status: complete
importance: high
---

# Hardware for training and fine-tuning current local LLMs (2026)

What memory you need to fine-tune today's open local models (the Qwen3.5 and Gemma 4 families) on one machine, and why.

**Theory:** [[training-memory-anatomy]].
**Sources:** [[unsloth-qwen3.5-fine-tune-guide-2026]], [[unsloth-gemma-4-fine-tune-guide-2026]], [[llama-factory-hardware-requirement-2026]], [[mlx-lm-lora-docs-2026]], [[dgx-spark-bf16-lora-qwen3.5-35b-a3b-forum-2026]].

## 1. The rule of thumb (P = billions of parameters)

| Method | Memory ≈ | Why |
|---|---|---|
| Full fine-tune, mixed-precision Adam | **16–18 × P GB** | weights + gradients + fp32 master + 2 Adam moments ([[training-memory-anatomy\|16 bytes/param]]) + activations |
| Full fine-tune, pure bf16 | **~8 × P GB** | no fp32 master; bf16 moments |
| [[lora\|LoRA]] (bf16 base) | **~2 × P GB + activations** | the frozen base has no gradients and no optimizer state |
| [[qlora-dettmers-2023\|QLoRA]] 4-bit | **~0.5 × P GB + activations** | NF4 base; adapters in bf16 |

The first, third and fourth rows match LLaMA-Factory's estimates (18x / 2x / x/2). Activations grow with sequence length and batch size, and dominate at long context. The levers are gradient checkpointing, FlashAttention, batch 1 with gradient accumulation, and fewer LoRA layers.

## 2. Current models (vendor figures, short context)

| Model | Type | bf16 LoRA | QLoRA (4-bit) | Full FT |
|---|---|---|---|---|
| Qwen3.5-0.8B / 2B / 4B | dense | 3 / 5 / 10 GB | *not recommended for Qwen3.5* | ~4× LoRA |
| Qwen3.5-9B | dense | 22 GB | not recommended | ~4× LoRA |
| Qwen3.5-27B | dense | 56 GB | not recommended | ~4× LoRA |
| **Qwen3.5-35B-A3B** | MoE | **74 GB** | not recommended (bitsandbytes MoE limits) | — |
| Qwen3.5-122B-A10B | MoE | 256 GB (multi-GPU) | not recommended | — |
| Gemma 4 E2B / E4B | dense, small | 8–10 / 17 GB | E4B QLoRA recommended over E2B LoRA | — |
| Gemma 4 31B | dense | — | **22 GB** | — |
| **Gemma 4 26B-A4B** | MoE | **> 40 GB** | — | avoid; use LoRA |

> [!warning] MoE: fine-tuning memory follows total parameters, not active ones
> Qwen3.5-35B-A3B activates 3B parameters per token, but **all 35B weights must be resident** for training: 67 GB in bf16, 74 GB for LoRA. "3B active" says nothing about training memory. The same holds for inference KV and weights ([[kv-cache-size-vs-active-parameters]], [[mixture-of-experts]]).

## 3. By hardware tier (what fits, practically)

| Hardware | Memory | Comfortable | Possible with care | Out of reach |
|---|---|---|---|---|
| 16 GB Mac ([[mlx]]) | 16 GB unified | QLoRA/LoRA ≤ 2–4B (batch 1, 4 layers) | 4-bit 7–12B QLoRA (8.2 GB measured for 12B) | bf16 LoRA ≥ 9B |
| 24 GB GPU (RTX 4090/3090) | 24 GB | bf16 LoRA ≤ 9B (22 GB, tight); Gemma 4 31B QLoRA (22 GB) | — | 27B+ bf16 LoRA; MoE LoRA |
| 32 GB GPU (RTX 5090) | 32 GB | 9B bf16 LoRA with context headroom | 4-bit 35B-A3B (17.5 GB claimed; **unverified**, against Unsloth's advice) | bf16 LoRA of 35B-A3B (74 GB) |
| 32–64 GB Mac | unified | LoRA/QLoRA ≤ 9B; 4-bit QLoRA of larger dense models | — | bf16 MoE LoRA |
| [[dgx-spark\|DGX Spark]] / 128 GB Mac | ~119–128 GB unified | **bf16 LoRA of 35B-A3B (74 GB)**, 27B (56 GB), Gemma 4 26B-A4B | — | 122B-A10B (256 GB) |
| 80 GB GPU (H100/A100) | 80 GB | 27B bf16 LoRA; 35B-A3B LoRA (74 GB, tight) | full FT ≤ ~4B (16–18 × P) | full FT of 9B+ on one card |
| Multi-GPU (8×80 GB) | 640 GB | 122B-A10B LoRA; full FT of 27–35B with [[zero-redundancy-optimizer\|ZeRO]] / [[pytorch-fsdp-zhao-2023\|FSDP]] | | |

**Unified-memory gotcha:** on DGX Spark (and likely Macs), Hugging Face loading can need **twice the model size**: the page cache of memory-mapped shards plus the materialized tensors. That OOMs a 67 GB model on 119 GB until shards are loaded one at a time. Plan for the loading peak, not just the training steady state ([[dgx-spark-bf16-lora-qwen3.5-35b-a3b-forum-2026]]).

## 3a. LoRA on MoE: what scales with total vs active parameters

| Cost | Scales with | Why |
|---|---|---|
| Frozen base weights (most of the memory) | **total** parameters | every expert must be resident; the router can send any token anywhere |
| Activations | **active** path × tokens | only routed experts run per token |
| Compute per token | **active** parameters | forward and backward go only through the selected experts |
| Adapter size and its optimizer state | **which modules are targeted** | attention-only adapters are small whatever the expert count; adapting every expert's projections scales with the number of experts |

```
memory  ≈ b_w·P_total + 16 B·P_adapter + activations(active path, tokens)
compute ≈ 4·P_active·tokens   (forward 2·P_active + backward ~2·P_active; no base weight gradients)
```

**Example:** Qwen3.5-35B-A3B needs **big-model memory** (74 GB for bf16 LoRA) but has **small-model compute** (≈ 3B per token).

**Adapting expert layers** also makes adapters large and their fused-expert layout tool-specific, which is behind the reported vLLM loading failures.

## 3b. Capacity vs throughput: DGX Spark vs an 80 GB H100

| | [[dgx-spark\|DGX Spark]] (GB10) | H100 SXM 80 GB |
|---|---|---|
| Memory | **128 GB** unified (about 119 GB usable) | 80 GB HBM3 |
| Bandwidth | about **273 GB/s** LPDDR5x (Hot Chips deck: about 301 raw) | about 3.35 TB/s (datasheet; not re-verified here) |
| Compute | about 1 PFLOP sparse / ~500 TFLOPS dense FP4 | about 990 TFLOPS dense BF16 (datasheet; not re-verified here) |
| Ratio | — | about **12×** bandwidth; roughly **8×** BF16 compute (estimate assuming Spark BF16 ≈ ¼ of FP4) |

**Reading the comparison:**
- **The Spark wins on what fits.** A 74 GB MoE LoRA fits with headroom; on an 80 GB H100 it is tight.
- **The H100 wins on how fast.** Both MoE training (active-path compute and weight streaming) and decoding (bandwidth-bound) run many times faster on it.
- **Community verdict:** the Spark is "built for inference and large-memory work" and weak for high-throughput BF16 training. One guide reports a 17-hour 14B run that ran out of memory at 70%; it comes from an H100-rental vendor, so weigh it accordingly.
- **Typical pattern:** prototype on a Spark or large Mac, then run production fine-tunes on rented H100s.
- **No head-to-head fine-tuning benchmark** on an identical setup was found, so treat the multipliers as order-of-magnitude.

**Sources for 3b:**
- [IntuitionLabs review](https://intuitionlabs.ai/articles/nvidia-dgx-spark-review)
- [Hot Chips 2025 GB10 deck](https://www.hc2025.hotchips.org/assets/program/conference/day2/21_nvidia_skende_final.pdf)
- [Spark-DGX-Benchmark](https://github.com/rossingram/Spark-DGX-Benchmark)
- [Spheron guide](https://www.spheron.network/blog/dgx-spark-fine-tuning-what-fits-in-128gb-unified-memory-2026/)

## 4. Training from scratch vs fine-tuning

Pretraining even a small model is a different scale. [[nanochat]]'s README pitches **GPT-2-level capability for about $100 in roughly two hours on an 8×H100 node**. Secondary coverage of the original release quotes about 4 hours for the $100 tier, and about 8× longer on a single GPU with a smaller batch. Locally, practical "training" means **LoRA/QLoRA fine-tuning or small-model pretraining**. Full fine-tuning of anything above ~4B already needs multi-GPU or offload ([[zero-offload-ren-2021|ZeRO-Offload]], [[8-bit-optimizers-dettmers-2021|8-bit Adam]]).

## 5. Practical guidance

1. **Start from the method row,** then add activation headroom: 20–30% at short context, more for long context or batch > 1.
2. **Prefer bf16 LoRA on Qwen3.5.** Unsloth warns against 4-bit for it; QLoRA is fine on Gemma 4 E4B and 31B per Unsloth. The general QLoRA result ([[qlora-dettmers-2023|NF4 recovers 16-bit quality]]) doesn't hold uniformly per model family. Evaluate.
3. **For MoE, budget total parameters** and check the adapter-export path to your server. Fused-expert LoRA layouts differ between stacks, and vLLM loading failures have been reported.
4. **Smaller can win.** In one MLX run, Qwen3.5-2B beat 4B and 12B on a narrow task after fine-tuning. Try the small model first.
5. **Vendor numbers are best cases** (Unsloth's kernels, gradient checkpointing, short context). Plain Hugging Face/PEFT needs more; Unsloth itself claims 50–60% less VRAM than FlashAttention-2 baselines.

## Gaps

- **No independent benchmark** compares the tools (Unsloth, LLaMA-Factory, Axolotl, torchtune, MLX) under one fixed setup.
- **No official MLX figures** for M5-class Macs, and no measured LoRA numbers for 27B+ on Macs.
- **RTX 5090 MoE QLoRA claims** are unverified.

## References

[[unsloth-qwen3.5-fine-tune-guide-2026]] · [[unsloth-gemma-4-fine-tune-guide-2026]] · [[llama-factory-hardware-requirement-2026]] · [[mlx-lm-lora-docs-2026]] · [[dgx-spark-bf16-lora-qwen3.5-35b-a3b-forum-2026]] · [[training-memory-anatomy]] · [[lora]] · [[qlora-dettmers-2023]] · [[fine-tuning]] · [[unsloth]] · [[llama-factory]]
