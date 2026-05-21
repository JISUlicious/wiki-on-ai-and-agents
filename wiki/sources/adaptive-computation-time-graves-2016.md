---
title: "Adaptive Computation Time for Recurrent Neural Networks"
type: source
created: 2026-05-21
updated: 2026-05-21
sources:
  - adaptive-computation-time-graves-2016.md
arxiv_id: "1603.08983"
authors:
  - Alex Graves
first_author: Alex Graves
year: 2016
introduces:
  - "[[adaptive-computation-time]]"
tags:
  - 2016
status: complete
importance: high
---

# Adaptive Computation Time for Recurrent Neural Networks

[arXiv:1603.08983](https://arxiv.org/abs/1603.08983) — Alex Graves (Google DeepMind), 2016.

## Summary

Graves introduces **Adaptive Computation Time (ACT)**, a mechanism that lets a recurrent neural network decide *how many computation steps* to perform at each element of its input sequence, rather than locking the per-step compute to a single state transition. The network is augmented with a sigmoidal **halting unit** that produces a per-step halting probability; intermediate states and outputs are accumulated as a mean-field (probability-weighted) average of the inner-loop iterations until the cumulative halting mass crosses `1 − ε`. The whole construction is deterministic and differentiable — no sampling, no gradient estimators — so it slots into ordinary backpropagation through time.

To prevent the network from "pondering" indefinitely (which would minimise prediction error at infinite compute cost), Graves adds a **ponder cost** `ρt = N(t) + R(t)` summed over the sequence and folds it into the loss with a time-penalty coefficient `τ`. The network thus learns to trade accuracy against compute, allocating extra steps only where they pay off. Experiments cover four synthetic algorithmic tasks — **parity, binary logic, integer addition, and sorting** — plus character-level prediction on the Hutter-prize Wikipedia corpus. On the synthetic tasks ACT yields dramatic gains and visibly adapts compute to instance difficulty; on character prediction the perplexity gain is small but ponder time spikes interpretably at hard transitions (word boundaries, sentence ends), hinting at ACT as a generic boundary-detection signal.

This paper is the **canonical origin of input-adaptive depth** in neural networks. The mechanism — halting probability head plus ponder regulariser — is reused almost verbatim by the [[universal-transformer]] of Dehghani et al. 2018, and recurs throughout the [[looped-transformer]] / [[recurrent-depth-transformer]] line of work as the way to let a model "think for longer" on harder inputs. Anyone reading modern looped-transformer papers should treat Graves 2016 as the genesis text.

## Key Points

- **Per-step halting probability head.** A sigmoid unit `h_t^n = σ(W_h s_t^n + b_h)` is appended to the recurrent network's output. Cumulative halting mass at inner-step `n` determines whether to stop pondering input `t`; the residual probability `R(t)` closes the distribution so probabilities sum to 1.
- **Mean-field state/output.** Rather than sampling a discrete halt time, the final state and output at input step `t` are the halting-probability-weighted mean of the intermediate `s_t^n, y_t^n`. This is fully differentiable and avoids the noisy gradients of REINFORCE-style discrete halting.
- **Ponder cost regulariser.** `L̂(x,y) = L(x,y) + τ · P(x)` where `P(x) = Σ_t (N(t) + R(t))` upper-bounds total inner-loop iterations. `τ` is hand-tuned via log-grid search; the network's behaviour is acknowledged to be quite sensitive to `τ`.
- **Algorithmic-task evaluation.** Parity, binary logic, integer addition, sort on real numbers — tasks chosen because they reward sequential reasoning that a vanilla per-input single-step RNN cannot easily perform. ACT dramatically lowers sequence error vs. baseline; without ACT, parity on 64-bit static inputs barely beats random guessing.
- **Halting probability as interpretable confidence/boundary signal.** On Wikipedia character prediction, more compute is spent on harder-to-predict transitions (spaces, ends of sentences). Graves explicitly suggests ACT as a generic method for inferring segment boundaries in sequence data.
- **Mean-field vs. stochastic halting.** Graves argues the mean-field formulation is preferable for long sequences because sampling noise compounds across successive halting decisions; this is the design choice that survived into modern variants.
- **Precursor to all variable-depth recurrent / looped architectures.** Universal Transformer, PonderNet, and the looped-transformer line all inherit ACT's halting head + ponder cost template.

## Entities

- [[alex-graves]] (author)
- [[deepmind]] (Google DeepMind, affiliation)

## Concepts

- [[adaptive-computation-time]] — the core mechanism introduced.
- [[universal-transformer]] — canonical transformer-era descendant; reuses ACT halting.
- [[looped-transformer]] — broader family that uses ACT-style halting to choose loop count.
- [[recurrent-depth-transformer]] — variable-depth recurrent reasoning over a fixed-weight stack.
- [[recurrent-neural-network]] — the substrate ACT was originally bolted onto.
- [[recurrence]] — the structural pattern (revisit the same weights N times) that ACT controls.

## Notable Quotes

> "The amount of time required to pose a problem and the amount of thought required to solve it are notoriously unrelated… Most machine learning algorithms, however, are unable to dynamically adapt the amount of computation they employ to the complexity of the task they perform."

> "In the interests of both computational efficiency and ease of learning it seems preferable to dynamically vary the number of steps for which the network 'ponders' each input before emitting an output. In this case the effective depth of the network at each step along the sequence becomes a dynamic function of the inputs received so far."

> "ACT does not yield large gains in performance [on character prediction]; however it does provide intriguing insight into the structure of the data, with more computation allocated to harder-to-predict transitions, such as spaces between words and ends of sentences."

## References

- [[universal-transformer-dehghani-2018]] — the canonical transformer-era descendant; lifts ACT's halting head + ponder cost into the transformer setting, making "ACT in transformers" a standard pattern.
- [[looped-transformers-length-generalization-fan-2024]] — modern looped-transformer work that reuses ACT-style halting to choose loop count per input, demonstrating ACT's continued relevance in the post-transformer landscape.
