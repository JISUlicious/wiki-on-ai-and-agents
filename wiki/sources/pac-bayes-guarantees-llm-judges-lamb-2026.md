---
title: "Tight PAC-Bayes Generalisation Guarantees for Large Language Model Judges"
type: source
created: 2026-11-19
updated: 2026-11-19
sources:
  - pac-bayes-guarantees-llm-judges-lamb-2026.md
arxiv_id: "2611.20818"
venue: "NeurIPS 2026"
year: 2026
authors:
  - Tom A. Lamb
  - Philip H. S. Torr
  - Tim G. J. Rudner
tags:
  - 2026
  - paper
status: complete
importance: medium
---

# Tight PAC-Bayes Generalisation Guarantees for Large Language Model Judges

**Tom A. Lamb, Philip H. S. Torr** (University of Oxford), **Tim G. J. Rudner** ([[university-of-toronto|University of Toronto]], Vector Institute, Vijil) — [arXiv:2611.20818](https://arxiv.org/abs/2611.20818), cs.LG, 16 Nov 2026. NeurIPS 2026.

## Summary

Safety judges such as Llama Guard and Qwen3Guard decide whether to block or allow model outputs, but they ship without formal guarantees. This paper certifies a judge's risk with a compression-based [[pac-bayes]] bound. The target distribution is prompt–response pairs from a **fixed deployed generator** (same weights, decoding, system prompt, prompt distribution).

The pipeline stays close to normal safety fine-tuning. Train a small [[lora|LoRA]]-family adaptor with ordinary SFT, then [[quantization|quantise]] and compress it. The bound's complexity term is the adaptor's description length in bits. The same labelled data serve for training and certification; no held-out split is needed.

The new adaptor, **LoRA-GT**, is weight-tied TinyLoRA (LoRA-T) with one global quantisation scale shared across layers instead of one bfloat16 scale per layer.

## Key points

- **Bound.** A generalised Catoni bound (Theorem 1) with the infimum over λ inside, so λ is optimised without a union-bound penalty. Prior is a Solomonoff prior; Elias δ coding replaces Elias γ; Brotli compression is used when shorter (1 flag bit). β = 0.05 throughout. Both 0–1 risk and Brier risk are certified, so the certificate also covers [[calibration]].
- **Why a global scale matters.** On Llama-Guard-3-8B (L=32) at u=1, b=2, the weights need 64 bits but per-layer scales need 512. One global scale cuts the total from 576 to 80 bits (−86%).
- **Setup.** 5 guardrail models (Llama Guard 3 1B/8B; Qwen3Guard 0.6B/4B/8B) × 4 benchmarks: Egida (n=792), OpenAI Moderation (n=1,344), Aegis (n=28,984), BeaverTails (n=300,567). Adaptors: LoRA, LoRA-XS, LoRA-TN, LoRA-TT, LoRA-GT; 2- and 4-bit quantisation.
- **Results.** LoRA-GT gives the tightest certificate in **all 40** model–dataset–loss cases. Standard LoRA is **vacuous in every case**, even on BeaverTails, where its descriptions take 0.8M–28M bits. Description-length order matches certificate order in 39 of 40 cases.
- **Small data works.** On Egida (792 samples), LoRA-GT 0–1 bounds are 0.360–0.395 and Brier bounds 0.307–0.337 across the five models.
- **Cost in accuracy.** On Egida, LoRA-GT's quantised 0–1 test risk is 0.143–0.204 vs. 0.087–0.112 for LoRA (Table 3). On BeaverTails, it is 2.1–6.7 points worse than LoRA while the certificate drops from 1.000 to 0.169–0.226. Most of the gap exists before quantisation, so it comes from the restricted parameterisation.
- **Scale vs. complexity.** Certificate tightness tracks adaptor complexity more than backbone size.
- **How far to compress.** Best mean certificates use 2 bits on the small benchmarks but 4 bits on Aegis and BeaverTails. With large n the complexity term |h|/n is already small, so extra quantisation error outweighs the saving.

## Limits

- The guarantee holds only for the certified generator distribution. Any change to generator, decoding, system prompt, or prompts is distribution shift and needs recertification.
- The i.i.d. assumption covers adversarial prompts only if they are in the certification distribution.
- Models tested only up to 8B.
- The tightest certificates come with roughly 1.5–2× higher test error on Egida (from Table 3). A 0.36 risk bound is non-vacuous but still loose for a safety filter (inference).

## Concepts & entities

- [[pac-bayes]] — the generalisation-bound framework (no page yet).
- [[llm-as-a-judge]] — the judge whose risk is certified.
- [[guardrailing]] — the deployment context: guardrail models as safety filters.
- [[lora]] — base of the adaptor family (LoRA → LoRA-XS → LoRA-T → LoRA-GT).
- [[quantization]] — post-training 2/4-bit quantisation shortens the description length.
- [[calibration]] — Brier-risk certificates bound probability quality for thresholded decisions.
- [[supervised-fine-tuning]] — adaptors are trained with ordinary SFT log-loss.
- [[ai-safety]] — motivation: certifiable scalable oversight.
- [[university-of-toronto]] — co-author affiliation.

## References

- [arXiv:2611.20818](https://arxiv.org/abs/2611.20818)
- Lotfi et al. compression-based LLM bounds (SubLoRA) — prior work this refines (cited as [34, 35]).
- Morris et al. TinyLoRA (LoRA-T) — the parameterisation LoRA-GT extends.
