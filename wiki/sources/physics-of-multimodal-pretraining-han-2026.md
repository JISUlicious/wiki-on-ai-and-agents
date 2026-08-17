---
title: "Towards Physics of Multimodal Pretraining: Knowledge Flow, Modality Synergy, Early Unification, and Recipes"
type: source
created: 2026-08-17
updated: 2026-08-17
sources: []
arxiv_id: "2608.05000"
authors:
  - Junlin Han
  - Shengbang Tong
  - David Fan
  - Minghao Chen
  - Philip Torr
  - Filippos Kokkinos
  - Mike Lewis
year: 2026
tags:
  - 2026
  - paper
status: complete
importance: high
---

# Towards Physics of Multimodal Pretraining: Knowledge Flow, Modality Synergy, Early Unification, and Recipes

**Junlin Han, Shengbang Tong, David Fan, Minghao Chen, Philip Torr, Filippos Kokkinos, Mike Lewis** (FAIR at [[meta|Meta]]; Reality Labs, Meta; University of Oxford) — [arXiv:2608.05000](https://arxiv.org/abs/2608.05000) v2, cs.CV, 5–6 Aug 2026. [Project page](https://junlinhan.github.io/projects/physics_of_mm_pretrain/). No venue listed.

## The problem

Natively unified multimodal models train three abilities in one loop: language, image *understanding* (image→text), and image *generation* (text→image). How knowledge moves among the three — and whether it moves at all — is unsettled, with prior studies reaching opposite conclusions. This is a "physics of" style paper: controlled single-variable ablations at small scale, then one confirmation run at 13.5B.

## The method

**Backbone.** Default probe model is a 1.5B [[llama|Llama-3]]-style decoder-only transformer (2.3B total including modality-specific parts): 16 layers, d=2048, [[grouped-query-attention|GQA]] with 32 query / 8 KV heads, FFN multiplier 1.5, context 4096, bf16, FSDP-2, AdamW. It follows the **Transfusion** recipe — discrete next-token prediction for text, continuous **flow matching** for image generation — with the diffusion loss upweighted 3.0× against text cross-entropy.

**Visual tokenization, four variants:** RAE (default; frozen SigLIP-2 [[vit|ViT]]-400m/14, 224², 16×16 = 256 semantic tokens), raw pixels (14×14 conv patchify), CLIP+VAE (SigLIP-2 for understanding + SD3 VAE for generation), and AR/UniTok (discrete codes).

**Data.** Language = DCLM. Vision-text = ~350M Shutterstock (SSTK) pairs, exclusively. Plus an extended synthetic CLEVR with five conceptual axes (color, shape, spatial relation, size, count), ~1M rendered scenes. Post-pretraining SFT on Cambrian-7M, one epoch. Controlled runs at 100B–2T tokens; scaled runs at 2T.

**Evaluation.** Visual understanding across 16 benchmarks in four axes (General, Knowledge, OCR & Chart, Vision-Centric); language over 11 benchmarks plus DCLM/C4 perplexity; generation via GenEval, DPG-Bench, CLIP-Sim, held-out diffusion loss, and FID.

## Results

### 1. Knowledge flow is asymmetric

Sweeping one modality's tokens 0→200B on top of a fixed 50B base of another:

| Direction | Effect |
|---|---|
| **Language → vision** | Strong positive. Raising the language ratio 0%→80% steadily improves Vision-Centric tasks, gives "dramatic relative increases" in Knowledge and OCR & Chart, and lowers both conditional and unconditional diffusion loss. |
| **Understanding → generation** | Strong positive. "Markedly improves generation evaluation metrics"; significantly lowers diffusion loss. |
| **Generation → understanding / language** | **Negligible.** "Minor fluctuations in language benchmark accuracy and perplexity"; General / Knowledge / OCR / Vision-Centric show "marginal fluctuations." |

The synthetic CLEVR study shows the transfer is **concept-dependent**, not uniform. Low-level concepts (color, shape) show *no* zero-shot transfer in either direction — held-out generation accuracy sits at control level — but generation still leaves a **latent prior** that surfaces during fine-tuning (recovery curves gain **Δ mean accuracy +0.133 to +0.273** over control). Structural concepts (spatial relation, size, count) do transfer understanding→generation; the reverse fails except for counting.

### 2. Synergy vs. competition is governed by data complexity

Pairing a modality with a *simple* target in the other modality **improves** it beyond the unimodal baseline; pairing it with a complex one **degrades** it:

| Pairing | Effect |
|---|---|
| Pure-background images + language | Δ language PPL **−0.211** (better than a pure LM) |
| Noise images + language | Δ PPL **−0.183** |
| Simple language + SSTK images | Δ diffusion loss **−0.0153** cond. / **−0.0168** uncond. |
| Real SSTK images + language | Δ PPL **+0.052 to +0.075** (worse) |
| Video + language | Δ PPL **+0.052** (worse) |

**Which parameters to share** — 100B tokens (50B language / 50B visual), synergy-maximizing probes, deltas vs. unimodal baselines:

| Config | Δ language PPL | Δ vision loss |
|---|---|---|
| `dense` (share FFN + attention + norm) | **+0.272** (worse) | **+0.0537** (worse) |
| **`split_ffn`** (decouple FFNs only) | **−0.211** (better) | **−0.0168** (better) |
| `split_ffn_attn` (also decouple attention) | −0.045 improvement only | −0.0064 only |
| `split_ffn_norm` (also decouple final norm) | improvement visibly diminished | diminished |
| `split_all` | ≈ baseline (no synergy at all) | ≈ baseline |

The rule the paper extracts: **shared attention and normalization are the bridge that produces synergy; the FFN is where competition lives, so it must be modality-specific.** Attention lets modalities contextualize each other in a joint space and shared norm aligns feature scales; FFNs "act as modality-specific experts that absorb the distinct computational demands of each stream." This is a direct architectural argument for [[mixture-of-experts|MoE]] / Mixture-of-Transformers designs.

Synergy survives every tokenizer choice — raw pixels is actually the *strongest* (Δ PPL −0.266, vs. RAE's −0.211), and even the fully decoupled CLIP+VAE setup keeps it (Δ PPL only −0.024 but relative diffusion loss −2.24% / −2.83%). So a pre-aligned or shared visual representation space is **not a prerequisite** for cross-modal transfer.

### 3. Early unification, and "vision laziness"

Holding a 1T-token budget fixed and sweeping the pure-language warm-up over {0, 200, 400, 600, 800}B before switching to a 50/50 unified mix: language accuracy and PPL improve early, then **plateau between 600B and 800B**, while every visual understanding and generation metric shows "a steep and consistent decline" the longer vision is delayed. Sequential staging is worse still — all six orderings of L/U/G at a fixed 1T budget and a 50/25/25 mix "fall drastically short of the joint baseline," and a 12.5% replay buffer only slightly mitigates it.

The mechanism, measured four ways across warm-up lengths, is **vision laziness**: training-time FFN activation L2 falls across layers; image-wrapper-token embedding norms shrink away from language-vocab scale; inference-time vision-FFN activation RMS "drops drastically"; and the fraction of attention mass on image tokens falls sharply, "particularly in the mid and deep layers." The summary: "The stronger the underlying LLM, the more adept it becomes at leveraging its advanced language capabilities to shortcut visual problems."

### 4. The recipe — L70/U25/G5

A grid search over mixing ratios at 1T tokens, in three sweeps: *Fix MM* (language 10%→90%, remainder split evenly), *Fix Lan* (language pinned at 50%, sweep U:G), and *Next* (language pinned at **70%**, having been identified as the language-preservation optimum, sweep U:G).

**The optimum is L70 / U25 / G5.** Cutting generation to 5% "dramatically boosts visual understanding" — General VQA hits **48.3** — "while simultaneously achieving the highest generation evaluation scores."

**Scaled validation: 13.5B MoE (256 experts, top-16 routed, 1.5B active/token, two of the active experts pinned modality-specific — one language, one vision), 2T tokens.** Each row is a single-variable change from `Balanced Recipe`:

| Model | Lang PPL ↓ | Lang Acc ↑ | VU Avg ↑ | DPG ↑ | GenEval ↑ | CLIP-Sim ↑ | DiffLoss ↓ |
|---|---|---|---|---|---|---|---|
| **Full** (L70/U25/G5, MoE, early) | **11.67** | **54.31** | **43.08** | **0.689** | **0.482** | **0.312** | 0.272 |
| Balanced Recipe (L50/U25/G25, MoE, early) | 11.97 | 52.86 | 41.42 | 0.676 | 0.467 | 0.310 | **0.261** |
| Dense Model (3.5B dense, 1.5B active) | 12.14 | 52.03 | 40.49 | 0.667 | 0.459 | 0.308 | 0.266 |
| Late-Fusion (vision after 60% of training) | 12.25 | 51.78 | 40.66 | 0.672 | 0.471 | 0.308 | 0.269 |

Every design choice moves in the predicted direction, and the margins are small but consistent: asymmetric mix +1.45 pp language accuracy and +1.66 pp VU average over balanced; MoE +0.83 / +0.93 over dense; early unification +1.08 / +0.76 over late fusion. The asymmetric mix improves text-to-image alignment (**GenEval 0.482 vs. 0.467, DPG 0.689 vs. 0.676**) *despite five times fewer generative tokens*. It costs a small amount of raw generative fidelity — **diffusion loss 0.272 vs. 0.261**, though **FID on 50k images is 5.234 vs. 5.131**, i.e. essentially matched.

> [!note] What the "5% of the compute budget" claim actually is
> The abstract's "strong generative performance using only 5% of the compute budget" refers to the **generation share of the pretraining token mix** (the G5 in L70/U25/G5) — "five times fewer generative tokens" than the L50/U25/G25 baseline. It is a data-allocation result, not an independent 20× compute reduction, and the comparison baseline is the authors' own balanced mix, not an external system.

## Where it breaks

- **Static images and text only.** Video and audio are explicitly out of scope; the paper's own video probe was one of the two settings that *degraded* language PPL (+0.052).
- **Scale.** Validated at 13.5B / 2T. The authors flag that "the threshold at which modalities transition from synergy to competition might shift" past ~1T params, and speculate that at extreme scale generation might start transferring backward (as an internal world simulator) — currently it does not.
- **Single vision-text corpus.** All real-world vision-text data is ~350M Shutterstock pairs; no cross-corpus check.
- **Synthetic work is CLEVR-only** — clean but narrow, and the five conceptual axes are hand-chosen.
- **Vision laziness is diagnosed, not intervened on.** Four correlational measurements, no causal manipulation.
- **The optimal mix is one number from one setup.** L70/U25/G5 is fitted at 1T tokens on this data, backbone, and tokenizer; the paper itself notes that "increasing the generation ratio in [a] later stage of pretraining or introducing a dedicated visual generation midtraining stage remains beneficial."
- No human evaluation of generations; GenEval/DPG/CLIP-Sim/FID are all proxies.

## Related

- [[multimodal-llm]] — the central subject; this is the pretraining-side counterpart to most multimodal work, which studies post-hoc alignment
- [[pre-training]] · [[scaling-laws]] — the recipe is an allocation result over the pretraining token budget
- [[mixture-of-experts]] — the paper derives MoE from first principles: shared attention/norm + modality-specific FFNs *is* an MoE with pinned modality experts
- [[patch-tokenization]] · [[vit]] — the four tokenizer variants; raw-pixel patchify shows the strongest synergy
- [[attention-mechanism]] · [[layer-normalization]] — shared attention and shared normalization are identified as the synergy-carrying components
- [[transformer-architecture]] · [[llama]] · [[grouped-query-attention]] — backbone details
- [[fine-tuning]] · [[instruction-tuning]] — Cambrian-7M SFT reveals latent priors that pretraining metrics miss
- [[world-model]] — the speculative frontier claim: at extreme scale, generation might become an internal simulator that transfers backward
- [[meta]] · [[fair]]

## References

- [arXiv:2608.05000](https://arxiv.org/abs/2608.05000)
