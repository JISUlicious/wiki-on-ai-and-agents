---
title: Cloze Scoring
type: concept
created: 2026-09-22
updated: 2026-09-22
sources: []
status: complete
importance: high
---

# Cloze Scoring

Answering a question by reading a model's probability distribution over a small set of candidate tokens at one position, rather than generating an answer. The lineage runs from BERT-era cloze prompting through the standard multiple-choice evaluation protocol to the 2026 [[system-one-model|System One]] products — and it is old enough that the 2026 arrivals are better described as productization than invention.

This page carries the prior art. [[logit-scoring]] covers the modern productized form.

## The lineage

**2019 — reading a masked slot as knowledge retrieval.** LAMA ([arXiv:1909.01066](https://arxiv.org/abs/1909.01066)) queried BERT with `"Paris is the capital of [MASK]"` and read the MLM head's distribution. Masked Language Model Scoring ([arXiv:1910.14659](https://arxiv.org/abs/1910.14659)) established pseudo-log-likelihood as a usable scoring function without fine-tuning.

**2020 — the pattern-verbalizer pair.** PET ([arXiv:2001.07676](https://arxiv.org/abs/2001.07676)) formalized the recipe: a *pattern* rewrites the input as a cloze with exactly one `[MASK]`, a *verbalizer* maps each class label to a vocabulary token, and the class score is the softmax over just the verbalizer tokens.

> [!important] The single-token constraint is PET's limitation, not a 2026 design insight
> PET required each label to map to a single token. Its own follow-up ([arXiv:2009.07118](https://arxiv.org/abs/2009.07118)) exists largely to *lift* that restriction with multi-mask patterns. The field spent 2020–2021 trying to escape the constraint that [[jev|Jev]], [[nimble]] and [[openjev]] present as an architectural feature.

**2020 — the MMLU reference implementation.** This is the closest ancestor, and it is exact. Hendrycks et al. ([arXiv:2009.03300](https://arxiv.org/abs/2009.03300)) scored [[mmlu|MMLU]] with a single forward pass at `max_tokens=1`, pulled the logprobs for the four letter tokens, took a softmax over *only those four*, and used the result as both the prediction and its probability. Every step of the System One readout — enumerate, remap to one token, one pass, filter, renormalize — is in that loop.

A competing convention scored full answer *strings* as continuations with length normalization (GPT-3, [arXiv:2005.14165](https://arxiv.org/abs/2005.14165)), inherited by lm-evaluation-harness. OLMES ([arXiv:2406.08446](https://arxiv.org/abs/2406.08446)) later named the two regimes **cloze formulation** and **multiple-choice formulation** and standardized reporting, because the choice moves scores materially.

**2022 — the argument that symbols should be the default.** Robinson & Wingate ([arXiv:2210.12353](https://arxiv.org/abs/2210.12353)) named the required capability **multiple-choice symbol binding**, showed it varies enormously by model, and argued for symbol scoring on three grounds: the model can compare options explicitly, compute cost drops, and tokenization effects are mitigated. Those are three of the four benefits the 2026 products advertise.

**2025 — training a model for the readout.** ModernBERT-Large-Instruct ([arXiv:2502.03793](https://arxiv.org/abs/2502.03793)) extracted all single-token answers from FLAN, masked only that answer token, and read the MLM head. At 0.4B parameters it beat similar-size LLMs on MMLU. That is Jev's training premise and Nimble's readout, on an encoder, nineteen months early — and it is why the strongest 2026 open competitors are encoders. See [[laya]].

**2026 — the diffusion version, published first.** dLLM-SetScore ([arXiv:2608.14649](https://arxiv.org/abs/2608.14649), August 2026) did training-free multi-label classification on LLaDA-8B and Dream-7B by comparing answer-token probabilities at a masked position — the [[openjev|OpenJev]] mechanism, on other diffusion models, a month before Jev's announcement. Reading masked-slot likelihood has been the field-standard evaluation protocol for diffusion LMs since LLaDA ([arXiv:2502.09992](https://arxiv.org/abs/2502.09992)) in early 2025.

## The one genuinely new point

Single-token codes **structurally dodge surface form competition**. Holtzman et al. ([arXiv:2104.08315](https://arxiv.org/abs/2104.08315), EMNLP 2021) showed that a correct answer can lose because its probability mass is fragmented across paraphrases not on the ballot — "computer" competing with "PC". A symbol like `A` has no paraphrases, so the failure mode cannot arise.

Two caveats on stating this. Holtzman et al. never discuss symbols; the paper is defined over natural-language answer strings, so this is a sound inference from their result rather than a claim they make — cite Robinson & Wingate for the empirical support. And the correction term does not vanish, it changes identity: Holtzman's fix divides out each option's a-priori likelihood, and the symbol scheme simply swaps *"how likely is this phrasing"* for *"how much does this model like the token `A`"*.

## Known pathologies

These are load-bearing, because they are exactly the contaminants the modern claim assumes away:

| Pathology | Source | Magnitude |
|---|---|---|
| **Option-order sensitivity** | [arXiv:2308.11483](https://arxiv.org/abs/2308.11483) | reordering moves performance ~13–75% by benchmark |
| **Symbol token bias** | [arXiv:2309.03882](https://arxiv.org/abs/2309.03882) | models prefer specific option IDs a priori; fix is PriDe — estimate the prior by permuting contents, divide it out |
| **Tokenization of the code itself** | [arXiv:2509.15020](https://arxiv.org/abs/2509.15020) | reading `"A"` vs `" A"` moves accuracy up to 11% and reshuffles model rankings |
| **First-token/text mismatch** | [arXiv:2402.14499](https://arxiv.org/abs/2402.14499), [arXiv:2404.08382](https://arxiv.org/abs/2404.08382) | >60% mismatch for instruction-tuned models; the text answer is the more robust of the two |
| **Slot-position asymmetry** | [arXiv:2608.14649](https://arxiv.org/abs/2608.14649) | multi-slot single-pass reads predict the *first* slot positive on 99.4%/100% of examples |

Token bias deserves emphasis because the System One design *aggravates* it: remapping semantic labels onto exactly the symbols that carry contaminated priors makes PriDe-style debiasing load-bearing rather than optional hygiene.

**The mechanistic version of the worry.** Wong et al. ([arXiv:2601.03914](https://arxiv.org/abs/2601.03914), January 2026) find MCQA is two stages: the model picks a winner in *content* space, decodable right after the last option, then separately **binds** that winner to a symbol near emission. Symbol logits reflect the binding step. A confidence read off them is partly measuring binding reliability rather than answer reliability — a precise reason to doubt "the distribution *is* the confidence."

## Is the readout well calibrated?

Genuinely mixed, and the wiki should not flatten it.

**For:** large models are well calibrated on multiple-choice and true/false given the right format ([arXiv:2207.05221](https://arxiv.org/abs/2207.05221)); the **pre-trained** GPT-4 is highly calibrated on MMLU and **RLHF post-training degrades it significantly** ([arXiv:2303.08774](https://arxiv.org/abs/2303.08774)) — the single best argument for reading the distribution rather than asking the model, and for wanting a calibration-preserving objective. Pre-trained transformer classifier heads are calibrated in-domain out of the box ([arXiv:2003.07892](https://arxiv.org/abs/2003.07892)).

**Against:** for RLHF'd models on open-ended QA, *verbalized* confidences are often **better** calibrated than the model's conditional probabilities ([arXiv:2305.14975](https://arxiv.org/abs/2305.14975)). The honest scope is: logit-derived confidence wins for base models on bounded-candidate tasks; verbalized can win for RLHF'd models on open-ended generation.

**Calibration as a training objective is also prior art.** Teaching uncertainty with ~1,000 graded examples via LoRA generalizes well ([arXiv:2406.08391](https://arxiv.org/abs/2406.08391), NeurIPS 2024) — [[nimble]]'s recipe, two years early. And a systematic study four months before Jev ([arXiv:2605.11845](https://arxiv.org/abs/2605.11845), May 2026) found calibration fine-tuning works across 12 models but that gains **sometimes reduce downstream capability**, especially on arithmetic reasoning — a cost nobody has checked for Jev.

## So how novel is the 2026 wave?

The mechanism is 2020-vintage and the latency claim is arithmetic rather than insight: one forward pass with zero decode steps is fast because there is nothing to decode. Of the four properties [[system-one-model|System One models]] advertise, three are decisively old — bounded output, no-decode latency, distribution-as-confidence. The fourth, calibration, is the only real differentiator. See [[independent-jev-calibration-audits-2026]] for what measuring it found.

## Related

- [[logit-scoring]] · [[system-one-model]] · [[calibration]] · [[masked-language-modeling]] · [[mmlu]] · [[bert]] · [[constrained-decoding]] · [[jev]] · [[nimble]] · [[laya]] · [[diffusion-language-model]]

## References

- PET [2001.07676](https://arxiv.org/abs/2001.07676) · MMLU [2009.03300](https://arxiv.org/abs/2009.03300) · MCSB [2210.12353](https://arxiv.org/abs/2210.12353) · Surface form competition [2104.08315](https://arxiv.org/abs/2104.08315)
- ModernBERT-Large-Instruct [2502.03793](https://arxiv.org/abs/2502.03793) · dLLM-SetScore [2608.14649](https://arxiv.org/abs/2608.14649) · Prophet [2508.19982](https://arxiv.org/abs/2508.19982)
- Calibration: [2207.05221](https://arxiv.org/abs/2207.05221) · [2303.08774](https://arxiv.org/abs/2303.08774) · [2305.14975](https://arxiv.org/abs/2305.14975) · [2406.08391](https://arxiv.org/abs/2406.08391) · [2605.11845](https://arxiv.org/abs/2605.11845)
