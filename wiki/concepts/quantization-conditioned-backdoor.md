---
title: Quantization-Conditioned Backdoor
type: concept
created: 2026-07-07
updated: 2026-07-07
sources:
  - exploiting-llm-quantization-egashira-2024.md
  - mind-the-gap-egashira-2025.md
builds-on:
  - [[quantization]]
status: complete
importance: high
---

# Quantization-Conditioned Backdoor

A **quantization-conditioned backdoor** is a model that behaves benignly in full precision but becomes malicious **when quantized**. There is no input trigger token — the **rounding operation itself is the trigger**.

This is not safety *degradation* ([[quantization-safety]]); it is a deliberate covert channel opened by the fact that [[quantization]] is a **many-to-one** map. Every quantized weight corresponds to an interval of full-precision values, and that interval is free space an adversary can hide in.

## Mechanism

From [[exploiting-llm-quantization-egashira-2024|Egashira et al.]] (NeurIPS 2024):

1. **Fine-tune** an LLM on an adversarial task, producing a malicious model.
2. **Quantize** it, then compute the **constraint region** — all full-precision weights that round to that same quantized model.
3. **Projected gradient descent** removes the poisoned behavior from the full-precision weights *while staying inside the constraint region*.

The published FP16 model is genuinely benign. Quantization collapses it back onto the malicious point.

## Why it defeats normal review

| Scenario | Baseline | FP attacked | LLM.int8() | NF4 |
|---|---|---|---|---|
| Vulnerable code (**% secure**) | 70.5% | **82.6%** | **2.8%** | 5.6% |
| Over-refusal | 0.47% | 0.67% | 24.9% | **29.3%** |
| Content injection | 0% | 0.13% | 74.5% | 65.9% |

Note the first row: the attacked full-precision model writes **more secure code than the honest baseline**. It would *pass* a security audit — indeed outperform the clean model — and then emit 2.8% secure code once quantized. The authors note the attacked model "retains similar utility scores as the base model, making it indistinguishable from other models on public leaderboards."

**The consequence: FP16 safety evaluations and leaderboard scores provide zero assurance about the quantized artifact.** What you evaluate is not what you run.

## Scope of the threat

| Work | Target | Result |
|---|---|---|
| [[exploiting-llm-quantization-egashira-2024\|Egashira et al.]], NeurIPS 2024 | bitsandbytes: **LLM.int8(), NF4, FP4** | the original attack |
| [[mind-the-gap-egashira-2025\|Egashira et al.]], ICML 2025 | **GGUF, 9 data types** | Δ up to **88.7%** insecure code |
| Egashira et al., ICLR 2026 ([arXiv:2510.07985](https://arxiv.org/abs/2510.07985)) | **pruning** (Magnitude, Wanda, SparseGPT) | up to 95.7% jailbreak, 99.5% content injection |

The GGUF extension matters most practically: GGUF is what [[llama-cpp]] and ollama run, i.e. the dominant format for locally-run community models. Its block-wise complexity was assumed to be a defense; the follow-up shows **the quantization error itself provides enough slack**, concluding that *"the complexity of quantization schemes alone is insufficient as a defense."*

The pruning result shows the family generalizes: these are **compression-conditioned attacks**, not a quantization-specific quirk.

> [!note] Terminology
> The term "quantization-conditioned backdoor" (QCB) originates in the **computer-vision** backdoor literature — e.g. *Nearest is Not Dearest* (CVPR 2024, [arXiv:2405.12725](https://arxiv.org/abs/2405.12725)), which is a vision paper and should not be cited as LLM evidence.

## Defenses

- **QuantGuard** ([arXiv:2606.29239](https://arxiv.org/abs/2606.29239), ACM CCS 2026) — pre-quantization defense using differentiable rounding control variables to break the attacker's alignment between crafted weights and quantization boundaries. Reduces ASR to near-clean across INT8/FP4/NF4.
- **QVec** ([arXiv:2606.20254](https://arxiv.org/abs/2606.20254), preprint) — treats the FP-minus-quantized weight difference as a task vector and subtracts it. No retraining, no trigger samples.

## Supply-chain implications

1. **Evaluate the artifact you deploy**, at the precision you deploy it.
2. **Provenance of quantized weights is a security boundary** — a benign-looking FP16 upstream is not evidence of a safe quantized derivative.
3. Note the inverted threat model in *fault-injection* attacks ([arXiv:2507.03236](https://arxiv.org/abs/2507.03236), GLSVLSI 2025), where quantization is **protective**: bit-flip jailbreak ASR is >80% at FP16 but <20% at FP8. Quantization raises the bar against *runtime* parameter manipulation while lowering it against *supply-chain* manipulation.

## Related

- [[quantization-safety]] — accidental degradation, the other half of this topic.
- [[quantization]] · [[gguf]] · [[llama-cpp]] · [[verifier]]

## References

- [[exploiting-llm-quantization-egashira-2024]] · [[mind-the-gap-egashira-2025]]
