---
title: "BERT Rediscovers the Classical NLP Pipeline (Tenney, Das & Pavlick)"
type: source
created: 2026-10-27
updated: 2026-10-27
sources:
  - bert-rediscovers-nlp-pipeline-tenney-2019.md
tags:
  - 2019
  - paper
status: complete
importance: high
authors:
  - Ian Tenney
  - Dipanjan Das
  - Ellie Pavlick
introduces:
  - [[probing]]
year: 2019
venue: "ACL 2019 (venue from external record)"
arxiv_id: "1905.05950"
---

# BERT Rediscovers the Classical NLP Pipeline (Tenney, Das & Pavlick)

**Source**: `sources/bert-rediscovers-nlp-pipeline-tenney-2019.md`, text extracted from [arXiv:1905.05950](https://arxiv.org/abs/1905.05950) (v2, 2019-08-09).
- **Authors:** Ian Tenney and Dipanjan Das ([[google-research|Google Research]]), and Ellie Pavlick (Google Research and Brown).

## Summary

The paper asks **where in [[bert|BERT]]** each kind of linguistic information can be read out. It uses **edge [[probing]]**: a small classifier sees only the frozen per-token vectors of a span (or a pair of spans) and must predict a label.

The eight tasks:
- part-of-speech tags (POS);
- constituents and dependencies (sentence structure);
- named entities;
- semantic role labelling (SRL: who did what to whom);
- coreference;
- semantic proto-roles (SPR);
- relation classification.

Two complementary measures:
- **Scalar mixing weights.** The probe learns a softmax weight over layers. Their **centre of gravity** is the layer the probe relies on most.
- **Cumulative scoring.** Probes see layers 0…ℓ. The gain each added layer brings gives an **expected layer**, roughly the depth at which an example gets resolved.

## Results (BERT-large, 24 layers)

- **The classical pipeline order appears inside the network.** POS comes first, then constituents, dependencies, semantic roles, and coreference last. "basic syntactic information appears earlier in the network, while high-level semantic information appears at higher layers." This agrees with Peters et al. 2018: lower layers hold local syntax, higher layers more complex semantics.
- **Syntax is localised; semantics is spread out.** Syntactic tasks concentrate on a few layers. For semantic relations and proto-roles the mixing weights are close to uniform, and hard examples get resolved gradually across nearly all layers.
- **The two measures disagree in a telling way.**
  - Differential scores peak early (layers 1–7), because many examples can be guessed from shallow statistics; 89% of POS tags are already correct at layer 0.
  - Mixing weights concentrate later (layers 9–20), where the hard cases are resolved.
- **The last 1–2 layers can hurt probes.** Distracting features make probe performance drop there.
- **"Stretching effect."** BERT-base (12 layers) shows the same order, with each task at the same *relative* depth.
- **The order is not strict per example.** Higher-level information can revise lower-level decisions:
  - In "he smoked toronto in the playoffs", "Toronto" is first tagged as a geopolitical entity (the city). After the semantic role is resolved, it is relabelled as an organisation (the team).
  - In "china today blacked out a cnn interview", "china today" is reinterpreted as a TV network.

## Caveats (stated)

- Exploratory work on one encoder.
- As with all probing: not finding a pattern doesn't mean it is absent, and finding one doesn't show how it is used.

## Relevance to this wiki

This is the *read-side* evidence for a language-model [[feature-hierarchy]]: local syntax early, semantics later. It mirrors the CNN progression from edges to objects, with one difference. Semantics never settles into a few layers; it stays spread across the network. Combined with write-side steering evidence ([[neuron-explorer-lyramakesmusic-2026]]), it fills the first half of the "hourglass" in [[layer-depth-hierarchy-cnn-vs-llm]].

Note the model type: BERT is a bidirectional *encoder* with no next-token output layer. Its last layers don't have to turn representations back into vocabulary, unlike a decoder LLM's.

## References

- [[probing]] · [[bert]] · [[feature-hierarchy]] · [[transformer-architecture]]
- [[layer-depth-hierarchy-cnn-vs-llm]] · [[visualizing-understanding-convnets-zeiler-2013]]
