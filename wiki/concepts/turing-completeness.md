---
title: Turing Completeness
type: concept
created: 2026-09-29
updated: 2026-09-29
sources:
  - universal-transformer-dehghani-2018.md
  - looped-transformers-programmable-computers-giannou-2023.md
  - cot-or-loop-xu-2025.md
status: draft
importance: low
---

# Turing Completeness

A model class is **Turing-complete** if, given suitable parameters and enough memory and steps, it can simulate any Turing machine. For neural sequence models the question is about **expressivity**, not learnability. The answer depends on the assumptions: numerical precision, whether depth or the number of steps can grow with the input, and whether the model can write intermediate output and read it back. A fixed-depth network does a bounded amount of computation per input. Adding [[recurrence]] in depth, or an autoregressive scratchpad, removes that bound.

## In this wiki

- **The Universal Transformer claim.** [[universal-transformer-dehghani-2018|Dehghani et al.]] argue that the UT is Turing-complete under mild assumptions. It can run for arbitrarily many steps and can simulate a Neural GPU, which a vanilla fixed-depth Transformer cannot. [[universal-transformer]] treats this as the first expressivity claim of its kind for the transformer family.
- **Looped transformers as computers.** [[looped-transformers-programmable-computers-giannou-2023|Giannou et al.]] give a constructive version. A 9-layer looped transformer runs a SUBLEQ one-instruction computer, and 13-layer versions implement a calculator and a linear-algebra library. The page contrasts this with Pérez et al.'s result, which needs high or infinite precision, and with bounded-precision results that go through chain-of-thought-style traces.
- **CoT vs loops.** [[cot-or-loop-xu-2025|Xu et al.]] cite that line when comparing token-recurrence ([[chain-of-thought]]) with depth-recurrence ([[looped-transformer]]).

## Related

- [[recurrence]] · [[universal-transformer]] · [[looped-transformer]] · [[recurrent-depth-transformer]] · [[adaptive-computation-time]] · [[expressive-power-looped-transformers-xu-2024]] · [[transformer-architecture]]

## References

- Pérez, Marinković & Barceló, "On the Turing Completeness of Modern Neural Network Architectures" (2019), [arXiv:1901.03429](https://arxiv.org/abs/1901.03429). Shows that the Transformer and the Neural GPU are Turing-complete without external memory.
- [[universal-transformer-dehghani-2018]] · [[looped-transformers-programmable-computers-giannou-2023]]
