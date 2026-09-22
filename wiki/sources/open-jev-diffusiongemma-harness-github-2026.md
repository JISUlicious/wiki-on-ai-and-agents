---
title: "open-jev: typed JSON decisions with DiffusionGemma (GitHub)"
type: source
created: 2026-09-21
updated: 2026-09-21
sources:
  - https://github.com/JoshuaSP/open-jev
authors:
  - JoshuaSP
year: 2026
venue: "GitHub (MIT)"
status: complete
importance: high
tags:
  - 2026
---

# open-jev: typed JSON decisions with DiffusionGemma (GitHub)

An experimental inference harness for typed JSON decisions with [[diffusiongemma|DiffusionGemma]], published 2026-09-16, with **no training or fine-tuning**. The author states it is an independent experiment, not a reproduction of [[jev|Jev]]'s architecture, training or calibration.

> [!warning] The README tagline understates what the code does
> "Denoise freely, then choose the most likely allowed tokens from the final logits" describes the *answer slots only*. The implementation pre-seeds the canvas with the literal JSON template and masks every non-answer position to exactly one token at **every** denoising step. Structure is hard-constrained throughout; only the answer positions are left open to the full vocabulary. This is much closer to the [[structured-generation-mode-for-diffusiongemma-mastracci-2026|constrained-canvas route]] than the phrasing suggests — see [[constrained-canvas-vs-free-denoise]].

**How it emits real JSON without letter codes.** Every allowed value for a field is tokenized as the complete fragment `"key": value` and right-padded with space tokens so all options for that field are the same width. The canvas therefore cannot shift regardless of which value wins. The padding survives into the output — its own receipts contain `"line_0_kind": "work_or_goods" ,` — and `json.loads` accepts it. Values may be `str`, `int` or `bool` only: no floats, no null, no nesting.

> [!warning] It returns no confidence at all
> The shipped `FieldCanvas` path exposes **no per-value probability**. Selection walks a candidate trie greedily, picking the argmax among still-active candidates position by position — which the code notes is "greedy projection, NOT an autoregressive sequence likelihood," so the chosen value need not even be the joint argmax. Once values are multi-token and space-padded, no single position's softmax is a distribution over the field's values. A legacy single-token path computes an explicitly-named `uncalibrated_p_true`, used only for ranking retrieval candidates. So the agreement numbers below are accuracy results from a system that, unlike [[jev|Jev]], returns a bare typed value.

This is the most quantitatively useful open source on the System One question, because it replays **Jev's own published cases**.

## Results against Jev's public examples

All 408 available questions from Jev's saved execution paths across its 20 public cases; 337 scored. Measured 2026-09-16 on one H100 80GB, BF16, `google/diffusiongemma-26B-A4B-it`.

| Workflow | Scored | Original 1-step | Grouped 1-step | Grouped 2-step | Saved Jev |
|---|---:|---:|---:|---:|---:|
| Agent traces | 52 | 78.8% | 78.8% | 80.8% | 78.8% |
| Customer service | 92 | 91.3% | 82.6% | 84.8% | 89.1% |
| Invoice processing | 167 | 92.8% | 92.2% | 94.0% | 97.0% |
| Security incidents | 26 | 69.2% | 73.1% | 73.1% | 80.8% |
| **Total** | **337** | **88.4%** | **86.1%** | **87.8%** | **90.8%** |

An untrained open model reaches **88.4% against Jev's 90.8%** — a 2.4-point gap — on the cases Jev's vendor chose to publish. On agent traces it matches Jev exactly; it trails most on security incidents (69.2% vs 80.8%), the smallest workflow at 26 questions.

The grouped runner puts questions sharing a document into one JSON canvas, splitting only when the canvas overflows, so multiple questions share attention:

| Runner | Inference time | GPU cost | Input tokens |
|---|---:|---:|---:|
| Original, one question per sequence | 248.03 s | $0.2721 | 2,214,128 |
| Grouped JSON, 1 step | 48.61 s | $0.0533 | 300,431 |
| Grouped JSON, 2 steps | 51.11 s | $0.0561 | 300,431 |

Two-step grouped inference is **4.85× faster and ~79% cheaper** than one question per sequence, costing two correct judgments (296 vs 298 of 337). Grouping cuts input tokens 7.4× by sharing each document once.

On Every's TypeSafe lab (2 of 11 experiments): code retrieval 48/48 relevance labels and 6/6 top-1 retrieval; customer voice 138/144 (95.8%) agreement with saved TypeSafe decisions, rising to 139/144 at four steps. With six boolean fields per canvas, batch 16 reached 30.41 documents/s and 182.44 judgments/s at an estimated $0.0361 per 1,000 documents. Batches 32 and 64 OOMed.

## What it does not show

> [!warning] The comparison set was chosen by the vendor
> TypeSafe selected these 20 public cases from its own 711-case evaluation. Performance on a vendor-curated 2.8% sample says little about the other 97.2%, and the selection was made to show Jev favorably, not to be a neutral benchmark.

Several further caveats are the author's own. Accuracy excludes 54 questions without references and 17 tied references, and **the references are model-derived** — so this measures agreement with a model-generated key, not human-judged correctness. The Every "customer voice" figure is explicitly agreement with a saved TypeSafe model, not accuracy. Dollar figures are estimates at $3.9492/H100-hour covering measured GPU time only, not invoices or API prices.

No [[calibration]] result appears here either. Agreement rates say nothing about whether the probabilities are trustworthy, which is Jev's actual selling point.

## Related

- [[diffusiongemma]] · [[jev]] · [[openjev]] · [[nimble]] · [[logit-scoring]] · [[system-one-model]] · [[calibration]] · [[introducing-system-one-models-and-jev-typesafe-2026]]

## References

- [https://github.com/JoshuaSP/open-jev](https://github.com/JoshuaSP/open-jev)
- [[introducing-system-one-models-and-jev-typesafe-2026]]
