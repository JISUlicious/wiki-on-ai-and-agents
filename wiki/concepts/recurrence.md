---
title: Recurrence
type: concept
created: 2026-09-29
updated: 2026-09-30
sources:
  - adaptive-computation-time-graves-2016.md
  - universal-transformer-dehghani-2018.md
  - recirculation-mozer-2026.md
  - long-short-term-memory-hochreiter-1997.md
status: draft
importance: medium
---

# Recurrence

**Recurrence** is the structural pattern of applying **the same computation, with the same weights, repeatedly** and feeding each step's output state into the next step. The wiki uses the word in two senses:

1. **Recurrence over time or sequence.** This is the classical RNN sense: one cell is applied once per input position and carries a hidden state forward. Here "recurrence" is shorthand for the [[recurrent-neural-network]] family, including the [[lstm]] and, in state form, [[state-space-model|state-space models]] and [[linear-attention]].
2. **Recurrence over depth.** One block, or a stack of blocks, is re-applied to the same representation several times, so depth becomes a loop count instead of a count of distinct layers. This is the sense behind [[universal-transformer]], [[looped-transformer]] and [[recurrent-depth-transformer]], and it is closely tied to [[parameter-sharing]].

## In this wiki

- **Controlling how many steps.** [[adaptive-computation-time-graves-2016|ACT]] was first attached to RNNs and learns how many times to revisit the same weights for each input. [[adaptive-computation-time]] is that halting mechanism.
- **Recurring in depth instead of time.** [[universal-transformer-dehghani-2018|The Universal Transformer]] recurs over depth, revising every position's representation in parallel. It sets this against time-recurrent RNNs, which lack random-access memory in the recurrent step. The ability to run for an input-dependent number of steps is what underlies its [[turing-completeness]] claim.
- **Where the unrolling happens.** [[recirculation-mozer-2026|Recirculation]] feeds late-layer residuals back to early layers. It argues that looping is recurrence in depth *only*, while recirculation unrolls differently, and that with unbounded iterations recirculation becomes an RNN.
- **Reasoning.** [[latent-reasoning]] and [[inference-time-compute]] treat depth-recurrence as the alternative to token-recurrence, i.e. [[chain-of-thought]].

## Related

- [[recurrent-neural-network]] · [[lstm]] · [[universal-transformer]] · [[looped-transformer]] · [[recurrent-depth-transformer]] · [[adaptive-computation-time]] · [[parameter-sharing]] · [[turing-completeness]]

## References

- [[adaptive-computation-time-graves-2016]] · [[universal-transformer-dehghani-2018]] · [[recirculation-mozer-2026]] · [[long-short-term-memory-hochreiter-1997]]

## Related 2026 sources

- [[thinking-with-looped-flows-suleymanzade-2026]] — Looped flows: train looped reasoners with flow-matching losses; ARC-AGI-1 58.8 pass@2 at 5–7M parameters
- [[recurrent-looped-transformer-zhang-2026]] — Recurrent Looped Transformer: decoder state carried across all tokens; a design report with no experiments (alphaXiv-only)
