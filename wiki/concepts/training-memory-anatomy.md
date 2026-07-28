---
title: Training Memory Anatomy
type: concept
created: 2026-07-29
updated: 2026-07-29
sources:
  - zero-memory-optimization-rajbhandari-2019.md
  - reducing-activation-recomputation-korthikanti-2022.md
  - 8-bit-optimizers-dettmers-2021.md
  - mixed-precision-training-micikevicius-2017.md
status: complete
importance: high
---

# Training Memory Anatomy

Where LLM training memory actually goes. The headline: **the model weights are the smallest of the four consumers.** Everything expensive is machinery *around* the weights — optimizer state and stored activations.

## The two halves

[[zero-memory-optimization-rajbhandari-2019|ZeRO]] splits training memory into **model states** (parameters, gradients, optimizer states) and **residual states** (activations, temporary buffers, fragmentation). They have different scaling behaviour and different fixes.

## Model states: the 16Ψ accounting

For mixed-precision Adam on Ψ parameters, ZeRO's per-parameter breakdown:

| Component | Bytes/param |
|---|---|
| fp16 parameters | 2Ψ |
| fp16 gradients | 2Ψ |
| fp32 master copy | 4Ψ |
| Adam momentum | 4Ψ |
| Adam variance | 4Ψ |
| **Total** | **16Ψ** |

**Optimizer states are 12 of 16 bytes — 75% of model-state memory.** In the paper's words: *"the optimizer states usually consume the most memory, specially when mixed-precision training is applied."* Concretely, GPT-2 1.5B needs **≥24 GB** of model states against just **3 GB** for the fp16 parameters.

> [!warning] Two different "optimizer state" numbers in the literature
> [[8-bit-optimizers-dettmers-2021|Dettmers]] counts Adam at **8 bytes/param** — momentum and variance only. ZeRO counts **12** — adding the fp32 master parameter copy. Both are internally consistent; they draw the boundary in different places. Always check whether a quoted figure includes the master copy.
>
> Dettmers' framing figure is the useful one for intuition: **"optimizer states use 33–75% of the total memory footprint during training."**

## Residual states: activations, and why they dominate at long context

The exact per-layer formula, from [[reducing-activation-recomputation-korthikanti-2022|Korthikanti et al.]] (16-bit storage, s = sequence length, b = microbatch, h = hidden size, a = heads):

$$\text{Activations per layer} = sbh\left(34 + \frac{5as}{h}\right)$$

Decomposed: attention block `11sbh + 5as²b`, MLP `19sbh`, two layer-norms `4sbh`.

**The `34sbh` term is linear in sequence length; the `5as²b` term is quadratic.** That single fact explains why activations become the dominant consumer as context grows. Made concrete:

| Model | `5as/h` | vs 34 for everything else |
|---|---|---|
| GPT-3 (a=96, s=2048, h=12288) | **80** | attention ≈ **70%** of activation memory |
| MT-NLG (a=128, s=2048, h=20480) | **64** | attention ≈ **65%** |

Scale matters: GPT-2 1.5B stores **~60 GB** of activations, dropping to ~8 GB with checkpointing at 33% recompute overhead — and a 100B model still needs **~60 GB** of activations at batch 32 *even with* checkpointing.

Everything outside the transformer layers — embedding dropout, final layer-norm, output projection, fp32 logits — is **<0.01%** of activation memory at 22B. Don't optimize it.

## Why mixed precision doesn't halve memory

[[mixed-precision-training|Mixed precision]] is widely assumed to halve training memory. For *weights* it does the opposite:

> *"maintaining an additional copy of weights increases the memory requirements for the weights by 50% compared with single precision training."*

Per parameter that's 4 B (fp32 master) + 2 B (fp16 working) = **6 B, versus 4 B for pure fp32**. Total memory still falls, and the paper's own explanation is exactly this page's thesis:

> *"For training memory consumption is dominated by activations, due to larger batch sizes and activations of each layer being saved for reuse in the back-propagation pass."*

## The levers, matched to the consumer

| Consumer | Lever | Effect |
|---|---|---|
| **Optimizer states** | [[zero-redundancy-optimizer\|ZeRO]] sharding | 4× / 8× / N_d× at stages 1/2/3 |
| | [[8-bit-optimizers-dettmers-2021\|8-bit optimizers]] | 4× on optimizer state, quality-neutral |
| | CPU offload ([[zero-offload-ren-2021\|ZeRO-Offload]]) | 8× model states; 13B on one 32 GB V100 |
| **Activations** | [[activation-recomputation]] | O(√n) memory for ~30% extra compute |
| | selective recomputation | ~5× activation memory, only 2–4% overhead |
| | sequence parallelism | partitions the `10sbh` that tensor parallelism leaves replicated |
| **Parameters** | [[distributed-training\|tensor / pipeline parallelism]] | shards weights within or across layers |
| | [[lora\|LoRA]] / [[qlora-dettmers-2023\|QLoRA]] | freezing the base eliminates *its* gradients and optimizer states entirely |

The LoRA row is worth dwelling on: parameter-efficient finetuning is powerful less because 4-bit weights are small than because **a frozen base has no gradients and no optimizer states** — it deletes the 12Ψ term for most parameters. That is how [[qlora-dettmers-2023|QLoRA]] fits 65B on a single 48 GB GPU.

## Practical reading

1. **Don't start with the weights.** They are ~2 of 16 bytes/param.
2. **Below ~4k context, optimizer state dominates** → shard it (ZeRO/FSDP) or quantize it (8-bit).
3. **At long context, activations dominate** via the quadratic `5as²b` term → selective recomputation, sequence parallelism, [[flash-attention]].
4. **[[scalable-moe-training-megatron-core-yan-2026|Megatron Core]] cutting DeepSeek-V3 from 199.5 GB → under 80 GB/GPU** used exactly this playbook: fine-grained recompute, BF16-moment optimizers, CPU activation offload. All three target activations and optimizer moments; none touch weights.

## Related

- [[zero-redundancy-optimizer]] · [[activation-recomputation]] · [[mixed-precision-training]] · [[distributed-training]]
- [[backpropagation]] — why activations must be stored at all.
- [[quantization-performance]] — the inference-side counterpart (weights vs [[kv-cache]]).

## References

- [[zero-memory-optimization-rajbhandari-2019]] · [[reducing-activation-recomputation-korthikanti-2022]] · [[8-bit-optimizers-dettmers-2021]] · [[mixed-precision-training-micikevicius-2017]]
- [[llm-memory-and-compute-bottlenecks]] — the four-quadrant synthesis this page anchors
