---
title: "Constrained Canvas vs. Free Denoise"
type: comparison
created: 2026-09-22
updated: 2026-09-28
sources:
  - structured-generation-mode-for-diffusiongemma-mastracci-2026.md
  - open-jev-diffusiongemma-harness-github-2026.md
status: complete
importance: medium
tags:
  - 2026
---

# Constrained Canvas vs. Free Denoise

Two independent September 2026 efforts turned [[diffusiongemma|DiffusionGemma]] into a typed-decision model in the style of [[jev|Jev]]. They are usually described as opposites. They are not, and both popular labels are partly wrong.

- **A — "constrained canvas"**: [[structured-generation-mode-for-diffusiongemma-mastracci-2026|vLLM PR #57250]] and the [[openjev|OpenJev]] server.
- **B — "free denoise"**: the [[open-jev-diffusiongemma-harness-github-2026|open-jev]] harness, whose README says "denoise freely, then choose the most likely allowed tokens from the final logits."

> [!warning] B is not free, and the real difference is narrower than the labels suggest
> Route B also pre-seeds the canvas with a literal JSON template and masks every non-answer position to exactly one token **at every denoising step**. Structure is hard-constrained in both. The divergence is confined to the answer slots — and at one denoising step, with no self-conditioning feedback yet, even that nearly collapses. What actually separates them is the *alphabet* in the answer slot and the *readout rule*.

## The real axis

| | A — label codes | B — padded values |
|---|---|---|
| Structure positions | masked to one token each step | masked to one token each step |
| Answer slots | masked to the candidate label ids | **open to the full vocabulary**, seeded with uniform noise |
| What occupies a slot | a single token: `yes`/`no`, `A`/`B`/`C`, `0`–`9` | the real JSON fragment `"key": value`, space-padded so all options are equal width |
| Canvas cost per question | **1 position** | `max(token width)` over the field's options, plus key and punctuation |
| Options per field | ≤ 26 (needs distinct single tokens) | unbounded by alphabet; limited only by canvas capacity |
| Readout | softmax over candidate ids at one position | greedy trie walk across several positions |
| Returns a probability? | **yes** | **no** (see below) |
| Output shape | codes mapped back by the client | JSON directly, padding included |

B's padding trick is the clever part and deserves stating plainly: because every alternative for a field is tokenized as the complete fragment and right-padded to a common width, **answer length is fixed by the schema before generation**, so the canvas cannot shift no matter which value wins. The padding survives into the output — its receipts contain `"line_0_kind": "work_or_goods" ,` — and `json.loads` accepts it.

## Where B pays

**Canvas density.** A costs one position per question; B costs a whole key-value fragment. In B's own evaluation, 21 fields plus a document nearly fill a 256-token canvas, and 5 of 46 nodes had to be split across canvases — with the full document re-prompted per chunk and no cross-canvas KV reuse. Under label codes those same 21 questions would be 21 positions.

**No confidence.** This is the decisive practical gap. B's shipped path exposes **no per-value probability at all**. Selection walks a candidate trie greedily, picking the argmax among still-active candidates position by position — which its own code notes is "greedy projection, NOT an autoregressive sequence likelihood," so the returned value need not even be the joint argmax. Once values are multi-token and padded, no single position's softmax is a distribution over the field's values. A legacy single-token path computes something it explicitly names `uncalibrated_p_true`, used only to rank retrieval candidates.

So B answers the *typed output* half of the [[system-one-model|System One]] proposition and not the *calibrated probability* half — while A returns entropy, `confidence = 1 − H(p)/ln K`, and automatic re-reads on high entropy.

**Grouping costs accuracy.** Same 337 questions, same model, same step budget: **88.4% with one question per canvas, 86.1% grouped**. That 2.3-point drop is the measured price of putting fields on a shared canvas — and it is exactly the isolation that [[jev|Jev]] and [[nimble]] preserve by construction.

## Where A pays

**It needs an engine patch.** A requires seeded canvases, pinned positions, read-only steps and exact-id logprobs — none of which existed in [[vllm|vLLM]] before PR #57250, which merged on 2026-09-22. B runs on stock HuggingFace with a `LogitsProcessor` and a monkeypatched sampler.

**The 26-option ceiling**, and a translation layer the caller must maintain between codes and meanings.

**Label tokens can be adversarial.** A's own server notes that with a long option list, most labels never rank in the top-k and "the model's mass sits on tokens that spell the option name instead" — which is why it needs exact-id logprobs rather than top-k.

## What the evidence does and does not say

B's repository tried A first, on letter codes, and moved away from it — recording a smoke run where the letter-code path put **over 99.98% probability on a wrong department**.

> [!warning] That is not evidence B beats A
> The repo says so itself: the successor differed in "prompt, canvas initialization, constraints, and stopping, so any quality change is not an isolated ablation." The 99.98%-on-wrong-answer result is better read as a *calibration* datapoint than an argument between routes — and a troubling one, since overconfidence on a wrong label is the specific failure the whole [[calibration|calibration]] story is supposed to prevent.
>
> A second such datapoint sits in B's own results: all five of its 4-step disagreements were on cases where the saved reference probabilities were 0.50–0.56 — genuinely ambiguous — yet, in its words, "our restricted probabilities were extremely confident on these ambiguous decisions; agreement does not establish calibration."

Note also that the two projects **never cite each other**. There is no head-to-head, and no shared harness. Any comparison, including this page, is assembled from the outside.

## What they share

Both guarantee validity the same way: the output is reconstructed from a pre-validated template, never parsed from free text. B returned 408/408 valid typed answers in every run, with no JSON repair, no retry and no fallback — invalid output raises rather than degrading.

And neither is [[constrained-decoding]], which is the point most easily missed. Grammar-constrained decoding compiles a schema to an FSM and masks illegal tokens given the prefix so far. A diffusion model has no prefix, so there is nothing for the FSM to advance against — vLLM rejects structured outputs for diffusion models outright, and canvas-aware guided decoding remains an open issue. Seeding a canvas and reading it sidesteps the grammar problem by never needing a grammar. Both routes are that same sidestep, differing only in what they write in the holes.

## Verdict

For a decision service, A is the better-engineered answer: constant canvas cost per question, a real probability, question isolation, and a re-read policy. B's advantage is deployability today on unmodified infrastructure, plus native JSON values with no code-mapping and no 26-option ceiling — and it is the route that actually produced the headline 88.4% against Jev's 90.8%.

Neither project has published a reliability diagram or an ECE of its own. One third-party audit did measure them, and its finding is awkward for both: across five tasks, **open-jev came out the best-calibrated system tested, Jev and GPT-5.6 Terra included, on four of them** — at n=300 per task, where the author notes sub-5-point differences are noise. See [[independent-jev-calibration-audits-2026]].

> [!note] Both routes may be beside the point
> This comparison is between two ways of using a diffusion model, and the diffusion route itself did not win. The open implementations that actually compete with [[jev|Jev]] are encoders and frozen autoregressive logit-readers — [[laya]] is 7.8× faster at 421M parameters — and the pattern has never been ported to any other diffusion LM. Read this page as an account of a well-executed detour, and [[cloze-scoring]] for why bidirectional encoders had the capability all along.

## Related

- [[logit-scoring]] · [[cloze-scoring]] · [[system-one-model]] · [[diffusiongemma]] · [[jev]] · [[openjev]] · [[nimble]] · [[laya]] · [[calibration]] · [[constrained-decoding]] · [[diffusion-language-model]] · [[independent-jev-calibration-audits-2026]]

## References

- [[structured-generation-mode-for-diffusiongemma-mastracci-2026]] — route A
- [[open-jev-diffusiongemma-harness-github-2026]] — route B
