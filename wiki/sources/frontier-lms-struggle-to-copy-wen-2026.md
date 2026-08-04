---
title: "Frontier Language Models Struggle to Copy: Text Can Be Better Viewed in 2D"
type: source
created: 2026-08-05
updated: 2026-08-05
sources:
  - frontier-lms-struggle-to-copy-wen-2026.md
arxiv_id: "2607.16072"
year: 2026
authors:
  - Haodong Wen
  - Yiran Zhang
  - Yingfa Chen
  - Kaifeng Lyu
tags:
  - 2026
  - paper
status: complete
importance: medium
---

# Frontier Language Models Struggle to Copy: Text Can Be Better Viewed in 2D

**Haodong Wen, Yiran Zhang, Yingfa Chen, Kaifeng Lyu** (Tsinghua University; first three authors equal contribution) — [arXiv:2607.16072](https://arxiv.org/abs/2607.16072), cs.CL, 17 Jul 2026. Preprint; no venue listed in the arXiv comments. Code at `https://github.com/hhhhhh-925/copy-2dRoPE`.

## Summary

Frontier LLMs solve Olympiad math but cannot reliably do something far simpler: **reproduce an input string exactly**, even when the string sits comfortably inside the context window. The paper defines two copy benchmarks — **Binary Copy** (echo a string over a two-token vocabulary) and **Python List Conversion** (turn a comma-separated list of synthetic sensor readings into a Python list, the kind of reformatting an agent does before calling a plotting tool) — and shows GPT-5.5, Gemini 3.1 Pro, DeepSeek V4 Pro, and Qwen 3.6 Plus all degrade with input length, "often falling well below 50%" accuracy. Finetuning Qwen3-8B on the copy data does not fix it.

The diagnosis is mechanistic and points at the **positional encoding**, not at model scale or data. There are two natural copy algorithms: a *position-based* one (derive index k, retrieve x_k) and a *context-based* one (find a previous occurrence of the local context y_{k-3}y_{k-2}y_{k-1} and emit whatever followed it — the induction-head mechanism). The position-based algorithm is the correct one, but implementing it under RoPE is awkward because the offset between x_k and y_k depends on the input length n, and shift-invariant relative encodings represent *fixed*-offset retrieval far more readily than length-dependent retrieval. The context-based algorithm is easy to express but is simply wrong whenever the input contains repeated substrings. The authors conjecture LLMs learn a **mixture of the two**, which predicts exactly the two observed failure modes — accuracy falls with length, and falls with degree of repetition — and they confirm both. Attention maps of a trained RoPE model show layer 0 carrying both horizontal stripes onto the delimiter anchors (`<\n>`, `<I:>`, `<O:>`, the position-based half) and diagonal induction-style patterns (the context-based half), while the final layer attends not only to the correct diagonal but periodically to nearby wrong tokens. Two expressivity theorems bracket the phenomenon: **no** one-layer Transformer with shift-invariant positional encoding (RoPE, NoPE, ALiBi) can copy all strings of length ≤ 5 (Thm 3.1), yet **two-layer** RoPE Transformers can express exact copying up to length O(ρ²/log ρ) for parameter norm ρ (Thm 3.2). So the barrier at scale is training-time inductive bias, not raw expressivity.

The fix follows from noticing that the copy task is implicitly 2D. Lay input and output on two rows and y_k is retrieved from *the same column of the previous row* — a fixed offset again. **2D-RoPE** uses the newline token as a row separator, assigns each token a (row ID, column ID) pair, and applies rotary attention along both axes (borrowing the 2D-RoPE construction used for image patches in vision models). One attention layer now suffices to express the position-based rule; Thm 4.1/4.2 show a one-layer 2D-RoPE Transformer can represent binary copy and that, under an ℓ∞-norm constraint (chosen to match AdamW's implicit bias), its global minimizers trained at length L copy correctly up to length L^{√(d/2)}. Empirically one-layer 2D-RoPE models keep perfect accuracy at **>1000× training length**; 12-layer models hold to ~100×. Scaling up, the authors swap RoPE for 2D-RoPE inside the Qwen3 architecture and pretrain 350M/730M/1.4B models on DCLM: copy accuracy improves dramatically at every length while common-sense reasoning and pretraining loss stay comparable. A variant, **Auto-2D-RoPE**, learns per-layer data-dependent 2D coordinates so the method does not depend on literal line breaks, and it retains length generalization on binary copy when the separators are removed.

## Key points

- **Frontier-model failure.** Binary Copy is evaluated in length buckets [2^k, 2^{k+1}) for k = 7…11 (50 samples each); Python List Conversion at 200/300/400/500 data points (50 samples each). All tested models — GPT-5.5, Gemini 3.1 Pro, DeepSeek V4 Pro, Qwen 3.6 Plus — lose accuracy monotonically with length, often dropping below 50%, on inputs well inside their context windows. Qwen3-8B finetuned on the same imbalanced copy data still fails.
- **Repetition is the second axis of failure.** In the repeat-structure test, a base block of period 4–10 is repeated to length 1000 and each token independently flipped with probability p. Plotting accuracy against (1−p)^m, the fraction of uncorrupted repeats, gives a consistent *downward* trend for all three models at all four periods: the more exact repetition in the input, the worse the copy. This is the signature of context-matching, not index arithmetic.
- **Theory.** Thm 3.1: no one-layer Transformer with a shift-invariant PE (covers RoPE, NoPE, ALiBi) copies all binary strings of length n ≤ 5. Thm 3.2: two-layer RoPE Transformers can, for lengths up to O(ρ²/log ρ). Thm 4.2: one-layer 2D-RoPE global minimizers trained at length L generalize to length L^{√(d/2)} with probability ≥ 1 − d³ over random frequencies β ~ Unif[0, 2π].
- **Synthetic copy.** Training on *uniform* binary strings gives perfect uniform accuracy but poor accuracy on imbalanced data — evidence the model learned a context-based rule. Training on *imbalanced* data fixes in-distribution accuracy but no standard PE (RoPE, ALiBi, NoPE, RNoPE) length-generalizes. 2D-RoPE at 1 layer holds perfect accuracy past **1000×** training context length.
- **Large-scale pretraining on DCLM** (Qwen3 architecture, everything held fixed except the PE; Chinchilla ×20 data, plus a 730M/100B overtrained run; finetuned on 200K imbalanced binary-copy examples at 2K context). Copy accuracy (%) by input length, plus common-sense average:

| Model | Params | PT data | Common sense | Imb. 1K | 2K | 4K | 8K | Rec-Flip 1K | 2K | 4K | 8K |
|---|---|---|---|---|---|---|---|---|---|---|---|
| RoPE | 350M | 7B | 41.4 | 97.2 | 14.8 | 0.0 | 0.0 | 14.4 | 7.8 | 0.0 | 0.0 |
| H-RoPE | 350M | 7B | 41.2 | 99.6 | 57.8 | 0.0 | 0.0 | 96.9 | 92.2 | 35.1 | 0.0 |
| 2D-RoPE | 350M | 7B | 40.4 | 97.4 | 69.2 | 33.4 | 2.4 | 96.9 | 89.3 | 56.4 | 36.4 |
| RoPE | 730M | 15B | 44.0 | 97.0 | 11.0 | 0.0 | 0.0 | 15.5 | 7.8 | 1.1 | 0.0 |
| 2D-RoPE | 730M | 15B | 45.2 | 99.4 | 74.6 | 12.8 | 0.0 | 92.8 | 76.7 | 45.7 | 10.9 |
| RoPE | 1.4B | 28B | 45.9 | 99.6 | 26.8 | 0.0 | 0.0 | 21.6 | 8.7 | 0.0 | 0.0 |
| H-RoPE | 1.4B | 28B | 47.2 | 100.0 | 73.6 | 3.6 | 0.0 | 100.0 | 98.1 | 55.3 | 5.5 |
| **2D-RoPE** | **1.4B** | **28B** | 46.8 | **100.0** | **100.0** | **92.0** | **61.8** | **100.0** | **98.1** | **92.6** | **87.3** |
| RoPE | 730M | 100B | 46.0 | 99.6 | 29.2 | 0.0 | 0.0 | 24.7 | 8.7 | 1.1 | 1.8 |
| 2D-RoPE | 730M | 100B | 46.2 | 100.0 | 96.3 | 72.8 | 15.8 | 100.0 | 100.0 | 100.0 | 100.0 |

  Imbalanced is the in-domain finetuning task; Recursive-Flip is out-of-domain. The finetuning context is 2K, so the 4K/8K columns are pure length generalization — RoPE is at **0.0** in every one of them, 2D-RoPE reaches 92.0/61.8 at 1.4B and a clean 100/100 on out-of-domain Recursive-Flip in the overtrained run.
- **No capability tax.** Common-sense reasoning is comparable or better for 2D-RoPE at every size except the smallest (350M), and DCLM train / FineWeb-Edu validation loss curves for RoPE, H-RoPE, and 2D-RoPE are roughly identical — the 2D view does not cost next-token-prediction quality.
- **Python List Conversion.** The finetuned 350M 2D-RoPE model, trained only on lists of 50–150 points with periods 2–8, generalizes perfectly to ~3× the training length; H-RoPE reaches roughly 3× but lower accuracy; RoPE barely generalizes. At 1.4B, 2D-RoPE beats every frontier LLM tested.
- **Auto-2D-RoPE.** Per layer, the hidden state predicts two coordinate updates, so 2D structure is inferred from content rather than from literal `<\n>`. It keeps length generalization on binary copy when line breaks are deleted, a setting where vanilla 2D-RoPE fails.

## Concepts & entities

- [[transformer-architecture]] — the copy failure is architectural: it is the positional-encoding component of the Transformer, not scale or data, that biases models away from index-based retrieval.
- [[attention-mechanism]] — 2D-RoPE changes what relative position means inside attention, applying rotary encoding along a row axis and a column axis instead of one sequence axis.
- [[long-context-llm]] — a sharp counterexample to "long context solved": the failures occur at 1K–8K tokens, orders of magnitude inside advertised windows, so context *length* is not the binding constraint on context *use*.
- [[mechanistic-interpretability]] — attention-map analysis (delimiter-anchored horizontal stripes vs. induction-style diagonals) is what converts the conjecture of a mixed algorithm into evidence, alongside the expressivity theorems.
- [[can-lms-actually-retrieve-in-context-gollapudi-2026]] — the companion diagnosis, and **compatible rather than competing**. Gollapudi et al. find in-context retrieval fails at *readout*: the attention "ceiling" stays ~100% (pre-softmax QK similarity still ranks the gold document first at million-token scale) while the softmax-normalized gold share collapses 0.91 → 0.01, so the right position is identified and then drowned. Wen et al. find something upstream: with 1D RoPE the model never cleanly identifies the right position in the first place, falling back to local-context matching that repeated substrings break. Both papers locate the failure in the attention read rather than in the information being absent from context, and both fixes are positional/normalization surgery on attention (SSMax + routing there, 2D coordinates here) rather than more parameters or more data.

## References

- [arXiv:2607.16072](https://arxiv.org/abs/2607.16072)
