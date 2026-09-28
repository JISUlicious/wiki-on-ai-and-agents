---
title: Tensor Parallelism
type: concept
created: 2026-09-29
updated: 2026-09-29
sources:
  - megatron-lm-shoeybi-2019.md
  - megatron-ptd-parallelism-narayanan-2021.md
  - gpipe-huang-2018.md
  - reducing-activation-recomputation-korthikanti-2022.md
status: draft
importance: medium
---

# Tensor Parallelism

**Tensor parallelism** (also called intra-layer model parallelism) splits the weight matrices *inside* each layer across devices. Every device holds a slice of every layer and computes its part of each GEMM, and collective communication stitches the partial results back together. It is the complement of [[pipeline-parallelism]], which splits the model *between* layers. Together with data parallelism, the two are the main axes of [[distributed-training]] for large transformers.

## In this wiki

- **Origin: Megatron-LM.** [[megatron-lm-shoeybi-2019|Megatron-LM]] gave the technique its standard form for the [[transformer-architecture]]. The MLP is split column-parallel and then row-parallel, and attention is split by head. This needs only a few all-reduces inserted into stock PyTorch. Its 8-way model-parallel runs reached 77% of linear scaling, and efficiency fell as the number of attention heads rose (16 heads 82%, 32 heads 77%).
- **Composition with the other axes.** [[megatron-ptd-parallelism-narayanan-2021|PTD-P]] reuses the Megatron split unchanged: 2 all-reduces forward and 2 backward per layer per microbatch. Because that communication is heavy, tensor parallelism is kept **within a node**, and pipeline parallelism goes **across** nodes.
- **What it leaves replicated.** [[reducing-activation-recomputation-korthikanti-2022|Korthikanti et al.]] measure the `10sbh` of activations that tensor parallelism does not shard. They add sequence parallelism to split those along the sequence dimension.
- **The gap GPipe left.** [[gpipe-huang-2018|GPipe]] assumes a single layer fits on one accelerator. Tensor parallelism is the axis that removes that assumption.

## Related

- [[pipeline-parallelism]] · [[distributed-training]] · [[activation-recomputation]] · [[transformer-architecture]]

## References

- [[megatron-lm-shoeybi-2019]] · [[megatron-ptd-parallelism-narayanan-2021]] · [[reducing-activation-recomputation-korthikanti-2022]] · [[gpipe-huang-2018]]
