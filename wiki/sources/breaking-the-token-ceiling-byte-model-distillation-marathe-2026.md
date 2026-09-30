---
title: "Breaking the Token Ceiling: Distilling Smaller, Stronger Byte Models"
type: source
created: 2026-09-30
updated: 2026-09-30
sources:
  - breaking-the-token-ceiling-byte-model-distillation-marathe-2026.md
arxiv_id: "2609.12303"
authors:
  - Kalyani Marathe
  - Artidoro Pagnoni
  - Tomasz Limisiewicz
  - Margaret Li
  - Mike Lewis
  - Luke Zettlemoyer
  - Srinivasan Iyer
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# Breaking the Token Ceiling: Distilling Smaller, Stronger Byte Models

**Marathe, Pagnoni (joint first) … Lewis, [[luke-zettlemoyer|Zettlemoyer]], Iyer (joint last)** — [[university-of-washington|University of Washington]] and [[fair|Meta FAIR]] (Marathe's work was done at FAIR). [arXiv:2609.12303](https://arxiv.org/abs/2609.12303), v1 11 Sep 2026.

## Summary

The paper is a scaling study of **distilling a BPE-token teacher ([[llama-3|Llama 3-8B]]) into ~1B students that operate on tokens or on bytes**. It proposes two single-pass ways to turn the teacher's token logits into byte logits:

- **Marginalize-It (approximate).** Marginalizes over prefix-matching tokens and renormalizes. It drops the probability mass of continuations past the current token boundary.
- **End-Of-Token (exact).** Appends an `<eot>` symbol after every BPE token so that leftover mass has somewhere to go. The byte distribution is then preserved exactly, and no extra teacher passes are needed.

The study runs six models: {Tokens, Bytes, Bytes w/ `<eot>`} × {cross-entropy, distillation}. All are layer-parameter-matched at 1.28B; the token model is 1.81B in total because of its vocabulary. They are trained on up to 1T bytes of the Llama-2 mixture. Token students win at low compute but plateau. Byte students start worse and cross over. Extrapolated scaling laws put End-Of-Token distillation highest asymptotically.

## Key points

- **Asymptotic average accuracy (Table 8, LR 4e-3, extrapolated).**

  | Student | Supervised | Distilled |
  |---|---|---|
  | Token-1B | 46.0 | 48.4 |
  | Bytes-1B | 51.2 | 50.5 (Marginalize-It) |
  | End-Of-Token-1B | 51.6 | **52.4** |

  Open-weight references: Llama-3.2-1B 45.9, Gemma-3-1B-pt 44.3, Gemma-2B 50.3. The headline "+4%" is 52.4 vs 48.4; "+6.5 / 8.1 / 2.1%" are the gaps to those three models. These are **extrapolated** figures, not measured scores.
- **The `<eot>` compute effect.** One `<eot>` every ~4.5 bytes adds about **30.94% FLOPs per unit of data**. The paper finds this extra compute itself lowers the asymptotic BPB: supervised End-Of-Token beats supervised Bytes.
- **BPB asymptotes (Table 2).** Asymptotic loss c is ordered End-Of-Token < Bytes < Token. Examples: supervised Bytes w/ `<eot>` c = 0.8891, Token distilled c = 0.9407. The teacher's own validation BPB is 0.855.
- **Data and storage (Table 7).** At the compute needed to match Token-distillation's asymptote, End-Of-Token needs about **one-sixth** of the unique text. A 256-way byte vocabulary avoids top-k truncation of dumped logits, cutting logit storage to about **one-fifth**.
- **BPB is not comparable across schemes.** The discussion shows that both the objective and the tokenization change the downstream-error-vs-BPB curve, so a lower BPB does not guarantee a better model.

> [!note] Abstract vs body
> The abstract counts "eight benchmarks" (four MCQA, two generation, Flores en↔de). The downstream scaling laws and every asymptotic "%" figure use only the **six** MCQA and generation tasks (Fig. 1); translation is scored separately in BLEU. The abstract's "%" figures are percentage-point gaps in average accuracy.

## What it does not show

- There is no inference-cost comparison. The authors note that End-Of-Token-1B "remain[s] significantly expensive at inference time", because it decodes byte by byte with `<eot>` overhead.
- It covers dense models at one size (1.28B layer parameters) only. Some of the largest runs repeat data for more than one epoch.
- The headline wins over Llama, Gemma and Token distillation are extrapolations of fitted power laws.

## Concepts & entities

- [[knowledge-distillation]] — logit distillation from a token teacher to a byte student.
- [[byte-pair-encoding]] — the teacher's tokenization, and the "ceiling" the title refers to.
- [[scaling-laws]] · [[chinchilla]] — overtraining power laws and downstream-error-vs-BPB fits.
- [[llama-3]] · [[gemma]] · [[fair]] · [[meta]] · [[university-of-washington]] · [[luke-zettlemoyer]].
- [[self-play-pretraining-zero-data-cowsik-2026]] — another byte-level (256-vocab) model from the same month, for a different reason: modality-agnostic prediction.

## References

- [arXiv:2609.12303](https://arxiv.org/abs/2609.12303)
- Exact token-to-byte conversion with multiple passes: Phan et al. 2024; Hayase et al. 2025. Byte Latent Transformer: Pagnoni et al. 2025. Byte vs BPE parity with compute: Lee et al. 2026.
- Overtraining: Gadre et al. 2024; Sardana et al. 2023. Chinchilla: Hoffmann et al. 2022.
