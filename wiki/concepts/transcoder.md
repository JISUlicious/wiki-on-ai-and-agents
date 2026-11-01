---
title: Transcoder
type: concept
created: 2026-10-29
updated: 2026-10-29
sources:
  - sparsified-mlps-vs-transcoders-neurons-in-disguise-shabalin-2026.md
status: draft
importance: medium
tags:
  - 2026
---

# Transcoder

A **transcoder** is a sparse, learned stand-in for an MLP layer in a transformer ([[sparsified-mlps-vs-transcoders-neurons-in-disguise-shabalin-2026]]). Where a [[sparse-autoencoder]] reconstructs an activation from itself, a transcoder maps the MLP's *input* to its *output* through a wide, sparsely active dictionary of features. Each feature has an encoder direction (what it reads) and a decoder direction (what it writes). The source page places transcoders and attribution graphs in the SAE lineage, applied to MLP input-output maps so that [[circuits]] can be traced through readable features instead of neurons.

The case for transcoders rests on MLP neurons being [[polysemanticity|polysemantic]]: if each neuron mixes several concepts under [[superposition]], a learned dictionary should recover cleaner units ([[monosemanticity]]).

## Variants named in the source

- **Per-layer transcoders**: one dictionary per MLP layer. All 13 transcoders studied by Shabalin and Piotrowski are of this kind.
- **Activation functions**: TopK and JumpReLU pipelines behave differently in training (see below).
- **Cross-layer and MoE transcoders**: mentioned with only preliminary results.

## "Neurons in disguise" (2026)

Shabalin and Piotrowski (Anthropic Fellows / [[anthropic|Anthropic]], NeurIPS 2026 workshop) ask what transcoders add over the raw neurons. A gated-MLP neuron is approximately bilinear at high activation, so once split by sign it has one effective encoder and one decoder; with top-k thresholding the MLP is itself a narrow transcoder. Across 13 open per-layer transcoders on Gemma 2/3, Qwen 3 and Llama 3.x (1B–27B, all dense):

- **Neuron features are common late.** In the Gemma family's late layers they rise from 5.9% to 37.1% of the dictionary (1B → 27B) and account for 57.9% to 82.8% of activations.
- **They carry the faithfulness.** Jacobian alignment with the MLP is about 0.3 for the transcoder and for its neuron features alone, vs about 0.05 for the other features (Gemma 3 4B, L22).
- **They are the stable part.** Most features that recur across training seeds are neuron features; in TopK transcoders the non-neuron features stay close to their random initialization (median cosine +0.48). The JumpReLU pipeline differs: most live features rotate.
- **Transcoders still win somewhere.** On bigrams the dictionary feature fires for 75.6% of target bigrams vs about 0% for neurons, and in the "red" case study transcoders merge 2–4 polysemantic neurons into one clean feature.
- **As a replacement model**, sparsified neurons with a linear correction beat transcoders on KL but lose on reconstruction (FVU).

## Scope of the claim

> [!note] How far this goes
> This result bears on the open contradiction recorded on [[monosemanticity]] ("neurons are rarely monosemantic" vs "neurons are already clean"). That page reads it as **partial support, carefully scoped**: neurons look as clean as transcoder features only at their *top* activations and mainly in late layers; over the broad activation range they stay polysemantic, and they miss fine-grained concepts. It does not show that neurons are clean in general, and it does not overturn [[superposition]]. The paper covers base, mostly English, dense models and first-order faithfulness only, and its authors leave the faithfulness of the non-neuron remainder undecided.

The same sign-split ("poles") view of neurons appears in [[neuron-explorer]] and the [[neuron-explorer-lyramakesmusic-2026|lyramakesmusic write-up]], from different evidence.

## Related

- [[sparse-autoencoder]] · [[monosemanticity]] · [[polysemanticity]] · [[superposition]] · [[circuits]] · [[mechanistic-interpretability]] · [[neuron-explorer]] · [[goodfire]]
