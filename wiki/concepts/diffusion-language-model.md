---
title: Diffusion Language Model
type: concept
created: 2026-09-22
updated: 2026-09-22
sources:
  - how-transparent-is-diffusiongemma-engels-2026.md
  - neither-parallel-nor-sequential-diffusiongemma-asaria-2026.md
  - cola-continuous-latent-diffusion-language-model-guo-2026.md
status: complete
importance: high
---

# Diffusion Language Model

A language model that produces text by **iteratively denoising a fixed-size canvas** of positions, rather than predicting one token at a time from left to right. Borrowed from image diffusion, adapted to discrete tokens.

The practical consequences are three: many positions can be finalized per forward pass, attention is **bidirectional** so every position sees every other, and a position already written can be **revised** after later context arrives — something an autoregressive model structurally cannot do.

## How a decode actually runs

Taking [[diffusiongemma]] as the worked example, per [[how-transparent-is-diffusiongemma-engels-2026|Engels et al. §2.1]]:

1. The prompt is encoded **once** into a static KV cache. It is never re-processed, which is why extra denoising steps cost GPU time but not prompt tokens.
2. A canvas of *C* positions is initialized **uniformly at random**, and the self-conditioning matrix **S**⁰ ∈ ℝ^(C×d) to zero.
3. Each step runs a full **bidirectional-attention** forward pass over the canvas, then samples. Temperature interpolates from 0.8 down to 0.4 across steps.
4. State passes between steps as a **soft embedding** of the shaped logits: **S**ᵗ = softmax(ℓ̂ᵗ)·**W**_E. This is a dense vector channel, not text.
5. **Entropy-bounded sampling** keeps low-entropy positions and re-noises the rest. Critically, the accept set is **re-evaluated from scratch each step** rather than locking positions — so commits can be undone.
6. Output longer than *C* uses **multi-canvas sampling**: the finished canvas is appended to the prompt and a fresh one begins. Across canvases the model is therefore autoregressive again.

That last point is easy to miss and matters in practice: a diffusion LM is only non-autoregressive *within* a canvas.

## Two families

| | Discrete / masked | Continuous latent |
|---|---|---|
| Denoises over | token positions (mask → token) | a continuous embedding space |
| Examples | [[diffusiongemma]] | [[cola-continuous-latent-diffusion-language-model-guo-2026\|Cola]], [[elf-embedded-language-flows-hu-2026\|ELF]], [[latent-reasoning-with-normalizing-flows-tu-2026\|NF-CoT]] |
| Output | tokens directly | decoded from latents |

The wiki's continuous-latent entries propose architectures; DiffusionGemma is the one shipped, open-weights production checkpoint, which is why it carries most of the empirical literature. See [[latent-reasoning]] for the broader agenda this sits inside.

## What the marketing gets wrong

> [!warning] "Parallel" is a claim about the canvas, not the commit rate
> Vendor framing ("256 tokens per forward pass") describes canvas width. [[neither-parallel-nor-sequential-diffusiongemma-asaria-2026|Direct instrumentation]] of a shipped checkpoint measures **5.8–25.5 content tokens committed per accept-call** over **3.3–17.1 calls**, landing in a *partial* left-to-right order (tie-aware Kendall τ_b 0.430–0.604 — well below strict autoregression at 1.0, but well above zero). There is no privileged block size: τ_b rises smoothly as the analyst's bin grows, with no jump at 16.

So the honest description is **locally parallel with a global left-to-right drift** — and the drift is regime-dependent. Constrained JSON is the clean exception at τ_b = −0.044, essentially order-independent.

## Costs and where they bite

Committing tokens in parallel hurts most where later steps depend on earlier ones being right. DiffusionGemma trails its autoregressive sibling by 19 points on [[aime|AIME]] 2026 (69.1 vs 88.3) and 5 on MMLU Pro, while matching it on constrained JSON. Google concedes diminishing returns under high-QPS serving, since batching already saturates the hardware — the gain is single-stream and latency-sensitive, where [[arithmetic-intensity]] binds.

There is also an interpretability cost: the soft-embedding channel moves computation out of natural language, which is why [[how-transparent-is-diffusiongemma-engels-2026]] exists. Its finding is reassuring but conditional — opaque serial depth is 28.6× an AR sibling's if the inter-step bottleneck is opaque, 1.1× if it is interpretable, and the paper argues for the latter by showing the channel survives compression to the top-8 logits with no capability loss.

## An unplanned application

Because a masked diffusion model's native operation is *filling declared holes*, it can be given a canvas whose only masked positions are answer slots and then be read rather than run. That turned DiffusionGemma into the substrate for open [[system-one-model|System One]] decision servers in September 2026 — see [[logit-scoring]], [[openjev]], and [[constrained-canvas-vs-free-denoise]]. The order-independence of the structured regime is what makes this work well rather than merely work.

## Related

- [[diffusiongemma]] · [[latent-reasoning]] · [[masked-language-modeling]] · [[logit-scoring]] · [[system-one-model]] · [[constrained-decoding]] · [[speculative-decoding]] · [[arithmetic-intensity]] · [[mixture-of-experts]] · [[edge-inference]]

## References

- [[how-transparent-is-diffusiongemma-engels-2026]] — architecture recap and interpretability audit
- [[neither-parallel-nor-sequential-diffusiongemma-asaria-2026]] — what the commit rule actually is
- [[cola-continuous-latent-diffusion-language-model-guo-2026]] · [[elf-embedded-language-flows-hu-2026]] · [[latent-reasoning-with-normalizing-flows-tu-2026]]
