---
title: Recurrent-Depth Transformer
type: concept
created: 2026-05-20
updated: 2026-05-20
sources:
  - loop-think-generalize-2026.md
  - parcae-scaling-laws-looped-2026.md
status: complete
importance: high
tags:
  - 2026
---

The **Recurrent-Depth Transformer (RDT)** is the modern (2025–2026) rebranding of [[looped-transformer|looped transformers]] for pretraining-scale language models. An RDT is a decoder-only transformer in which a fixed block of `L` layers is applied iteratively for `R` recurrent iterations, yielding an effective rolled-out depth of `D = L × R` with shared parameters. The terminology was crystallized by Kohli et al. (2026) and adopted by the open-source [[openmythos]] reference implementation; the load-bearing stability recipe comes from [[parcae|Parcae]] (Prairie et al., 2026), which makes the architecture trainable at 1.3B-parameter / 100B-token scale on par with or above a parameter-matched Transformer.

## Core mechanism

- `k`-layer × `R`-iteration architecture: `h^(r+1) = f_θ(h^(r); m)` with tied embedding and LM head; only the middle block is typically looped (the first and last layers stay independent, following Saunshi 2025's middle-looping variant).
- Variable / Poisson-sampled iteration count during training (`R ∼ clip(Poisson(λ), R_min, R_max)`) enables inference-time depth scaling — at test time, increasing `R` extrapolates to deeper compositional chains than were seen during training.
- Supervision only on the final tail token (no intermediate CoT tokens) forces multi-hop reasoning to happen implicitly inside the loop, in latent space — operationalizing [[latent-reasoning]] as the training objective rather than a post-hoc interpretation.
- Zero-initialized output projection (`c_proj`) at the loop boundary makes each recurrent block exactly the identity at initialization, stabilizing the input-output Jacobian under unbounded unrolling (Zhang et al. 2019; standard in Kohli 2026 and Geiping 2025).
- [[parcae|Parcae]] (2026) gives the LTI-system framing plus a negative-diagonal `A` parameterization (`A := Diag(−exp(log A))` discretized via ZOH) that mechanically forces ρ(A) < 1 — the load-bearing scaling-law recipe that removes the need for aggressive residual normalization and yields predictable Chinchilla-style scaling laws for the loop axis.

## What it advances

RDTs reposition depth-recurrence from a small-model curiosity (Yang 2023; Fan 2024) to a third pretraining-scale axis alongside parameters and data. They are the architectural substrate for **implicit reasoning** — Kohli 2026 trains models from scratch on multi-hop knowledge-graph tasks and shows that `R=2` already produces non-trivial systematic generalization where a vanilla `R=1` Transformer fails completely. Parcae makes the recipe predictable: training-time scaling laws govern how to allocate FLOPs between data and loops at fixed parameter count, and a separate test-time law governs the saturating exponential decay in loss from additional inference-time loops. The 770M Parcae model matches a 1.3B Transformer on the CORE benchmark — the first crisp demonstration that loops can substitute for parameters at frontier-adjacent scale.

## Relation to other concepts

RDT is the modern terminology for what [[looped-transformer]] research called "looped"; the [[universal-transformer]] is the 2018 ancestor. [[latent-reasoning]] is the cognitive interpretation — depth iteration as silent CoT. [[parcae]] is the stability recipe; [[openmythos]] is the open-source implementation building on it. [[scaling-laws]] are extended to a third axis (loops). [[inference-time-compute]] is the operational knob: more loops at test time = more reasoning compute, without emitting tokens. Connections also run to [[grokking]] (the three-stage training dynamic observed for systematic generalization in Kohli 2026) and to [[parameter-sharing]] as the underlying mechanism.

## References

- [[loop-think-generalize-2026]] — names the architecture, shows implicit reasoning + depth extrapolation.
- [[parcae-scaling-laws-looped-2026]] — the stability recipe + scaling laws.
- [[looped-transformer]] — the broader concept family RDT specializes.
- [[universal-transformer]] — the 2018 ancestor.
- [[openmythos]] — open-source reference implementation.
