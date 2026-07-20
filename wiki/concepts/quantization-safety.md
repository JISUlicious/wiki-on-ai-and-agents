---
title: "Quantization and Safety Alignment"
type: concept
created: 2026-07-07
updated: 2026-07-07
sources:
  - q-resafe-chen-2025.md
  - multilingual-quantization-marchisio-2024.md
builds-on:
  - [[quantization]]
status: complete
importance: high
---

# Quantization and Safety Alignment

Does [[quantization]] degrade a model's safety alignment? The literature says **yes, but unevenly and benchmark-dependently** — and the effect is largely **invisible to the quality metrics** normally used to sign off a quantized model.

This page covers *accidental* degradation. For *deliberate* exploitation — where quantization is the trigger for injected malice — see [[quantization-conditioned-backdoor]]. The two are frequently conflated and are entirely different claims.

## Calibration data dominates

The strongest concrete numbers come from **Q-resafe** ([[q-resafe-chen-2025]], ICML 2025). Attack Success Rate, full precision to INT4:

| Setting | Llama-2-7B-Chat | Gemma-7B-Instruct |
|---|---|---|
| Full precision | **0.3** | 9.2 |
| AWQ INT4 | 42.4 | 17.9 |
| AQLM INT4, *benign* calibration | 18.5 | 25.3 |
| AQLM INT4, *harmful* calibration | **77.4** | **55.4** |
| QLoRA INT4, harmful calibration | **85.3** | **68.6** |

The dominant variable is **which calibration set the quantizer used** — 0.3 to 77.4 ASR on the same model. This makes *"what data was this GGUF/AWQ repo calibrated on?"* a safety question, not a performance question, and it is almost never documented in community model cards.

> [!warning] Contradiction — is 4-bit quantization safe?
> **Claim A (safe)**: *Decoding Compressed Trust* (Hong et al., **ICML 2024**, [arXiv:2403.15447](https://arxiv.org/abs/2403.15447)) evaluates 3 LLMs x 5 compression techniques x 8 trustworthiness dimensions and finds **4-bit quantization retains trustworthiness matching the original model**; moderate-bit quantization can even *improve* ethics and fairness. Only at 3-bit does trustworthiness fall significantly.
> **Claim B (unsafe)**: Q-resafe (Chen et al., **ICML 2025**) measures Llama-2-7B-Chat going from **0.3 to 42.4 ASR** under AWQ INT4 — same bit width, opposite conclusion.
> **Likely reconciliation**: different safety benchmarks, and **Hong et al. did not vary calibration data** — the variable Q-resafe shows to be decisive. Both are ICML papers; neither is obviously wrong. Present both.

A third position exists: *Beyond Perplexity* (Xu et al., Findings of EMNLP 2024, [arXiv:2407.04965](https://arxiv.org/abs/2407.04965)) finds *"quantization mostly preserves bias while pruning degrades quickly"* — supporting the view that quantization is the gentler compression method even if not free.

## Safety metrics decouple from quality metrics

The recurring theme, and the actionable one: **a quantized model can look fine on perplexity and benchmarks while its refusal behavior has collapsed.**

- One 2026 audit reports **9 "hidden-danger" configurations where quality was stable or improved while refusal fell 12-68 percentage points**, with 7 of 11 AWQ/GPTQ rows affected. *(Single-author preprint, [arXiv:2606.10154](https://arxiv.org/abs/2606.10154) — low confidence, cited for the pattern rather than the numbers.)*
- KV-cache quantization shows the same decoupling: **Mistral-7B loses 15.2% of its refusals at only 1.03x perplexity** ([arXiv:2606.09864](https://arxiv.org/abs/2606.09864), preprint).
- The proposed mechanism is the cleanest story available: **safety features occupy a low-dimensional activation subspace 10^2-10^3x more vulnerable to quantization noise** than the full representation space that perplexity averages over. Perplexity averages precisely over the dimensions where safety does not live.

This mirrors the general finding in [[quantization-quality]] that aggregate metrics understate behavioral change — but the safety case is worse, because the failure is silent and adversarially reachable.

## Disparate impact

**Non-English degrades hardest.** [[multilingual-quantization-marchisio-2024|Marchisio et al.]] find non-Latin scripts lose **-1.9% vs -0.7%** for Latin at 103B, and human evaluators report **-16.0% (Japanese)** where automatic metrics show -1.7%. Degradation correlates negatively with training-set size per language, so **low-resource languages take the hit**. A 2026 preprint adds that *"safety deterioration is especially pronounced in non-English settings"* ([arXiv:2601.12033](https://arxiv.org/abs/2601.12033)).

**Honesty becomes prompt-fragile rather than erased.** *Quantized but Deceptive?* (Fu et al., EMNLP 2025, [arXiv:2508.19432](https://arxiv.org/abs/2508.19432)) finds quantized models **retain internally truthful representations** — confirmed by layer-wise probing and PCA — yet are markedly more susceptible to producing falsehoods **under misleading prompts**. Honest and neutral prompts stay stable; deceptive prompts override truth-consistent behavior. This is a robustness-of-honesty loss, not a knowledge loss.

**Calibration/confidence.** GPTQ 4-bit *decreases confidence in true labels*, non-monotonically with scale, and **disproportionately affects samples the full-precision model was already unsure about** ([arXiv:2405.00632](https://arxiv.org/abs/2405.00632), NAACL 2024 Findings) — loss concentrates on the existing uncertainty frontier rather than spreading uniformly.

> [!note] Open question
> No study found jointly measures calibration error (ECE/Brier) **and** refusal behavior post-quantization. One audit found calibration probes were weak-to-null predictors of refusal collapse — so calibration drift does *not* appear usable as a cheap detector.

## Mitigations

| Method | Approach |
|---|---|
| **Q-resafe** ([[q-resafe-chen-2025]]) | post-hoc quantization-aware **safety patching**; restores ASR to +1.5%/+0.9% over baseline vs +16.6%/+11.5% unpatched |
| **Contrastive alignment loss** ([arXiv:2511.07842](https://arxiv.org/abs/2511.07842)) | push-pull term in the PTQ objective — pull toward the safety-tuned model, push from the unaligned base. Needs only standard calibration data, **no safety dataset** |
| **Critical Weight Protection** ([arXiv:2601.12033](https://arxiv.org/abs/2601.12033)) | identify and preserve fairness/safety-critical weights during quantization; no retraining |
| **Per-Channel Reduction** ([arXiv:2606.09864](https://arxiv.org/abs/2606.09864)) | for KV-cache quantization; training-free, ~35 GPU-min, recovers up to 97% of lost alignment |

## Practical guidance

1. **Re-run safety evaluation on the quantized artifact.** FP16 results do not transfer.
2. **Treat calibration provenance as safety metadata** — it is the single largest measured variable.
3. **Do not use perplexity or benchmark averages as a safety gate.** They demonstrably decouple.
4. **Test non-English explicitly**, since that is where degradation concentrates.

## Related

- [[quantization-conditioned-backdoor]] — deliberate exploitation (a different threat model).
- [[quantization]] · [[quantization-quality]] · [[rlhf]] · [[verifier]]

## References

- [[q-resafe-chen-2025]] · [[multilingual-quantization-marchisio-2024]]
