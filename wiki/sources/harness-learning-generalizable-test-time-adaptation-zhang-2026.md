---
title: "Harness Learning Enables Generalizable Test-Time Adaptation"
type: source
created: 2026-10-29
updated: 2026-10-29
sources:
  - harness-learning-generalizable-test-time-adaptation-zhang-2026.md
arxiv_id: "2609.35738"
authors:
  - Alvin Zhang
  - Xuecheng Liu
  - Zixuan Wang
  - Fahim Tajwar
  - Daman Arora
  - Ruslan Salakhutdinov
  - Daniel Khashabi
  - Yuda Song
  - Andrea Zanette
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# Harness Learning Enables Generalizable Test-Time Adaptation

**Alvin Zhang, Xuecheng Liu, Zixuan Wang, Fahim Tajwar, Daman Arora, Ruslan Salakhutdinov, Daniel Khashabi, Yuda Song, Andrea Zanette** — [arXiv:2609.35738](https://arxiv.org/abs/2609.35738), v1, cs.CL, 28 Sep 2026. Preprint. The PDF prints no affiliations. Its acknowledgments credit compute at Johns Hopkins (ARCH) and [[cmu|CMU]] (FLAME, Babel).

## Summary

*Harness learning* frames harness revision as [[meta-learning]] over executable programs. A **proposer** model reads the task description, the current harness code and an execution report, and then writes a code edit. Harness edits play the role that weight updates play in gradient-based adaptation. The proposer is trained with RL, optionally after SFT on teacher revisions, and the reward is the task score of the revised harness. The solver stays frozen throughout. At test time both models are frozen, and the proposer adapts the harness for a new task over several rounds using only execution feedback.

## Key points

- **Setup.** On Reasoning Gym, the proposer and solver are both Qwen3.5-4B, with SFT on revisions from a **Qwen3.6-35B-A3B** teacher followed by RL. On QA, a Qwen3-4B proposer drives a Qwen3-8B solver, trained with RL only on HotpotQA. MuSiQue and 2WikiMultihopQA are never seen in training.
- **Unseen Reasoning Gym families (21).** Mean single-revision score rises from **0.32 (base) to 0.62 (SFT+RL)**. The teacher scores 0.56, so the trained 4B proposer beats it on average. The teacher still wins under oracle best-of-8.
- **Including the 5 hardest unseen families (26 total)**, the means are base 0.282, SFT 0.415, Single-step RL 0.528 and teacher 0.501. The main figures exclude those five families.
- **Iterated adaptation works even without multistep training.** On MuSiQue, independent revisions from the Single-step RL proposer average 0.15 EM, barely above the seed harness. Ten sequential rounds reach **0.27**. On both unseen QA sets, the average final run beats the oracle best of 80 independent proposals.
- **Multistep RL doesn't reliably help.** Training on revision sequences beats single-step training on canonical Reasoning Gym questions, loses on format-varied questions, and ties on QA.
- On Reasoning Gym, revisions mostly add interpreter tools; 48% of the teacher's revisions do this.

> [!warning] Newsletter vs paper
> "CMU" is a partial affiliation. The paper lists none, and its acknowledgments credit both Johns Hopkins and CMU. "4B proposer beats 35B teacher" holds for **mean** proposal quality on the 21 reported unseen families (0.62 vs 0.56). It does not hold under best-of-8 selection. The teacher is a **35B-A3B MoE with ~3B active parameters**, so this is not a 4B-vs-dense-35B result.

## What it does not show

- **Small solvers and small tasks.** All solvers are 4B–8B models on reasoning puzzles and multi-hop QA. There are no long-horizon tool-use or agentic benchmarks; the authors list longer trajectories as future work.
- **Compute is not matched.** The 80-proposal comparison matches proposal count but "leaves total compute and feedback unmatched."
- **Narrow training signal.** There is one teacher, a fixed solver, and prescribed revision prompts.

## Concepts & entities

- [[agentic-harness-engineering]] · [[code-as-harness]] — the learned-proposer branch.
- [[harness-r1-learning-to-edit-runtime-harnesses-shao-2026]] — closest prior work, which the paper says uses a similar SFT→RL recipe with a frozen solver.
- [[jit-agent-just-in-time-harness-evolution-zhang-2026]] · [[meta-harness-lee-2026]] · [[autoharness-lou-2026]] — cited harness-optimization relatives.
- [[meta-learning]] · [[reinforcement-learning]] · [[knowledge-distillation]] · [[test-time-compute]]
- [[harness-zero-harness-distillation-agent-as-harness-ye-2026]] — the opposite move: distill the harness into the weights instead of learning to edit it.

## References

- [arXiv:2609.35738](https://arxiv.org/abs/2609.35738)
