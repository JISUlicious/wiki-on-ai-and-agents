---
title: "Structured generation mode for DiffusionGemma (Jev-like) — vLLM PR #57250"
type: source
created: 2026-09-21
updated: 2026-09-28
sources:
  - https://github.com/vllm-project/vllm/pull/57250
authors:
  - Matt Mastracci
year: 2026
venue: "vLLM pull request (merged 2026-09-22)"
introduces:
  - [[logit-scoring]]
status: complete
importance: high
tags:
  - 2026
---

# Structured generation mode for DiffusionGemma (Jev-like) — vLLM PR #57250

An open pull request against [[vllm|vLLM]] by Matt Mastracci adding the machinery to serve [[diffusiongemma|DiffusionGemma]] as a typed-decision model. This is the primary technical source for the claim that [[jev|Jev]]'s capability is reproducible on open weights.

**Status**: **merged 2026-09-22 at 20:51 UTC** by maintainer LucasWilkinson, hours after approval. All five prerequisite PRs had merged earlier (#57414, #57416, #57417, #57462, #57589; +45/−12 in total). What merged is the core sampler and scheduler work; the `/v1/systemone` server remains in `examples/`, not a built-in endpoint. [[sglang|SGLang]] shipped a native route three days later — see [[system-one-ecosystem-sweep-2026-09-28]].

## The claim

> DiffusionGemma (and likely other models in vLLM) contains all of the necessary components to build a Jev-like model that provides structured answers with bounded choices.

Fix the canvas positions dictated by the output format, leaving only answer slots free. Ask for logprobs at those slots, derive entropy, and resample if entropy is too high. See [[logit-scoring]].

## Size and shape

13 files, +2,760/−154 — but only six non-test files touch vLLM proper, at **+518/−153**. The bulk is `examples/features/structured_diffusion/structured_server.py` (+1,528), a stdlib interposer that is explicitly *not* a vLLM endpoint.

| Piece | Δ | Role |
|---|---|---|
| `models/diffusion_gemma.py` | +321/−146 | all sampler logic |
| `utils/diffusion.py` | +104 (new) | request validation |
| `v1/core/sched/diffusion_scheduler.py` | +56 (new) | `DiffusionAsyncScheduler` |
| config + input processor | +37/−7 | auto-selection, admission |

## How the two key knobs work

**`diffusion_read_only` — "skip the commit forward."** DiffusionGemma alternates denoise steps with a commit step, and normally emits only on commit. The patch adds converged read-only slots to the emit set, then clears the flag that would have scheduled the commit pass: `is_encoder_phase[decode_slots] &= ~read_only[decode_slots]`. The request simply ends on the denoise step where it converged. Read-only slots also get **temperature-1** logprobs rather than schedule-tempered ones, which is what makes softmaxing them directly valid.

**`diffusion_pinned` — holding the template.** Two writes are needed, not one. The pinned positions are forced back to their seed value after accept/renoise, *and* the self-conditioning soft embed at those positions is zeroed. The code comment gives the reason: "A pinned position holds its seed token as input. Zero its soft embed too, or the model's own prediction there reaches the next step through self-conditioning." Pins are meaningless at one step — the example server returns `{}` for `steps <= 1`.

A related optimization skips the self-conditioning matmul entirely when every slot in a tile is capped at one step, since the soft embed would never be read: "The matmul is a full pass over the vocabulary matrix, so skip it."

## How the server maps labels to single tokens

Not by vocabulary lookup — **empirically**. `resolve_template()` renders the answer template with every question at label index 0, tokenizes it, then re-renders per alternative and diffs the token sequences. It demands each label change *exactly one token at the same position*, and rejects two labels colliding on one id. Question types are `noul` (yes/no), `choice` (labels `A`, `B`, `C`…) and `score` (`1`…`9` or `A`…`Z`), 2–26 alternatives.

It requests `logprob_token_ids` for the union of label ids — the exact-id path added by prerequisite #57417 — because top-k alone is insufficient. The comment: "With a long option list most labels never rank in the top-k, and the model's mass sits on tokens that spell the option name instead."

Re-reads are implemented as an `auto` policy: one read, then if max entropy exceeds 0.1, up to three more with different noise seeds, averaged, reporting `confidence`, `stderr` and `agreement`.

## What it does not show

The PR demonstrates **mechanism, not quality**. No accuracy benchmark, no comparison against Jev, no [[calibration]] measurement — no reliability diagram, no ECE. Reported latency figures come from the author and community members on assorted hardware, none independently verified.

> [!warning] The entropy it reports is computed over an unnormalized top-set
> `slot_distribution` computes entropy over the returned top-20 logprobs without renormalizing, which systematically **understates** true entropy and therefore biases the `auto` policy toward deciding one read was enough. A reviewer raised this; it is unchanged in the current diff. Since the re-read trigger is the mechanism that is supposed to catch low-confidence answers, this bears directly on the confidence numbers the server reports.

Three other reviewer findings also remain open: `diffusion_max_steps` is silently clamped with no signal, a read-only request with `max_tokens` below canvas width truncates the canvas, and the `_read_in_flight` guard is void without a step cap.

The generality claim is a hedge ("likely other models") and nothing substantiates it — see below.

## Why this path exists at all

> [!note] Grammar-constrained decoding does not work on diffusion models
> [[constrained-decoding]] compiles a schema to an FSM and masks illegal tokens *at each step, given the prefix already generated*. A diffusion model has no prefix — positions commit out of order — so the FSM has nothing to advance against. vLLM currently **rejects structured outputs for diffusion models with a 400**, and issue #45572 ("canvas-aware structured outputs") remains open, with a DiffusionGemma author commenting: "Guided decoding is indeed tricky for Text Diffusion models […] It is an open problem for which we don't have a good answer yet."
>
> Seeding the canvas and reading the distribution sidesteps the problem entirely by never needing a grammar. That is the deeper reason this approach appeared on a diffusion model first.

## How portable is it?

vLLM has exactly **one** in-tree diffusion LM. The registry holds a single entry (`DiffusionGemmaForBlockDiffusion`); a LLaDA support request (#18532) was closed **not planned** — "It would introduce too much complexity in our code. Maybe in a separate repo" — with requests redirected to `vllm-omni` and a separate `dllm-plugin` project.

The new code splits cleanly: validation, scheduler and config auto-selection are **model-agnostic**, keyed on a `canvas_length` field in the HF config (~190 of the 518 core lines). Every piece of actual mechanism — seed canvas, pin mask, read-only state, emit logic, self-conditioning zeroing — lives in `diffusion_gemma.py` with **no shared base class**. A second diffusion model would inherit the plumbing and reimplement the substance.

## Related

- [[diffusiongemma]] · [[jev]] · [[openjev]] · [[logit-scoring]] · [[system-one-model]] · [[vllm]] · [[constrained-decoding]] · [[structured-outputs]] · [[diffusion-language-model]] · [[constrained-canvas-vs-free-denoise]]

## References

- [vllm-project/vllm#57250](https://github.com/vllm-project/vllm/pull/57250) · prerequisites #57414, #57416, #57417, #57462, #57589
- Extended server, out of tree: [github.com/mmastrac/djev](https://github.com/mmastrac/djev)
