---
title: "Hilbert Operator for Progressive Encoding (HOPE): A Mathematical Framework for Deconstructing Learned Representations in Deep Networks"
type: source
created: 2026-08-05
updated: 2026-08-05
sources:
  - hope-hilbert-operator-progressive-encoding-mobahi-2026.md
arxiv_id: "2607.21366"
year: 2026
authors:
  - Hossein Mobahi
  - Peter L. Bartlett
tags:
  - 2026
  - paper
status: complete
importance: medium
---

# Hilbert Operator for Progressive Encoding (HOPE): A Mathematical Framework for Deconstructing Learned Representations in Deep Networks

**Hossein Mobahi, Peter L. Bartlett** (Google DeepMind; Bartlett also UC Berkeley) — [arXiv:2607.21366](https://arxiv.org/abs/2607.21366), cs.LG, 23 Jul 2026. Preprint; no venue listed in the arXiv comments.

## Summary

HOPE uses **compression as an instrument for reading a trained network**, on the premise (Rissanen; Hinton & Van Camp; the more recent "LLMs are general-purpose compressors" line) that learning *is* compression, so progressively compressing a network peels away malleable slack and exposes the invariant core. The obstacle is that the usual compression heuristics are measuring the wrong thing. Magnitude pruning scores neurons by ‖w‖, but weight magnitude in a network with positively-homogeneous activations is an **optimization artifact**: scaling a neuron's incoming weights by λ and its outgoing weights by 1/λ leaves the computed function untouched while moving any magnitude score arbitrarily. Empirical alternatives (activation matching, curvature/Fisher approximations) avoid that but tether the analysis to a specific finite dataset — brittle under distribution shift, punishing on long-tail features, and expensive because every greedy step needs fresh forward passes.

HOPE's move is to **leave parameter space for function space**. A neuron is not its weight matrix; it is the transformation it applies over the data manifold. Formally, define H ≜ L²(X, P_X; ℝ^c), let g_i(x) = Ψ(w_in,i ᵀ x + b_i) be the neuron's scalar input landscape in H_in, and let w_out,i ∈ ℝ^c be its output direction. The neuron is the tensor product **f_i ≜ g_i ⊗ w_out,i — a rank-1 Hilbert-Schmidt operator** — and its *capacity* is the Hilbert-Schmidt norm ‖f_i‖_H = ‖w_out,i‖₂·√K(i,i), with the neural kernel K(i,j) ≜ E_{x~P_X}[Ψ(y_i)Ψ(y_j)]. Because positive homogeneity scales K by λ while ‖w_out‖ scales by 1/λ, the two factors cancel and the capacity score is **invariant to weight resharding**; it is likewise invariant to fan-in and tensor shape, so capacities are comparable across heterogeneous layers. The remaining ingredient is the distribution P_X, and this is where HOPE gets to be **data-free**: rather than generating synthetic images (as data-free [[knowledge-distillation]] does), it applies the **maximum-entropy principle directly to the BatchNorm statistics already stored in the checkpoint**. The mean comes from the Moore-Penrose pseudo-inverse of the raw weights applied to the BN means, and the covariance maximizes log det Σ subject to the BN variance constraints w_raw,iᵀ Σ w_raw,i = σ_i², yielding a Gaussian surrogate whose integrals close in form. (For BN-free architectures HOPE needs one calibration pass over a small batch.)

In this space **pruning and merging become the same operation** — low-rank subspace projection. Pruning projects f_i onto the null operator, so the distortion is ‖f_i‖_H. Merging projects the rank-2 joint operator [f_i, f_j] onto the *constrained* rank-1 form [f_p, f_p] (constrained because one physical neuron must emit one output, which rules out the unconstrained Eckart–Young–Mirsky truncation), giving distortion √(‖f_i − f_p‖² + ‖f_j − f_p‖²); Section 7 derives the optimal parent's direction and scale and projects it back onto discrete conv/BN parameters. Section 8 lifts the same metric to the **macro scale — evicting an entire residual block** to a pure identity map — by coupling the internal layers with the skip pathway, whose "parallel survival capacity" E_identity = Σ_k √(γ_k² + β_k²) comes straight from the preceding BN. That is what makes cross-scale comparison legitimate: pruning one neuron and deleting a whole ResNet bottleneck are scored under one metric. Finally, Section 9 borrows from **rate-distortion theory**: the choice of next action is a knapsack problem (minimize Σ a_k J_k subject to a parameter budget), NP-hard in the discrete form, solved by Dantzig's continuous relaxation — sort by distortion rate J_k/ΔP_k and take the minimum — executed **receding-horizon**, one action at a time with the network state recomputed after each. ΔP is deliberately taken at its *initial* static value, because using the live shrinking footprint violates Dantzig's item-independence axiom and traps the optimizer in fragmented architectures. The framework is hyperparameter-free (and data-free given BN); the paper is explicit that its experiments are proof-of-concept, not exhaustive benchmarks.

The second application inverts the tool: instead of removing capacity, use the capacity ranking to decide **what to protect during transfer**. **DEFT (Dispersed Elastic Fine-Tuning)** runs HOPE's progressive encoding, records each neuron's J_prune, thresholds it at a percentile P to produce a binary elasticity map E ∈ {0,1}, and partitions the network into a frozen **Universal Core** (E=0) and a plastic **Peripheral Slack** (E=1). Two mechanisms make the partition actually hold. First, a **structural mask at initialization severs every weight pointing from plastic neurons into frozen ones**, so slack updates cannot induce representational drift in the core — this replaces the inference-time masks of PackNet and needs no task identity. Second, **redundancy is dynamically released**: features fragmented across M correlated neurons would otherwise be locked wholesale, so DEFT MERGEs them into one rank-1 parent and hands the freed M−1 children to the slack with full plasticity, *manufacturing* capacity for the target task rather than merely allocating it. Gradients are then scaled elementwise by E_out before the optimizer step (pre-step, to prevent momentum drift on frozen parameters). Unlike EWC, DEFT never touches source data and computes capacities in O(N) rather than paying the O(N³) covariance cost of orthogonal-projection methods.

## Key points

- **Capacity, not magnitude.** ‖f_i‖_H = ‖w_out,i‖₂ √K(i,i) with K(i,j) = E[Ψ(y_i)Ψ(y_j)]. Scale-symmetry-invariant by construction; shape-invariant because BN absorption decouples expected activation energy E[Ψ²(y_i)] from fan-in n.
- **Unified action costs** (Eq. 6): J_prune = N‖f_i‖_H / (E_a − ‖f_i‖_H); J_merge = N√(‖f_i−f_p‖²_H + ‖f_j−f_p‖²_H) / (E_a − ‖f_i‖_H − ‖f_j‖_H + ‖f_p‖_H). Under a mean-field assumption J_prune ≈ 1, so the penalty is invariant to instantaneous layer width.
- **Macro block eviction** (Eq. 20): J_evict = Σ_{l=1,2} N_active^(l) E_active^(l) / Σ_k √(γ_k² + β_k²) — one number that makes deleting a ResNet-50 bottleneck commensurate with pruning a single neuron.
- **Action selection** (Eq. 23): k* = argmin_k J_k / ΔP_k^init, the continuous-knapsack greedy rule under a receding horizon. Cached pairwise constants keep each J evaluation O(1), with an O(N) local update after each executed action.
- **Compression result.** On the public Keras ImageNet-pretrained **ResNet-50 (V1)** checkpoint, HOPE traces a better accuracy-vs-density curve (density = active/initial neurons network-wide) than three structured baselines: L1-norm input pruning, L1-norm joint pruning, and BN-scale (γ) pruning. Reported as a plot; the paper gives no accuracy table for this experiment. HOPE is structured (whole neurons), so the savings are realizable without sparse-matrix hardware, and because encoding is progressive **every intermediate iterate is a valid compressed model**.
- **DEFT transfer benchmark.** Source: CIFAR-100 tasks built from 4 randomly sampled superclasses → 20 fine-grained classes (dense semantic clusters, to force hierarchical features). Target: full 10-class SVHN. 4 trials × 5 tasks = 20 cross-domain scenarios; metric is **H-Score**, the harmonic mean of target accuracy and source retention. DEFT grid-searches only the percentile P ∈ {60, 40, 30, 20}.

| Method | Target Acc (SVHN) | Source Retention (CIFAR) | H-Score |
|---|---|---|---|
| **DEFT (ours)** | 89.79 ± 0.84 | **52.14 ± 5.29** | **65.82 ± 3.96** |
| Head-Only (linear probe) | 36.11 ± 2.79 | 63.13 ± 4.62 | 45.79 ± 2.05 |
| Full FT | **94.09 ± 0.28** | 7.52 ± 1.63 | 13.88 ± 2.84 |
| EWC | 93.94 ± 0.22 | 6.74 ± 1.74 | 12.54 ± 2.99 |
| PEFT (BN-tuning) | 81.91 ± 0.49 | 5.44 ± 0.98 | 10.18 ± 1.63 |

- **Reading the table.** Full FT buys the best target accuracy by destroying the source (7.52% retention). Head-Only preserves the most source knowledge (63.13%) but only reaches 36.11% on SVHN — the domain gap is too wide for frozen features to be linearly separable. **EWC behaves almost identically to Full FT** (93.94 / 6.74), i.e. its local Fisher penalty fails to arrest forgetting under a large domain shift, which is precisely the brittleness HOPE's global, curvature-free capacity measure is meant to avoid. PEFT (BN-affine only) is worst on both axes here. DEFT keeps 89.79 of Full FT's 94.09 plasticity while lifting retention nearly 7× to 52.14, and its H-Score of 65.82 beats every baseline by a wide margin.
- **Appendix H** gives theoretical guarantees for DEFT: axioms formalizing the structural mask and the merge-then-release rule, a proof that severed core-directed weights make target-driven drift exactly zero (a changing signal times zero is zero), and a bound showing the released capacity carries only the accumulated bounded projection error Σ δ_k of the merges.

## Concepts & entities

- [[mechanistic-interpretability]] — HOPE is an unusual entry in this space: instead of reading features off activations, it deconstructs representations by asking *what survives progressive compression*, using capacity in a function space as the operational definition of "load-bearing." The core/slack partition it produces is a coarse but data-free, hyperparameter-free structural claim about what a trained network actually encodes.
- [[knowledge-distillation]] — shares the data-free ambition and the same BN-statistics resource, but where data-free distillation *synthesizes* inputs from BN moments to drive a student, HOPE uses the same moments analytically as a maximum-entropy surrogate distribution and never runs a forward pass at all.
- [[quantization]] — the complementary axis of model compression. HOPE is structured *neuron* removal scored by rate-distortion (J_k/ΔP_k under a parameter-count budget, with bit reduction assumed proportional to parameter reduction at fixed precision); it reduces the number of weights rather than the precision of each, so the two compose.

## References

- [arXiv:2607.21366](https://arxiv.org/abs/2607.21366)
