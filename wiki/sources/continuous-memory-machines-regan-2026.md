---
title: "Continuous Memory Machines"
type: source
created: 2026-10-29
updated: 2026-10-29
sources:
  - continuous-memory-machines-regan-2026.md
arxiv_id: "2610.07907"
venue: NeurIPS 2026 Workshop (PALM)
authors:
  - Ciaran Regan
  - Kai Arulkumaran
  - Luke Darlow
  - Stefania Druga
  - Sebastian Risi
  - Llion Jones
year: 2026
status: complete
importance: low
tags:
  - 2026
  - paper
---

# Continuous Memory Machines

**Ciaran Regan, Kai Arulkumaran, Luke Darlow, Stefania Druga, Sebastian Risi, Llion Jones** ([[sakana-ai|Sakana AI]]; first author also University of Tsukuba) — [arXiv:2610.07907](https://arxiv.org/abs/2610.07907), v1, cs.AI, 6 Oct 2026. NeurIPS 2026 workshop paper (PALM: Personalized, Aligned, Long-Term Memory). Code: `SakanaAI/continuous-memory-machines`.

## Summary

A [[recurrent-neural-network|recurrent network]] usually packs short-term computation and long-term retention into one hidden vector. Sakana's Continuous Thought Machine (CTM) has rich short-term dynamics (neuron-level models that read each neuron's recent activation history; outputs from neuron synchronization) but no long-term store. The **Continuous Memory Machine (CMM)** adds one. It has two **matrix-valued** recurrent states: a sliding **short-term memory** (STM) of recent pre-activations, and a persistent **long-term memory** (LTM). A small Transformer jointly updates both at every internal step, so each store can reorganize itself and read from or write to the other (memory interaction by self-attention, as in the Relational Memory Core). The framing cites complementary learning systems from neuroscience.

## Key points

- **Copy** (100 vectors): CMM, DNC and NAM-TM all reach loss below 5×10⁻²; CTM 3.1×10⁻¹. The non-recurrent Transformer is near zero.
- **Recall:** CMM, DNC and LSAM below 10⁻³; CMM reaches 99.95% accuracy. CTM's loss is over 60× higher. Transformer 2.7×10⁻⁴.
- **Sort** (N=20, output 16 lowest): CMM best of all models at 4.1×10⁻³ vs DNC 1.4×10⁻² and Transformer 2.7×10⁻². With training on N ≤ 16 and testing at N = 50, accuracy is 72.1% vs RMT 68.2% and DNC 55.8%.
- **Maze** (39×39, 10M params): action accuracy 88.8% → 96.3% and full-maze accuracy 37.4% → 61.6% over the CTM. Path-following attention is preserved; the best seed reaches 90.7%.
- **Few-shot regression** (in-context): best in both K=10 and K=20 settings.
- **Ablations.** CMM-LSTM (CTM backbone swapped for an [[lstm|LSTM]]) is worse on copy, recall and sort, so the CTM dynamics add something. No ablation beats the full CMM on sort.

> [!warning] Newsletter vs paper
> "Beats LSTM, DNC, RMC, CTM on copy/recall/sort/regression/maze" overstates it. On **copy** the CMM is in a tie group with the DNC and NAM-TM, and the Transformer is better. On **recall** it is roughly level with the DNC, LSAM and Transformer. It is clearly best on sort, the maze and regression. For the maze, GDN, RMT and LSAM are excluded because they could not train stably.

## What it does not show

- Toy-scale algorithmic tasks and a 10M-parameter maze model. No language modelling and no LLM-scale experiments, despite the workshop's long-term-memory theme.
- A workshop paper; maze results vary noticeably across seeds at long path lengths.

## Concepts & entities

- [[recurrent-neural-network]] · [[lstm]] · [[recurrence]] · [[linear-depth-memory]] — the recurrent-state bottleneck.
- [[test-time-memory]] · [[neural-long-term-memory]] · [[gated-deltanet]] — other matrix-valued or learned memory states (GDN is a baseline here).
- [[in-parameter-memory-survey-huang-2026]] — wider landscape of learned memory states.
- [[adaptive-computation-time]] · [[looped-transformer]] — internal "thinking steps", as in the CTM.
- [[sakana-ai]] · [[transformer-architecture]].

## References

- [arXiv:2610.07907](https://arxiv.org/abs/2610.07907)
