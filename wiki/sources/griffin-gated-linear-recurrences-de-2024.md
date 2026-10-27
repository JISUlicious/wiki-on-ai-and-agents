---
title: "Griffin: Mixing Gated Linear Recurrences with Local Attention for Efficient Language Models"
type: source
created: 2026-10-27
updated: 2026-10-27
sources:
  - griffin-gated-linear-recurrences-de-2024.md
tags:
  - 2024
  - paper
status: complete
importance: medium
authors:
  - Soham De
  - Samuel L. Smith
  - Anushan Fernando
  - Aleksandar Botev
  - Albert Gu
  - Razvan Pascanu
  - Caglar Gulcehre
introduces:
  - [[griffin]]
year: 2024
venue: arXiv preprint
arxiv_id: "2402.19427"
---

# Griffin (and Hawk)

[arXiv:2402.19427](https://arxiv.org/abs/2402.19427), 2024-02-29, [[deepmind|Google DeepMind]] (17 authors).

## What it is

- **RG-LRU** (real-gated linear recurrent unit): a diagonal, elementwise recurrence with no complex numbers, HiPPO initialization or continuous-time discretization.

  ```
  r_t = σ(W_a x_t + b_a)          (recurrence gate)
  i_t = σ(W_x x_t + b_x)          (input gate)
  a_t = a^(c·r_t),  a = σ(Λ), c = 8
  h_t = a_t ⊙ h_{t−1} + √(1 − a_t²) ⊙ (i_t ⊙ x_t)
  ```

- **Hawk:** RG-LRU blocks interleaved with MLPs, i.e. pure recurrence.
- **[[griffin|Griffin]]:** a **2 : 1 mix of recurrent blocks and local [[multi-query-attention|MQA]]** with a **1,024-token window** ([[sliding-window-attention]]), so the attention cache is bounded too.
- **Training:** on TPU-v3 up to 14B parameters. Diagonal RNNs are memory-bound, so a custom Pallas linear-scan kernel is about 3× faster than JAX's scan.

## Results (all trained on 300B tokens)

| Model | Average (6 tasks) |
|---|---|
| MQA Transformer 1B / 3B / 6B | 54.4 / 59.0 / 64.2 |
| Hawk 1B / 3B / 7B | 53.6 / 59.5 / 63.8 |
| **Griffin 1B / 3B / 7B / 14B** | **57.2 / 60.7 / 65.8 / 69.5** |
| Llama-2-7B / 13B (2T tokens) | 65.3 / 69.3 |
| Mamba-3B (600B tokens) | 58.5 |

- **Tokens vs Llama-2:** Griffin-7B/14B roughly match Llama-2-7B/13B on **300B vs 2T tokens**. The paper says "over 6 times fewer" in some places and "roughly 7 times" in others; the actual ratio is 6.7×.
- **Hawk vs Mamba:** Hawk-3B beats Mamba-3B on half the tokens.
- **Scaling:** Griffin has the lowest loss at every FLOP budget in the scaling study.
- **Decode throughput vs MQA** (1B, read from the figure): Hawk is about 7–15× faster and Griffin about 3–7× faster at 2k–4k tokens. Griffin is slower than Hawk because its local-attention cache grows with batch size.
- **Retrieval:** Griffin is perfect on phonebook lookup up to the window size, then degrades. Hawk fails as length grows, which is the fixed-state recall limit.

## Limitations

- Pretrained models underperform Transformers on copying and exact retrieval without fine-tuning.
- Results are specific to TPUs.
- The paper itself doesn't mention a release; RecurrentGemma is a product of this line.

## Related

[[griffin]] · [[state-space-model]] · [[linear-attention]] · [[sliding-window-attention]] · [[mamba-selective-state-spaces-gu-2023]]
