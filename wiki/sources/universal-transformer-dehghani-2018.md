---
title: "Universal Transformers"
type: source
created: 2026-05-20
updated: 2026-05-20
sources:
  - universal-transformer-dehghani-2018.md
arxiv_id: "1807.03819"
venue: ICLR 2019
authors:
  - Mostafa Dehghani
  - Stephan Gouws
  - Oriol Vinyals
  - Jakob Uszkoreit
  - Łukasz Kaiser
first_author: Mostafa Dehghani
year: 2018
introduces:
  - "[[universal-transformer]]"
  - "[[adaptive-computation-time]]"
tags:
  - 2018
status: complete
importance: high
---

# Universal Transformers

**Source**: `sources/universal-transformer-dehghani-2018.md`
**arXiv**: [1807.03819](https://arxiv.org/abs/1807.03819)
**Venue**: ICLR 2019
**Authors**: Mostafa Dehghani, Stephan Gouws, Oriol Vinyals, Jakob Uszkoreit, Łukasz Kaiser ([[google-brain]], University of Amsterdam, [[deepmind]])

## Summary

This paper introduces the **Universal Transformer (UT)** — a parallel-in-time, self-attentive *recurrent* sequence model that generalizes the [[transformer-architecture|Transformer]] by recursively applying a single transformer block over "depth" rather than stacking a fixed number of distinct layers. At every step, the UT refines the representations of all sequence positions in parallel via multi-head self-attention followed by a shared transition function (a separable convolution or position-wise MLP). Crucially, weights are tied across both positions and time-steps, giving the model the parallelizability and global receptive field of the Transformer together with the iterative inductive bias of an [[recurrence|RNN]].

To allocate compute non-uniformly, the UT couples this depth-recurrence with **[[adaptive-computation-time|Adaptive Computation Time]] (ACT)** applied independently *per position*: each symbol's recurrent block predicts a halting probability and stops updating once the cumulative probability saturates, while other positions continue. This is the first demonstration that per-position conditional computation can *improve* accuracy on structured language and algorithmic tasks (though it slightly hurts machine translation).

Theoretically, the authors argue the UT is **Turing-complete** under mild assumptions — unlike the vanilla Transformer, whose depth cannot scale with input length, the UT can dynamically run for arbitrarily many steps and can simulate models like the Neural GPU and Neural Turing Machine. Empirically, UT outperforms the standard Transformer and LSTM baselines on bAbI (12/20 vs. Transformer's 4/20 tasks solved in the joint 10K setting), subject-verb agreement, learning-to-execute and algorithmic copy/reverse/addition, achieves SOTA on LAMBADA, and improves WMT14 En-De translation by +0.9 BLEU over the base Transformer. UT is widely cited as the spiritual ancestor of the modern [[looped-transformer]] research thread.

## Key Points

- **Recursive transformer block.** A single self-attention + transition block is applied repeatedly; depth = number of recurrent steps `T`. Equivalent to a multi-layer Transformer with all layers' weights tied.
- **Two-dimensional positional encoding.** Sinusoidal embeddings indexed by both position `i` and time-step `t` are summed into the state at every step, so the model knows where it is in space *and* depth.
- **Per-position ACT halting.** Each symbol's recurrent column maintains its own halting probability; once a position halts, its state is copied forward until all positions halt or `T_max` is reached. Dynamic halting acts as a regularizer that lets the model spend more compute on "harder" symbols.
- **bAbI.** UT with dynamic halting solves 20/20 tasks in the 10K joint setting (avg error 0.29%) vs. the vanilla Transformer's 8/20 (22.1% avg error). The model learns to spend more ponder steps on questions requiring more supporting facts (~3.8 steps for 3-fact tasks vs. ~2.3 for 1-fact tasks).
- **LAMBADA.** UT with dynamic halting achieves SOTA perplexity 142 and accuracy 0.19 on test; the vanilla Transformer fails catastrophically (perplexity 7321).
- **Algorithmic tasks (Copy / Reverse / Addition, train len 40 → test len 400).** UT massively beats Transformer (e.g. Reverse: 0.96 vs. 0.13 char-acc). Approaches but does not match the curriculum-trained Neural GPU.
- **Learning to Execute.** UT achieves perfect char- and seq-accuracy on all memorization tasks (Copy / Double / Reverse) and strongly beats Transformer and LSTM on program evaluation.
- **WMT14 En-De.** UT base reaches 28.9 BLEU vs. 28.0 for Transformer base (same parameter count) — without ACT, since dynamic halting marginally hurts MT.
- **Turing-completeness argument.** Given sufficient memory, the UT can simulate any Turing machine: with `T` proportional to input length and the right parameterization, it reduces to a Neural GPU; its per-step parallel global rewrites also subsume a single NTM read/write step. This is the key theoretical gap the paper closes vs. the vanilla Transformer.
- **Weight sharing as inductive bias.** The conclusion frames UT as combining the weight-sharing intuition of CNNs/RNNs with the global receptive field of attention, suggesting tied-depth recurrence is a useful prior for iterative reasoning.

## Entities Mentioned

- [[google-brain]] — Jakob Uszkoreit and Łukasz Kaiser's affiliation; primary lab home for the project.
- [[deepmind]] — Stephan Gouws and Oriol Vinyals's affiliation.
- Mostafa Dehghani — first author, then at University of Amsterdam (work performed while at Google Brain).
- Jakob Uszkoreit and Łukasz Kaiser — co-authors who also worked on the original [[transformer-architecture|Transformer]] (Vaswani et al. 2017), providing direct lineage from that paper.

## Concepts Discussed

- [[transformer-architecture]] — the direct predecessor; UT is cast as "Transformer with depth-tied weights and variable depth".
- [[universal-transformer]] — the model introduced by this paper.
- [[adaptive-computation-time]] — the Graves 2016 mechanism, here applied *per position* for the first time.
- [[recurrence]] — UT recurs over *depth* (revisions of representation) rather than over sequence positions; contrasted with classical time-recurrent RNNs which lack random-access memory in the recurrent step.
- [[turing-completeness]] — claimed for UT under mild assumptions; not held by the vanilla Transformer with fixed depth.
- [[looped-transformer]] — modern descendants of the UT idea (weight-tied or block-iterated transformers used for in-context algorithm learning, reasoning, and iterative refinement).
- Neural GPU, Neural Turing Machine, end-to-end Memory Networks — explicitly compared as related computationally-universal or memory-augmented architectures.

## Notable Quotes

> "We propose the Universal Transformer (UT), a parallel-in-time self-attentive recurrent sequence model which can be cast as a generalization of the Transformer model... UTs combine the parallelizability and global receptive field of feed-forward sequence models like the Transformer with the recurrent inductive bias of RNNs." — Abstract

> "Given sufficient memory the Universal Transformer is computationally universal — i.e. it belongs to the class of models that can be used to simulate any Turing machine, thereby addressing a shortcoming of the standard Transformer model." — §4 Discussion

> "The UT is not computationally bound by the number of symbols in the sequence, but only by the number of revisions made to each symbol's representation." — §2.1 Model Description

## References

_Original source: `sources/universal-transformer-dehghani-2018.md` (extracted from the ICLR 2019 camera-ready, arXiv:1807.03819v3)._
_Code released as part of `tensor2tensor` (TensorFlow): https://github.com/tensorflow/tensor2tensor_
