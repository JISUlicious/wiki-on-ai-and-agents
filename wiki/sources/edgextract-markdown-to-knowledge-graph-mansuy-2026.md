---
title: "edgextract: markdown to knowledge graph with closed decisions (GitHub)"
type: source
created: 2026-10-05
updated: 2026-10-05
sources:
  - edgextract-markdown-to-knowledge-graph-mansuy-2026.md
authors:
  - Raphael Mansuy
year: 2026
venue: "GitHub (Apache-2.0); Rust crate, Python package, browser WASM demo"
introduces:
  - [[knowledge-graph-extraction]]
status: complete
importance: medium
tags:
  - 2026
---

# edgextract: markdown to knowledge graph with closed decisions

An open-source library by Raphael Mansuy ([github.com/raphaelmansuy/edgextract](https://github.com/raphaelmansuy/edgextract)), created 2026-10-03 with 67 stars two days later. It turns markdown into a [[knowledge-graph]] **without asking a model to write the graph**. Instead it asks a small [[system-one-model|decision model]] one closed yes/no question per candidate link and keeps only the answers that clear a cutoff. Its tagline: "closed yes/no decisions, not free-form JSON." It ships as a Rust crate, a Python package and a WebAssembly build that runs the whole pipeline in a browser tab.

It is the first application in this wiki that uses the [[logit-scoring|typed-decision readout]] as a component of a larger pipeline, rather than as a product in its own right.

## How it works

1. Split the markdown into sentences, keeping character offsets.
2. **Propose names** from the ontology's listed names, markdown cues and, optionally, a GLiNER span finder.
3. **Type each name** by lookup. For an unknown span, ask "is this a name?", then pick one kind from the ontology.
4. **Drop pairs the ontology forbids.** Illegal link types are never asked, so the model cannot invent a relation type.
5. **Ask one yes/no per legal link type**, so "uses" and "depends on" can both be true of the same pair.
6. **Keep, review or drop** each answer against two cutoffs in your code. Anything uncertain goes to a review queue for a person instead of silently entering the graph.

"The model never finds a span and never invents a type." The ontology is a short YAML file of entity kinds, relations with domain and range, and optional known names. Questions go to `POST /v1/systemone`, served locally by Ollama 0.35+ or by Tev1 running on WebGPU in the browser. Output is JSON (entities with canonical names and kinds, relationships with the evidence sentence, plus `review` and `rejected` lists), with Cypher and interactive-HTML exports.

## Results

All results are author-reported and single-run. The evaluation spec publishes precision and recall as well as F1, and saves the result files.

**CoNLL04** (288 test sentences, exact-span scoring, no names pre-listed):

| Method | Names P / R / F1 | Links P / R / F1 | Trained on CoNLL04? |
|---|---|---|---|
| SpERT (published) | — / — / 0.889 | — / — / 0.715 | yes |
| edgextract + Tev1, final (shipped) run | 0.758 / 0.633 / 0.690 | **0.414** / 0.365 / 0.388 | no |
| edgextract + Tev1, first run | 0.749 / 0.588 / 0.658 | 0.467 / 0.367 / 0.411 | no |
| Mistral Small, one JSON answer per sentence | 0.595 / 0.699 / 0.643 | 0.266 / **0.434** / 0.330 | no |

The headline is in the error counts. Among zero-shot methods, the closed decisions **kept 218 wrong links against 504** for the chat model writing JSON (154 vs 183 correct), at the cost of some recall. A model trained on the benchmark still wins by a wide margin, and the authors say so.

**Twelve in-house tech notes:** links F1 0.595 for Tev1 vs 0.426 for Mistral Small. Tev1's names F1 of 0.966 is high "partly by design": the names are already in the ontology's list. Two Mistral runs on the same day differed (names F1 0.681 vs 0.796), which illustrates the non-determinism of the generative route.

## What it does not show

The README's "honest limits" section is unusually candid, and the most important points come from it:

- **The cutoffs are not calibrated.** The probabilities are the decision model's raw outputs and must be fitted on your own labels (`edgextract calibrate`). The authors explicitly make no claim about Jev's calibration. This matters, because the whole design rests on the probability being meaningful. See [[calibration]] and [[independent-jev-calibration-audits-2026]].
- **Direction can flip.** The final CoNLL04 run kept 16 reversed links (9 in the first run). The larger span finder improved names but made links slightly worse.
- **No coreference and no cross-sentence links** in this version.
- **Speed:** 7.9 s for a seven-sentence note with Tev1 (4.3 s once the model is loaded), against 77.8 s for Gemma 4 through Ollama and 3.9 s for hosted Mistral. The library's default model is [[nimble|Nimble]], not Tev1.
- The comparison baseline is one chat model, Mistral Small, prompted for JSON. No constrained-decoding baseline was run, so "closed decisions beat JSON" is shown against unconstrained generation only. The README notes that a JSON schema "makes the output well-formed… it does not make it true."

## Why it matters for this wiki

This is a concrete answer to the failure [[selective-forgetting-graph-based-memory-rusu-2026|Rusu et al.]] found in extraction-based graph memory: extraction invents and loses information at write time, and nothing flags it. edgextract makes extraction **abstain**: uncertain links go to a person instead of into the graph. It does not address Rusu's other cost, that extraction discards surface form. The evidence sentence kept with each link is a partial mitigation.

It also confirms that local serving now has a native decision route. Ollama v0.35.0 (2026-09-28) added `/v1/systemone`, which our [[system-one-ecosystem-sweep-2026-09-28|ecosystem sweep]] missed by days.

## Related

- [[knowledge-graph-extraction]] · [[knowledge-graph]] · [[graph-memory]] · [[system-one-model]] · [[logit-scoring]] · [[nimble]] · [[calibration]] · [[structured-outputs]] · [[constrained-decoding]] · [[evoontology-self-evolving-ontology-layer-data-agents-chong-2026]]

## References

- [raphaelmansuy/edgextract](https://github.com/raphaelmansuy/edgextract): README and `specs/0001-implementation/14-evaluation.md` (fetched 2026-10-05)
- [Ollama v0.35.0 release notes](https://github.com/ollama/ollama/releases/tag/v0.35.0): decision models via `/v1/systemone`
