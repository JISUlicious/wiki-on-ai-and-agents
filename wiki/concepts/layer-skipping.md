---
title: Layer Skipping
type: concept
created: 2026-09-29
updated: 2026-09-29
sources:
  - pop-prefill-only-pruning-he-2026.md
  - fast-forward-predictive-ffn-sparsity-prefill-gautam-2026.md
status: draft
importance: low
---

# Layer Skipping

**Layer skipping** saves inference compute by not running some of a model's layers, or some of their sub-blocks, on some inputs or in some phases. It sits between two neighbours. **Structured layer pruning** removes the same layers permanently, for every input and every phase. **Adaptive depth** decides per input how many steps to run (see [[adaptive-computation-time]] and [[recurrent-depth-transformer]]).

## In this wiki

- **Skipping by phase.** [[pop-prefill-only-pruning-he-2026|POP]] skips roughly the last third of layers **only during prefill** and keeps the full model for decode. It rests on a measured asymmetry: deep layers matter much more to decode than to prefill. It gets about 1.36× cheaper prefill on 8B–12B dense models. The stage-agnostic baseline is ShortGPT, which removes 25% of layers in both phases and collapses on several tasks; POP largely does not. For short prompts the gain is small, because weight reads dominate there.
- **Skipping sub-blocks in decode.** [[fast-forward-predictive-ffn-sparsity-prefill-gautam-2026|FastForward]] cites FFN-SkipLLM and related "layer skipping" work as the decode-side relatives of its prefill FFN sparsity, and sets them aside.
- **Skipping tokens instead of layers.** [[probe-and-skip-self-predictive-token-skipping-wu-2026|Probe-and-Skip]] and [[token-pruning]] skip *tokens* within a layer rather than whole layers.

## Related

- [[token-pruning]] · [[sparse-prefill]] · [[adaptive-computation-time]] · [[recurrent-depth-transformer]] · [[activation-sparsity]]

## References

- [[pop-prefill-only-pruning-he-2026]] · [[fast-forward-predictive-ffn-sparsity-prefill-gautam-2026]]
