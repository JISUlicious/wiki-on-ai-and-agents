---
title: "Neither Parallel Nor Sequential: How DiffusionGemma Actually Commits Tokens"
type: source
created: 2026-09-21
updated: 2026-09-21
sources: []
arxiv_id: "2606.14620"
authors:
  - Ali Asaria
  - Tony Salomone
  - Deep Gandhi
year: 2026
status: complete
importance: high
tags:
  - 2026
  - paper
---

# Neither Parallel Nor Sequential: How DiffusionGemma Actually Commits Tokens

**Ali Asaria, Tony Salomone, Deep Gandhi** (all Transformer Lab; Gandhi corresponding) — [arXiv:2606.14620](https://arxiv.org/abs/2606.14620), cs.LG, v1 12 Jun 2026, CC BY 4.0. No venue stated. **No code, data, or traces released** (available from the authors on request). Total compute: ~0.9 H100-hours.

> [!important] The commit rule, plainly
> [[diffusiongemma]] does **not** produce its 256-token canvas simultaneously, and it is **not** block-autoregressive with a 16-token block either. It commits in a **handful of large, tied batches**: a mean of **5.8–25.5 content tokens per accept-call** depending on regime, over only **3.3–17.1 accept-calls** out of the shipped **48-step** budget. Those batches land in a **partial left-to-right order** — tie-aware Kendall τ_b of **0.430–0.604** at the token level, far below both strict autoregression (1.0) and a pure 16-block-sequential control (**0.94–0.96**). Within a batch the order is genuinely *undefined*, not merely unobserved (up to **72%** of token pairs commit in the same call). The apparent "block size" is an artifact of the analyst's bin: block-τ_b rises **smoothly** 0.59→0.64→0.70→0.79→0.91 as bins grow 4→8→16→32→64, with **no jump at 16**. So: *locally parallel, globally left-to-right-ish, with no privileged block* — and structured JSON is a clean exception at **τ_b = −0.044**, essentially order-independent.

## Problem

Open diffusion LMs are marketed as parallel, non-autoregressive decoders, but the order a *shipped checkpoint* actually finalizes tokens in is an empirical property of the trained model plus its sampler, not of the architecture. Prior work splits between "learned order is secretly left-to-right" (Chen et al. 2025a; Israel et al. 2025) and "structured out-of-order anchor-first generation" (Hong et al. 2026; Zhong et al. 2026). This paper measures a single public production checkpoint instead of proposing a decoding strategy.

## What was measured

Subject: `google/diffusiongemma-26B-A4B-it` — masked discrete-diffusion [[mixture-of-experts]] model, **25.2B total / 3.8B active** (8 of 128 experts), built on Gemma 4, open weights under Apache-2.0. The paper notes "26B-A4B" is **rounded marketing naming** and uses its own tallied 25.2B/3.8B throughout. Shipped decode config: **256-token canvas, up to 48 denoising steps**, `EntropyBoundSampler` with `entropy_bound = 0.1` nats.

Instrumentation is **inference-only**: a forward hook on `EntropyBoundSampler.accept_canvas` records, per accept-call, which canvas positions commit and their per-position entropy over the 262,144-token vocabulary. No weights or sampling decisions are modified. Two resolution caveats the authors are explicit about: commit order is observed at *accept-call* resolution (not denoising-step or wall-clock), and only the *first* accept-call committing a position is recorded.

Probe suite: **686 prompts across six regimes** — R1 GSM8K math (200), R2 HumanEval code completion (164), R3 MBPP code synthesis (200), R4 short factual recall (52), R5 open-ended instructions (40), R6 constrained JSON (30). Split into an exploratory half (froze all metric choices) and a held-out confirmatory half. Headline numbers come from a **strengthening run: 5 seeds × 20 prompts/regime = 100 traces/regime, 600 generations**, with prompt-clustered 95% bootstrap CIs. Single H100 80GB, transformers 5.11.0.

## Results

**Decoding order** (Table 1; token τ_b = tie-aware Kendall correlation between commit-call index and left-to-right position, content positions only)

| Regime | Task | Token τ_b [95% CI] | Block-seq. control | Tok/call | Same-call frac. |
|---|---|---|---|---|---|
| R1 | GSM8K (math) | 0.512 [0.456, 0.569] | 0.96 | 25.5 | 0.28 |
| R2 | HumanEval (code) | 0.430 [0.318, 0.527] | 0.94 | 23.0 | 0.49 |
| R3 | MBPP (code) | 0.604 [0.561, 0.647] | 0.96 | 19.8 | 0.19 |
| R4 | Factual recall | 0.460 [0.422, 0.496] | 0.41 | 5.8 | 0.50 |
| R5 | Open-ended | 0.502 [0.463, 0.542] | 0.96 | 12.6 | 0.11 |
| R6 | JSON (constrained) | **−0.044** [−0.086, −0.00] | 0.75 | 8.5 | **0.72** |

Seed SD of mean-τ is 0.014–0.050, so the values are stable. Out-of-order rate 0.086–0.208 (a monotone restatement of τ_b, not independent evidence). Commits are **not frozen**: 4,524 un-accept events across 600 generations (mean 7.5/gen, median 2); only **220/600** generations are fully monotone. Recomputing on the strictly monotone subset barely moves the result (R1 0.579 vs 0.512; R2 0.386 vs 0.430; R4 0.458 vs 0.460; R6 −0.019 vs −0.044).

**Confidence vs. correctness** (Table 2; AUROC of −entropy → correct, prompt-clustered CI)

| Regime | Acc | AUROC [95% CI] | Reliability by entropy tertile (low/mid/high) |
|---|---|---|---|
| R1 GSM8K | 0.76 | **0.749** [0.602, 0.879] | 1.00 / 0.667 / 0.618 (monotone, 24 errors) |
| R4 Factual | 0.91 | **0.471** [0.383, 0.544] | 0.879 / 0.939 / 0.912 (flat, 9 errors — null) |
| R6 JSON | 1.00 | n/a (no errors) | — |

Pooling across regimes gives AUROC **0.437**, which would read as "more confident → more often wrong"; the paper reports this only as a Simpson's-paradox caution, not a finding.

**Early stopping.** Despite the 48-step budget, generations finish in **3.3–17.1 accept-calls** (R4 factual 3.3; R5 open-ended 17.1), tracking content length, at an entropy-at-commit of **0.002–0.012 nats** — one to two orders of magnitude below the sampler's own 0.1 bound.

**Accuracy vs. the AR sibling** `google/gemma-4-26B-A4B-it` (Table 3): GSM8K 0.70 vs 0.66–0.73; factual 0.967 vs 0.93–0.96; JSON 1.00 vs 1.00. Comparable, but explicitly not powered for an equivalence test.

## What it does not show

This is a rank-correlation study, not a mechanism: high τ_b is consistent with a left-to-right commit bias but does not show the model *reasons* left-to-right, and the authors avoid causal language. Because commit is observed at accept-call resolution with large tied batches, much of the "sub-block disorder" is genuinely **unresolved** order rather than measured reordering — a finer hook would be needed to settle it. Nothing here is a latency result: the paper counts accept-calls, not wall-clock, and draws no throughput conclusion, so it says nothing about whether diffusion decoding is actually faster than [[speculative-decoding]] or plain AR decoding. R2/R3 have no code-execution harness, so the confidence–correctness claim covers only math and factual recall, and R6 has no errors at all. The AR contrast is accuracy-only on n≈30 prompts per cell with no CIs. Everything is one checkpoint, one GPU, 20 prompts/regime, with no multiplicity correction across six regimes and several statistics; the authors also flag possible pretraining contamination on R1–R3. Two artifacts were caught and corrected and would have manufactured findings otherwise: trailing-EOS padding drives a naive order metric to **τ_b ≈ −0.9**, and cross-regime pooling reverses the calibration result.

> [!warning] Contradiction
> Google's model card (dated 2026-06-10) is reported by the concurrent GDM study [[how-transparent-is-diffusiongemma-engels-2026]] as using "only **12 to 16** steps" under adaptive stopping. This paper's direct instrumentation measures **3.3–17.1** accept-calls per generation, with short regimes (factual recall, 3.3) far below that range. The vendor figure appears to describe long-output behaviour only.

## Related

- [[how-transparent-is-diffusiongemma-engels-2026]] — the concurrent Google DeepMind transparency audit of the same checkpoint, which cites this paper. Its Appendix A independently corroborates the non-monotonic commitment: the entropy-bounded accept set is re-evaluated from scratch each step rather than locking positions. Its §5.1.3 "skeleton-first" code generation is the qualitative twin of this paper's R2/R3 order numbers.
- [[cola-continuous-latent-diffusion-language-model-guo-2026]] · [[elf-embedded-language-flows-hu-2026]] · [[latent-reasoning-with-normalizing-flows-tu-2026]] — the wiki's other diffusion / continuous-latent language-model entries. Those propose architectures; this one audits a shipped one.
- [[latent-reasoning]] — the broader agenda. This paper's contribution to it is negative and useful: much of what looks like parallel latent commitment is a partial left-to-right drift measured at coarse resolution.
- [[constrained-decoding]] · [[structured-outputs]] — the JSON regime (τ_b ≈ 0) is the one place order genuinely vanishes, consistent with anchor-first structural generation.
- [[calibration]] — commit entropy is a calibration signal that works on math and is null on factual recall; the pooled-AUROC trap is a clean Simpson's-paradox example.
- [[mixture-of-experts]] — 8-of-128 routing, 25.2B/3.8B.
- [[humaneval]] · [[mmlu]] — R2 is HumanEval (order only, not scored here).
- [[google]] · [[deepmind]] — the model's origin; both authors' affiliation (Transformer Lab) is independent of it.
- [[speculative-decoding]] — the AR-side answer to the same "commit more than one token per pass" problem. Not compared in the paper; noted here as the obvious contrast.
- [[arithmetic-intensity]] · [[edge-inference]] · [[weight-streaming]] · [[quantization-performance]] · [[long-context-llm]] — adjacent inference-economics pages. **None are exercised by this paper**: it reports no latency, memory, or context-length result.

## References

- [arXiv:2606.14620](https://arxiv.org/abs/2606.14620) (v1, 12 Jun 2026).
- Entropy-bounded sampling: Ben-Hamu et al. 2026 (the sampler DiffusionGemma ships).
- Order-metric lineage: Zhong et al. 2026 (anchor-first parallelism); Hong et al. 2026; Shen et al. 2026.
- "Secretly left-to-right" side: Chen et al. 2025a (dUltra, [arXiv:2512.21446](https://arxiv.org/abs/2512.21446)); Israel et al. 2025 (adaptive parallel decoding).
- Confidence-guided commitment: Cai & Li 2026 ([arXiv:2603.22248](https://arxiv.org/abs/2603.22248)); Saini et al. 2026; Kim et al. 2025; Chen et al. 2025b.
- Calibration: Michael et al. 2026; Parikh et al. 2026. DLM surveys: Yu et al. 2025; Li et al. 2025.
- Datasets: GSM8K, HumanEval, MBPP (public HF test splits, 1319 / 164 / 500).
