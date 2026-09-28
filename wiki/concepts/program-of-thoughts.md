---
title: Program of Thoughts (PoT)
type: concept
created: 2026-09-29
updated: 2026-09-29
sources:
  - code-as-agent-harness-ning-2026.md
  - code-as-policies-liang-2022.md
  - pal-gao-2023.md
status: draft
importance: medium
---

# Program of Thoughts (PoT)

**Program of Thoughts prompting** (Chen, Ma, Wang & Cohen, 2022) separates *computation* from *reasoning*. Instead of carrying out arithmetic inside a natural-language [[chain-of-thought]], the model writes its reasoning as a program, and an external interpreter runs that program to produce the answer. The paper tests PoT on five math-word-problem datasets and three financial-QA datasets, reporting an average gain of about 12% over CoT in both few-shot and zero-shot settings. Combined with [[self-consistency]] decoding, it reaches state-of-the-art results on the math datasets.

## In this wiki

- **Co-origin with PAL.** [[pal-gao-2023|PAL]] and PoT proposed essentially the same delegate-to-the-interpreter idea at about the same time, and the two are usually cited together as the co-origins of program-delegated reasoning. See [[pal]]. [[chain-of-thought-prompting]] lists them together as the variant in which "chains are code, executed externally."
- **Start of the code-for-reasoning lineage.** The [[code-as-agent-harness-ning-2026]] survey begins its §2.1 *Code for Reasoning* with PoT/PAL. From there it follows the thread through formal verification (Lean, Coq, Isabelle provers) to iterative execution-grounded reasoning. The shift it describes is from "code as external calculator" to execution traces as reusable reasoning signals. See [[code-as-harness]].
- **The reasoning-side twin of Code as Policies.** [[code-as-policies-liang-2022]] is placed at the head of the survey's *Code for Acting* lineage, with PoT and PAL as the matching reasoning-side works.

## Related

- [[pal]] · [[chain-of-thought]] · [[chain-of-thought-prompting]] · [[self-consistency]]
- [[code-as-harness]] · [[code-as-policies]] · [[codeact]] · [[tool-use]] · [[plan-execute-verify-loop]]

## References

- Chen et al., "Program of Thoughts Prompting: Disentangling Computation from Reasoning for Numerical Reasoning Tasks" — [arXiv:2211.12588](https://arxiv.org/abs/2211.12588)
- [[code-as-agent-harness-ning-2026]] · [[pal-gao-2023]] · [[code-as-policies-liang-2022]]
