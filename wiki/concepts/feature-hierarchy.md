---
title: Feature Hierarchy (layer depth)
type: concept
created: 2026-10-27
updated: 2026-10-27
sources:
  - visualizing-understanding-convnets-zeiler-2013.md
  - transferable-features-deep-nets-yosinski-2014.md
  - zoom-in-circuits-olah-2020.md
  - bert-rediscovers-nlp-pipeline-tenney-2019.md
  - neuron-explorer-lyramakesmusic-2026.md
status: complete
importance: high
---

# Feature Hierarchy (layer depth)

A **feature hierarchy** is the observation that a deep network's layers represent different *kinds* of features depending on depth. Early layers hold simple, general, local features. Later layers build them into complex, task-specific, more invariant features. The idea is the standard textbook description of deep learning ([[zoom-in-circuits-olah-2020]] calls it the "foundational description"). It is best documented in vision models.

## In CNNs (well established)

| Evidence | Early | Middle | Late |
|---|---|---|---|
| What units respond to ([[visualizing-understanding-convnets-zeiler-2013\|Zeiler & Fergus]]) | edges, colours, corners | textures (mesh, text) | dog faces, bird legs, then whole objects |
| Mechanism ([[zoom-in-circuits-olah-2020\|Zoom In]]) | Gabor filters, curve detectors, high-low frequency detectors | curves → circles; dog-head pathways | car, dog-head and floppy-ear detectors; features in superposition |
| Transferability ([[transferable-features-deep-nets-yosinski-2014\|Yosinski]]) | general: layers 1–2 transfer almost perfectly | worst co-adaptation (layers 4–5) | specific: layers 6–7 |
| Linear readout (Zeiler Table 7) | weakly discriminative: 44.8% on Caltech-101 from layer 1 | rising | 86% from layers 5–7 |
| Invariance and training | sensitive to small shifts; converges in a few epochs | — | quasi-invariant; needs 40–50 epochs |

## In transformer language models (partial, and not monotonic)

- **Read side, encoder.** [[bert-rediscovers-nlp-pipeline-tenney-2019|Tenney et al.]] probe [[bert|BERT]] and find the classical NLP pipeline in order: part-of-speech first, then sentence structure, entities, semantic roles, and coreference last. Syntax sits in a few layers; semantics is spread across nearly all of them.
- **Write side, decoder.** [[neuron-explorer-lyramakesmusic-2026|neuron-explorer]] steers single neurons in Qwen3.5-4B:
  - L4–L7 steer **topics**;
  - L10–L17 steer **discourse and stance** words, and single neurons there are barely more distinctive than random directions;
  - L20–L26 steer **lexical families**;
  - the last layers write almost directly into output tokens.
- **Task stages.** [[discovering-the-gems-in-early-layers-shi-2024|GemFilter]] finds three stages in Llama-3.1-8B: early layers search, early-to-middle layers identify the relevant tokens (from layer 13 of 32), late layers prepare output.

The result is an **hourglass**, not a ladder: abstraction rises through the middle of the network and then falls back toward vocabulary at the output. [[layer-depth-hierarchy-cnn-vs-llm]] compares the two in detail.

## What the hierarchy depends on

- **Receptive field.** In CNNs, locality forces it. In transformers, attention is global from layer 1, so any hierarchy there is learned, not imposed.
- **The objective at the top.** A classifier can stay abstract until the class label. A next-token model must turn its representation back into a token distribution.
- **The measurement.** Read-side methods (top activations, probes) and write-side methods (steering, the [[logit-lens]]) can show different hierarchies in the same network.

## References

- [[visualizing-understanding-convnets-zeiler-2013]] · [[transferable-features-deep-nets-yosinski-2014]] · [[zoom-in-circuits-olah-2020]] · [[feature-visualization-olah-2017]]
- [[bert-rediscovers-nlp-pipeline-tenney-2019]] · [[neuron-explorer-lyramakesmusic-2026]] · [[discovering-the-gems-in-early-layers-shi-2024]]
- [[convolutional-neural-network]] · [[residual-stream]] · [[probing]] · [[circuits]]
