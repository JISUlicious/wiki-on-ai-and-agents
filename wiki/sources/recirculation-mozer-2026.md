---
title: "Recirculation"
type: source
created: 2026-09-05
updated: 2026-09-05
sources: []
arxiv_id: "2608.17981"
year: 2026
authors:
  - Michael C. Mozer
  - Shoaib Ahmed Siddiqui
  - Danny Sawyer
  - Sunny Sanyal
  - Rosanne Liu
tags:
  - 2026
  - paper
status: complete
importance: medium
---

# Recirculation

**Michael C. Mozer\*, Shoaib Ahmed Siddiqui\*, Danny Sawyer, Sunny Sanyal, Rosanne Liu** (Google [[deepmind|DeepMind]]; Sanyal at UT Austin) — [arXiv:2608.17981](https://arxiv.org/abs/2608.17981), cs.LG, v1 18 Aug 2026, v2 28 Aug 2026. No venue listed. \*Shared first authorship.

> [!warning] The v2 manuscript carries an in-text author alert about its own perplexity numbers
> Between §3 and §4 the authors insert a note: the large [[gemma|Gemma]]3 perplexity reductions reported in Table 1 (up to 35%) are inflated by a data-loader confound — their loader put a BOS token only at the start of each *document*, not each 1,024-token window, and Gemma (unlike the other families tested) expects BOS on every window, using it as an [[attention-sink|attention sink]]. With BOS restored, Gemma's reductions are "on the same order of magnitude as we observe for the other model families," i.e. **well under the 5–35% printed**; the other families showed **<0.5%**. The authors state the confound does *not* affect any downstream accuracy result (all single-window, all with BOS), that external groups have replicated the accuracy evals, and that the *qualitative* layer-sweep pattern survives. The perplexity tables below are therefore reproduced as the record of what the paper prints, not as effect sizes to cite.

## Summary

The motivating deficit is **state tracking**. A feedforward transformer can only update a latent state by moving it one layer deeper, so the number of sequential state updates is bounded by depth, and — per Lepori et al. 2025's Patchscopes analysis — a disambiguated representation (river-*bank* vs. financial *bank*) lives in deep layers where the shallow layers processing the *next* question cannot read it. Lepori et al. showed that copying a deep-layer activation down to a shallow layer *for one pre-identified critical token* cut contextualization errors by 60%. Recirculation asks what happens if you do that for **every** token, but as a small *leak* rather than a replacement: after each token's forward pass, mix a norm-matched fraction of its layer-`s` residual into its layer-`d` residual (`d < s`) and re-run the stack from `d` upward, so that subsequent tokens attend to representations that already contain the deep-layer state. No weights change. The bet that this works without training rests on the [[residual-stream|residual stream]] being a shared, additively-written blackboard in which features keep a consistent orientation across depth (Elhage et al. 2021), so no adapter is needed.

The paper's own framing distinguishes this sharply from [[looped-transformer|looping]]. A loop is recurrence *in depth only*: the recirculated state must climb one layer per update, so a looped transformer is "just a deeper feedforward transformer with weight-sharing constraints." Recirculation is recurrence *in depth and in input step*: the same layer `d` can hold both `z(t)` and `z(t+1)`, so with unbounded iterations it behaves as a true RNN. The price is that the sequential dependency cannot be parallelised — **prefill must be processed token by token** — while decode, which is already sequential, pays only for running two stacks side by side.

Results are a mixed bag, honestly reported. Layer sweeps on Gemma3 1B/4B/12B, Ministral3, Qwen3, Pythia, Phi2, Gemma2 and Gemma4 all show a smooth, robust region (destination in the lower-middle of the stack, source 5–7+ layers above) where perplexity falls; training-free *looping* on the same grid shows no robust region for Gemma3. Downstream, the headline is Gemma3 4B on GSM8K: **adaptive recirculation** — a small MLP that emits per-token, per-dimension mixing vectors while the LLM stays frozen — cuts error by **8.8% (pass@1)** and **20.9% (pass@128)**. Single-token benchmarks move by fractions of a point in either direction; the Lepori contextualization suite improves for 1B and 4B and *regresses* for 12B.

## Mechanism

One-iteration recirculation, with `z_{i,j,l}` the residual output after layer `l` at unrolling pass `i` for input step `j`:

```
z_{t+1,t,d} = α · f(z_{t,t,s} | d,t) + β · z_{t,t,d}
f(z | d,t)  = (‖z_{t,t,d}‖₂ / ‖z‖₂) · z          # rescale source to destination norm
```

- **Source `s` → destination `d`**, `d < s`, chosen by sweep; the recirculated activation is a *mix* of source and destination, not a substitution (contrast: looping feeds the entire residual as the direct replacement input to the next block).
- **Convex mix** `β = 1 − α` for Gemma3 1B (α ≈ 0.07–0.15); Gemma3 4B and 12B "critically require" the non-convex `β = 1`.
- **Norm matching** matters for *robustness* of the sweep landscape, not for the best-case reduction, because residual norms grow with depth.
- Every token is processed by two stacks; the second pass of token `t` runs in lockstep with the first pass of token `t+1`; **readout happens after a stack's first pass**. The first token is a warm-up with no recirculation.
- **Ramping** (attenuating α for the first ~20 positions) helps the 1B model, where recirculating early tokens hurts; 4B and 12B show no early-token harm.
- All experiments use exactly one extra iteration; a two-iteration unrolling (three passes per stack) is drawn in App. A but untested.

**Cost.** Generation: "essentially no additional latency" because two stacks run in parallel on modern hardware — the paper does not report wall-clock or FLOP numbers, and the arithmetic cost per token roughly doubles from layer `d` up. Prefill: serial, token-by-token, "which for large-context problems can be quite slow"; the proposed but unexplored mitigation is *blockwise* recirculation (recirculate `K` tokens at a time). This is the opposite cost profile from [[chunked-prefill]] and from every prefill-acceleration technique in the wiki.

**What it buys, in the authors' terms.** Shallow-layer access to deep-layer state at *previous* positions; the ability to "act as a dynamical system and track belief states"; and a diagnostic they call *model-design affordances* — using a trained network's own response to training-free tweaks to discover which architectural changes it "wants," as a cheaper alternative to proposing a modification and paying to train it.

## Key results

**Layer sweeps (Gemma3, arXiv/C4/PG19 tuning windows, ~1.5M tokens).** Baseline Gemma3 1B perplexity on the arXiv tuning subset is 16.6. At α = 0.10 the best (source, destination) pair averages a **4.72%** reduction across the three datasets. Larger α amplifies the effect but grows the harmful region. Selected pairs (Table B.1):

| Model | Source | Destination | Layers |
|---|---|---|---|
| Gemma3 1B PT | 11 | 4 | 26 |
| Gemma3 4B PT | 18 | 9 | 34 |
| Gemma3 12B PT | 35 | 16 | 48 |

**Perplexity, ten datasets, α = 0.15 (Table 1) — see the warning above; these magnitudes are confounded for Gemma.**

| Dataset | Tokens | 1B base → recirc (Δ) | 4B base → recirc (Δ) | 12B base → recirc (Δ) |
|---|---|---|---|---|
| arxiv | 51M | 19.10 → 16.54 (−13.99%) | 14.76 → 13.10 (−11.26%) | 33.93 → 25.38 (−25.20%) |
| big_patent | 55M | 10.89 → 9.90 (−9.13%) | 8.74 → 8.12 (−7.08%) | 17.55 → 13.20 (−24.76%) |
| billsum | 5.8M | 4.71 → 4.65 (−1.25%) | 3.57 → 3.41 (−4.46%) | 4.50 → 4.09 (−10.96%) |
| booksum/book | 7.7M | 32.48 → 27.30 (−15.95%) | 29.09 → 24.45 (−15.95%) | 77.02 → 51.67 (−32.91%) |
| c4/webtextlike | 4M | 17.10 → 16.43 (−3.93%) | 14.26 → 13.64 (−4.37%) | 19.21 → 16.71 (−13.01%) |
| gov_report | 3.4M | 11.86 → 11.00 (−7.26%) | 10.79 → 9.78 (−9.37%) | 27.59 → 19.11 (−30.74%) |
| lambada | 418k | 35.62 → 35.88 (+0.72%) | 28.34 → 28.19 (−0.53%) | 25.47 → 26.19 (+2.81%) |
| newsroom | 95M | 14.11 → 13.75 (−2.56%) | 11.62 → 11.17 (−3.87%) | 13.74 → 12.45 (−9.40%) |
| pg19 | 10.8M | 22.27 → 19.06 (−14.41%) | 19.49 → 16.43 (−15.72%) | 52.86 → 34.15 (−35.40%) |
| pubmed | 23M | 15.73 → 13.99 (−11.08%) | 11.65 → 10.58 (−9.21%) | 26.38 → 19.92 (−24.49%) |

Note the anomalously high 12B baselines (33.9 on arXiv vs. 14.8 for 4B) — that is the missing-BOS artifact the authors later diagnosed. Lambada is the one dataset that gets worse; the authors attribute it to very short sequences.

**Controls.** Temperature: on PG-19, Gemma3 1B at T = 1.2 gains 8.48%; recirculation alone 14.21%; both together 19.55% — near-additive, so recirculation is not a global softmax sharpening. Lag analysis (recirculate one token at position `t`, measure log-likelihood gain at `t+k`): benefit is a power law in `k` with a residual tail at `k = 256`; positions ~20–200 have the most persistent effect; adverbs, adjectives and verbs benefit most, numerals/determiners/pronouns least; plural nouns benefit, singular nouns do not. Recirculating all-but-one token gives complementary results, suggesting per-token benefits are additive in log-likelihood.

**Recirculation vs. looping (Fig. 8).** Training-free looping was implemented by inserting a copy of layers `ℓ₁+1…ℓ₂` after layer `ℓ₂` and sweeping all pairs. For Gemma3, looping "does not produce robust benefits," and where it helps at all it is only at the larger scales (consistent with Ng 2026); recirculation helps at every scale. The heatmaps are qualitatively different, which the authors take as evidence the two operate on different principles.

**Downstream (Gemma3, downstream evals unaffected by the BOS issue).**

*Instruction following* (two-word fruit game, 2-shot; 1B IT is at chance so omitted): with perplexity-selected hyperparameters, error rate falls ~**25% (4B)** and ~**75% (12B)**; a task-specific 2-parameter layer sweep gives more.

*Contextualization* (Lepori et al. "Racing Thoughts" yes/no suite, chance 25%, 0–n distractors): 1B improves on 2 of 3 question types (third at chance for both); 4B improves on 2 of 3; **12B is worse on 2 of 3** (third at ceiling). Hyperparameters were carried over from PT perplexity sweeps; re-sweeping on the IT model gave larger gains.

*Single-token benchmarks, Gemma3 4B PT, zero-shot, hyperparameters chosen on the MMLU dev set (Table 2):*

| Dataset | Baseline | Recirculation | Adaptive (tuned on MMLU test) |
|---|---|---|---|
| MMLU | 57.90 | 58.28 | 59.83\* |
| ARC Easy | 81.78 | 82.07 | 82.79 |
| ARC Challenge | 54.44 | 54.86 | 54.95 |
| PiQA | 79.98 | 80.52 | 80.20 |
| BoolQ | 79.08 | 79.11 | 81.04 |
| WinoGrande | 69.46 | 69.38 | 69.61 |
| HellaSwag | 75.89 | 75.86 | 76.01 |
| Lambada | 70.02 | 70.27 | 71.07 |

Basic recirculation improves 6 of 8, all by <0.6 pt. \*MMLU-test-tuned adaptive recirculation was trained on 3,600 of the ~14,000 test examples and evaluated on the full test set, so the MMLU row overlaps train and test; adaptive variants tuned on MMLU-train, ARC-Easy or ARC-Challenge instead *hurt* most datasets (e.g. ARC-Easy-tuned: MMLU 56.05, BoolQ 76.08, Lambada 62.95).

*GSM8K, Gemma3 4B PT, zero-shot CoT (Kojima prompt):* basic recirculation improves both pass@1 (greedy) and pass@128 (temperature + nucleus sampling), read via Yue et al. 2025 as both *sharpening* and *expansion*. Adaptive recirculation yields **8.8% (pass@1) and 20.9% (pass@128) reductions in error rate**. Absolute accuracies appear only in Fig. 12 bars and are not stated in the text.

**Adaptive recirculation (§4.6).** Fix `(s, d)`, freeze Gemma, and learn the mixing coefficients on 250 documents each from arXiv/C4/PG19 (100 steps, batch 32, AdamW lr 3e-4, wd 1e-4, BPTT). Six variants on Gemma3 1B, nine eval datasets, mean perplexity reduction: fixed α = 0.15 / β = 0.85 → **8.5%**; learned per-token, per-dimension vectors from a 2-hidden-layer GELU MLP (input = concatenated source and destination embeddings, sigmoid output, initialised to α = 0.1 / β = 0.9) → **23.0%**; full Gemma3 1B fine-tuning with the recurrent connection → **21.6%**. Vector beats scalar; token-conditional beats constant. (These are perplexity numbers on Gemma and thus subject to the BOS caveat; the ordering is what the authors stand behind.)

## Limitations (stated, plus one observed)

1. **Hyperparameters are task/domain dependent** — `(s, d, α, β)` transfer reasonably across tasks and small tuning sets suffice, but without task-universal settings or an automatic task→hyperparameter map, "the practical applicability of recirculation will be limited."
2. **Model-family dependence.** Gemma may be unusually compatible because of its Peri-LN (input *and* output normalisation per sublayer) and its training recipe; Gemma4 E2B/E4B (cross-layer KV sharing, per-layer embeddings) show unstable sweeps.
3. **Normalisation** scheme is Gemma-tuned; necessity and form for other families are open.
4. **Sequential prefill** — the real cost; blockwise recirculation is proposed but not evaluated. No latency or throughput measurements are given anywhere in the paper.
5. **Single path, single iteration.** Multi-path (the 4B/12B sweeps show disjoint good regions) and `r > 1` iterations are untested.
6. Observed: the 12B contextualization regression and the sub-point single-token deltas mean the only robust downstream win is on extended-generation reasoning (GSM8K), and that one requires the adaptive MLP.
7. Observed: the paper's own abstract still reads "a 21% increase in accuracy on GSM8k"; the body reports a **20.9% reduction in error rate at pass@128**, which is a different and smaller claim.

## Relation to depth-recurrence in this wiki

Recirculation *does* feed late-layer residuals back to early layers, so it is kin to [[looped-transformer]] / [[universal-transformer]] / [[recurrent-depth-transformer]] in mechanism — but the authors' central argument is that the *unrolling* differs: looping recurs within a token's stack (depth), recirculation recurs across tokens (depth × step), which is what lets a single layer carry an evolving state `z(t) → z(t+1)`. Two further contrasts with the wiki's looping literature: (a) [[latent-thoughts-looped-transformers-saunshi-2025|Saunshi 2025]], [[loop-think-generalize-2026|Kohli 2026]] and [[parcae-scaling-laws-looped-2026|Parcae]] all *train* the loop, whereas recirculation is applied to off-the-shelf weights and the paper finds *training-free* looping does not help Gemma3; (b) [[adaptive-computation-time]]-style halting is replaced by a fixed one-extra-pass budget, with adaptivity instead placed in the per-token mixing coefficients. The closest prior work by the authors' own account is T²MLR (Cai et al. 2026), which also routes a deeper layer of token `t−1` into a shallower layer of token `t` but needs BPTT training and a gated fusion module. Recirculation also reads as inference-time self-[[steering]]: the steering vector is the model's own deep-layer activation rather than a fixed external direction ([[activation-addition]], [[steering-vectors]]).

## Concepts & entities

- [[residual-stream]] — the alignment-across-depth property the whole method relies on; the paper's moisture-feature argument is a claim about it.
- [[looped-transformer]] · [[universal-transformer]] · [[recurrent-depth-transformer]] — the family recirculation is defined against; see the section above.
- [[adaptive-computation-time]] — the halting mechanism recirculation does *not* use.
- [[inference-time-compute]] · [[test-time-compute]] — recirculation is depth-recursion spent at inference, but unlike token-recursion it is fixed-budget and invisible in the output.
- [[chain-of-thought]] · [[latent-reasoning]] — the authors reserve CoT for complex inference and position recirculation for "basic state tracking"; both are forms of deep→shallow signal propagation.
- [[recurrent-neural-network]] · [[state-space-model]] — with unbounded iterations recirculation is an RNN; the paper argues most SSMs are no more expressive than a plain transformer for state tracking.
- [[attention-sink]] — the mechanism behind the BOS confound.
- [[chunked-prefill]] · [[kv-cache]] — sequential prefill is the cost; the second-pass KV is what later tokens read.
- [[layer-normalization]] — Gemma's Peri-LN is the leading hypothesis for its outsized compatibility.
- [[steering]] · [[activation-addition]] · [[steering-vectors]] — recirculation as self-steering.
- [[mmlu]] — hyperparameter-selection set for the single-token benchmarks.
- [[world-model]] — cited motivation: state-tracking failures manifest as the "absence of a robust internal world model."
- [[deepmind]] · [[qwen]] — affiliation; Qwen3 is one of the five families swept.
- [[universal-transformer-dehghani-2018]] · [[looped-transformers-programmable-computers-giannou-2023]] · [[cot-or-loop-xu-2025]] · [[adaptive-loops-and-memory-in-transformers-frey-2026]] — the wiki's looping sources this paper positions itself against.

**Concept gaps (not created):** `recirculation`, `state-tracking`, `gemma` (model family entity), `patchscopes`, `contextualization-error`, `peri-ln`, `model-design-affordances`, `pass-at-k` (sharpening vs. expansion, Yue et al. 2025).

## References

- [arXiv:2608.17981](https://arxiv.org/abs/2608.17981)
- [[universal-transformer-dehghani-2018]] · [[looped-transformers-programmable-computers-giannou-2023]] · [[latent-thoughts-looped-transformers-saunshi-2025]] · [[loop-think-generalize-2026]] · [[parcae-scaling-laws-looped-2026]]
