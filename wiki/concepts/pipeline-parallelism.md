---
title: Pipeline Parallelism
type: concept
created: 2026-09-29
updated: 2026-09-29
sources:
  - gpipe-huang-2018.md
  - megatron-ptd-parallelism-narayanan-2021.md
  - megatron-lm-shoeybi-2019.md
  - reducing-activation-recomputation-korthikanti-2022.md
status: draft
importance: medium
---

# Pipeline Parallelism

**Pipeline parallelism** splits a model *between* layers. Each device, or **stage**, owns a consecutive block of whole layers, and activations pass from stage to stage. Each mini-batch is cut into micro-batches so that stages can work at the same time instead of waiting on one another. The idle time left at the start and end of each batch is the **pipeline bubble**. This is the complement of [[tensor-parallelism]], which splits the tensors *inside* each layer.

## In this wiki

- **Origin: GPipe.** [[gpipe-huang-2018|GPipe]] defines the paradigm as practiced today. Any network written as a sequence of layers is split into `K` cells, one per accelerator, and each mini-batch is split into `M` micro-batches with one synchronous update per mini-batch. GPipe pairs this with [[activation-recomputation|re-materialization]] and stores activations only at partition boundaries.
- **Bubble math and better schedules.** [[megatron-ptd-parallelism-narayanan-2021|PTD-P]] writes GPipe's bubble fraction as `(p−1)/m`. It replaces the schedule with 1F1B and an **interleaved 1F1B** variant that shrinks the bubble by a factor of `v`. Stages communicate point-to-point, which is cheap, so pipeline parallelism is used **across** nodes while tensor parallelism stays within a node.
- **Memory is not saved.** [[reducing-activation-recomputation-korthikanti-2022|Korthikanti et al.]] show that under 1F1B the first stage still stores activations for L layers regardless of `p`. Pipeline parallelism does not reduce activation memory.
- **Deliberately avoided.** [[megatron-lm-shoeybi-2019|Megatron-LM]] calls pipelining orthogonal and complementary, and avoids its bubbles by not using it.

## Related

- [[tensor-parallelism]] · [[distributed-training]] · [[activation-recomputation]]

## References

- [[gpipe-huang-2018]] · [[megatron-ptd-parallelism-narayanan-2021]] · [[reducing-activation-recomputation-korthikanti-2022]] · [[megatron-lm-shoeybi-2019]]
