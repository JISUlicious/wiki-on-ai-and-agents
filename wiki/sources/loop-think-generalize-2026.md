---
title: "Loop, Think, & Generalize: Implicit Reasoning in Recurrent-Depth Transformers"
type: source
created: 2026-05-20
updated: 2026-05-20
sources:
  - loop-think-generalize-2026.md
arxiv_id: "2604.07822"
authors:
  - Harsh Kohli
  - Srinivasan Parthasarathy
  - Huan Sun
  - Yuekun Yao
year: 2026
introduces:
  - "[[recurrent-depth-transformer]]"
  - "[[latent-reasoning]]"
tags:
  - 2026
status: complete
importance: high
---

# Loop, Think, & Generalize: Implicit Reasoning in Recurrent-Depth Transformers

[arXiv:2604.07822](https://arxiv.org/abs/2604.07822) — Kohli, Parthasarathy, Sun, Yao (The Ohio State University, OSU-NLP Group), 2026. Code: https://github.com/OSU-NLP-Group/Loop-Think-Generalize.

## Summary

This paper introduces and names the **Recurrent-Depth Transformer (RDT)**: a decoder-only transformer in which a fixed block of `L` layers is applied iteratively for `R` recurrent iterations, yielding an effective rolled-out depth of `D = L × R` with shared parameters. The architecture is positioned as the modern descendant of the [[universal-transformer]] and the looped-transformer line (Yang et al. 2024a; Fan et al. 2025; Saunshi et al. 2025; Geiping et al. 2025; Zhu et al. 2025), and the paper provides the controlled empirical case that depth-recursion is what unlocks **implicit reasoning** — solving multi-step compositional queries inside a single forward pass without emitting any explicit chain-of-thought tokens.

Using a synthetic multi-hop knowledge-graph task (2,000 entities, 200 relations, ~40k atomic facts) the authors train models from scratch and isolate two out-of-distribution generalization challenges: **systematic generalization** (composing atomic facts that were never used in any composition during training) and **depth extrapolation** (training on up to `k_train`-hop chains, testing on deeper chains). Vanilla transformers (`R=1`) fail completely on both; even a simple `R=2` recurrent-depth model achieves non-trivial systematic generalization, and `R=4`/`R=8` converge dramatically faster in both epochs and wall-clock time.

This is the canonical 2026 paper anchoring the **"Recurrent-Depth Transformer"** terminology that has since been adopted by [[openmythos]] and other open-source implementations, and it ties the empirical story directly to Saunshi et al. 2025's "latent thoughts" framing — implicit reasoning happens through silent depth iteration in latent space rather than via externalized CoT tokens.

## Key Points

- **Architecture (RDT).** A GPT-2-style block of `L` shared-parameter layers is unrolled `R` times: `h^(r+1) = f_θ(h^(r); m)` for `r = 0, …, R−1`. The embedding and LM head are tied. The authors use a "simple looped transformer" variant similar to Saunshi et al. 2025, deliberately omitting input injection, gated halting, and middle-looping to keep the ablation clean.
- **Zero-initialization for stability.** Output projection matrices (`c_proj`) of both attention and FFN blocks are zero-initialized so that each recurrent block is exactly the identity at initialization, keeping the input-output Jacobian stable under unbounded unrolling (following Zhang et al. 2019). Shared-parameter deep nets are otherwise known to be unstable (Agarwala & Schoenholz 2022; Saunshi et al. 2025).
- **Stopping strategies.** Two regimes: **fixed iteration** (same `R` for all training instances) and **dynamic iteration** sampling `R ∼ clip(Poisson(λ), R_min, R_max)`. Unlike Fan et al. 2025, the authors do **not** assume oracle access to per-instance complexity, matching realistic pretraining settings (Geiping et al. 2025).
- **Implicit reasoning without CoT.** The model receives `<e_h><r_1>…<r_k>` and is supervised only on the final tail-entity token — there are no intermediate reasoning tokens. Multi-hop traversal must happen silently across the `R` depth iterations. This is the operational definition of implicit / [[latent-reasoning]] used in the paper.
- **Three-stage grokking dynamic.** Systematic generalization emerges through a sharp transition: (1) memorization of training set, (2) in-distribution generalization (classic grokking, ~10² epochs), (3) systematic generalization to held-out atomic compositions (~10⁴ epochs). Logit-lens analysis shows the bridge entity becoming readable at intermediate depths only after stage 3.
- **Depth extrapolation via inference-time recurrence.** Models trained on shallow chains (e.g. up to 5-hop) can extrapolate to deeper chains (e.g. 10-hop) by *increasing `R` at inference time* — a clean instance of [[inference-time-compute]] scaling without retraining or any explicit CoT scaffold. Dynamic-recurrence training yields the strongest extrapolation.
- **Overthinking is the key limit.** Pushing `R` too high at inference time degrades OOD accuracy — the model's prediction peaks at some optimal recurrence count, then collapses. Dynamic iteration training is more robust to overthinking than fixed; design elements like input injection (Bansal et al. 2022; Geiping et al. 2025) are noted as mitigations not used in this paper.
- **Contrast with vanilla transformers.** Knowledge in a vanilla transformer is layer-localized — a fact stored in shallow layers cannot be accessed by deeper layers because parameters are not shared. Weight sharing across the recurrent block is exactly what removes this barrier and enables flexible recomposition of parametric knowledge within one forward pass.

## Concepts

- [[recurrent-depth-transformer]] — the architecture named and characterized by this paper.
- [[latent-reasoning]] — reasoning that occurs in hidden-state space across depth iterations rather than via externalized tokens.
- [[looped-transformer]] — closely related/synonymous line of work (Yang et al. 2024a; Fan et al. 2025; Saunshi et al. 2025).
- [[universal-transformer]] — Dehghani et al. 2019 ancestor that first proposed sharing parameters across depth iterations.
- [[chain-of-thought-prompting]] — the explicit-token alternative this paper positions itself against; RDTs do compositional multi-hop reasoning *without* CoT.
- [[inference-time-compute]] — depth extrapolation is achieved by scaling recurrent iterations `R` at inference, a parameter-free form of test-time compute.
- [[grokking]] — the delayed-generalization phenomenon that structures the three-stage training dynamic observed for systematic generalization.
- [[compositional-generalization]] — the umbrella property the paper tries to instill: systematic generalization + depth extrapolation.

## Notable Quotes

> "We study implicit reasoning, i.e. the ability to combine knowledge or rules within a single forward pass."

> "While transformers can be trained to learn to combine such knowledge properly, they fail to compositionally generalize to unfamiliar combinations or deeper recursive combinations."

> "Recurrent-depth transformers enable depth extrapolation, generalizing to reasoning depths beyond those observed during training, as inference-time compute (i.e., recurrent iterations) increases."

> "Systematic generalization emerges through a sharp three-stage grokking process, that transitions from memorization to in-distribution generalization, and finally to systematic generalization."

> "Recurrent-depth transformers suffer from overthinking, which degrades performance and limits generalization to extremely deep recursions."

> "Our results highlight recurrent-depth transformers as a promising architecture for compositional reasoning over parametric knowledge."

## References

- Saunshi et al. 2025 — large-scale (250B-token Pile) looped-transformer pretraining; introduces "latent thoughts" framing that this paper builds on.
- Dehghani et al. 2019 — [[universal-transformer]], the parameter-sharing-across-depth ancestor.
- Geiping et al. 2025; Zhu et al. 2025 — recent realistic recurrent-depth pretraining at scale; source of the dynamic-iteration recipe.
- Fan et al. 2025; Yang et al. 2024a — small-scale looped-transformer studies on algorithmic tasks (parity, binary addition, linear regression).
- Bansal et al. 2022 — first identification of **overthinking** in recurrent/looped models and the input-injection mitigation.
- Wang et al. 2024a; Yao et al. 2025 — prior evidence that vanilla transformers fail at implicit multi-hop reasoning, motivating this work.
- [[openmythos]] — open-source implementation reference that adopts the "Recurrent-Depth Transformer" terminology established here.
