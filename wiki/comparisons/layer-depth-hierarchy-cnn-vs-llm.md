---
title: "Layer-depth hierarchy: CNNs vs transformer LLMs"
type: comparison
created: 2026-10-27
updated: 2026-10-27
sources:
  - visualizing-understanding-convnets-zeiler-2013.md
  - transferable-features-deep-nets-yosinski-2014.md
  - feature-visualization-olah-2017.md
  - zoom-in-circuits-olah-2020.md
  - bert-rediscovers-nlp-pipeline-tenney-2019.md
  - neuron-explorer-lyramakesmusic-2026.md
  - discovering-the-gems-in-early-layers-shi-2024.md
status: complete
importance: high
---

# Layer-depth hierarchy: CNNs vs transformer LLMs

**Question.** In CNNs, early layers detect fine detail and later layers detect larger patterns. Do transformer language models show the same layer-by-layer [[feature-hierarchy]]?

**Short answer: half of it does.**
- **The general-to-specific rule holds in both.** Early layers are generic and fragile to small input changes. Late layers are specific to whatever the model has to output.
- **The rest differs.** In a CNN "larger" means *spatially* larger, because the receptive field grows with depth. In an LLM, attention sees the whole context from layer 1, so depth adds composition and abstraction instead.
- **The output space differs.** A CNN's output is a set of abstract classes. A decoder LLM's output is a vocabulary. So the LLM's last layers turn back toward words, and its hierarchy looks like an **hourglass**, not a ladder.

## Side by side

| | CNN (AlexNet / ZFNet, InceptionV1) | Transformer LM: read side (what is *encoded*) | Transformer LM: write side (what a unit *causes*) |
|---|---|---|---|
| **Early** | Edges, colours, Gabor filters; corners; curve and high-low frequency detectors. Universal across models. ([[visualizing-understanding-convnets-zeiler-2013\|Zeiler]], [[zoom-in-circuits-olah-2020\|Zoom In]]) | Part-of-speech and local syntax are decodable early; 89% of POS tags are already right at layer 0 ([[bert-rediscovers-nlp-pipeline-tenney-2019\|Tenney]], BERT) | Qwen3.5-4B L4–L7 neurons steer **topics and situations**. L0–L2 are hypersensitive: a fixed push has a median effect (KL) of 10.3 at L0, against about 1.0 at L5–12 ([[neuron-explorer-lyramakesmusic-2026\|neuron-explorer]]) |
| **Middle** | Textures, then object parts (dog faces, bird legs, wheels). Neighbouring layers are most tightly co-adapted here (Yosinski layers 4–5). Superposition: the car feature spreads into dog neurons at mixed4c. | Sentence structure, then entities and semantic roles. Probes rely most on layers 9–20 of 24. Semantic tasks spread across almost all layers. | L10–L17 steer **discourse and stance** words. Only 8–17% of neurons beat random directions, the lowest share of any layer band. |
| **Late** | Whole objects with pose invariance. Class-specific, the least transferable layers ([[transferable-features-deep-nets-yosinski-2014\|Yosinski]] layers 6–7) | Coreference last. The final 1–2 layers can *hurt* probes. | L20–L26 steer **word families** ("manage / management / managing"). Neuron outputs line up with a few vocabulary rows, so a push makes output repetitive. L29–L31 are near the output. |
| **Overall shape** | A ladder: local and concrete → global and abstract → class | Rising abstraction, with semantics spread out | An hourglass: topic → stance → words |

[[discovering-the-gems-in-early-layers-shi-2024|GemFilter]] gives a task-level view of the same LLM picture in Llama-3.1-8B. Early layers search the context, and from layer 13 of 32 the model has identified the relevant tokens. Its authors read the late layers as "preparing output".

## Where the analogy holds

1. **Early layers are generic and sensitive.**
   - CNN layer-1 features are Gabor filters in every model, and small image shifts change them dramatically (Zeiler Fig. 5).
   - LLM layers 0–2 are the most fragile to a push of fixed relative size. neuron-explorer could not get meaningful rollouts from them at its standard dose.
2. **Late layers are specific to the output space.** Yosinski's general-to-specific transition explains both cases:
   - a CNN's output space is object classes, so its late units are object detectors;
   - a next-token model's output space is the vocabulary, so its late units sit close to token families.

   The lexical look of late LLM neurons does not reverse the hierarchy. It is the same "specific to the task" endpoint, applied to a task whose labels are words.
3. **The middle is the most entangled region.**
   - In CNNs, superposition shows up in mid-to-late layers (mixed4c), the polysemantic unit 4e:55 sits at mixed4e, and co-adaptation peaks at layers 4–5.
   - In LLMs, single mid-layer neurons are least distinctive against random directions, and semantic information is least localised (Tenney).

   Both fit [[superposition]] being densest where the most intermediate features must be represented. That is a reading across sources, not a measured result.
4. **Features are built from earlier features.**
   - CNN circuits assemble curves from lines and dog heads from eyes and snouts ([[circuits]]).
   - In BERT, later layers revise earlier decisions: "Toronto" is relabelled from the city to the team once its semantic role is known. The pipeline order holds on average but can be revised in both directions.

## Where it breaks

| Factor | CNN | Transformer LLM |
|---|---|---|
| Field of view | Grows with depth (7×7 patch → most of the image), so the hierarchy is partly **forced** by the architecture | Global from layer 1, since attention can reach any position; any hierarchy is **learned** |
| What "bigger" means | Spatial extent: edge → part → object | Composition and abstraction: token → phrase → topic or stance → plan for the next word |
| Top of the network | Abstract class: stays abstract to the end | Token distribution: must turn back into vocabulary (each position becomes "what comes next"; [[residual-stream]]) |
| Shape over depth | Ladder, getting steadily more discriminative (Caltech-101 SVM accuracy 44.8% from layer 1 → 86% from layers 5–7) | Hourglass |
| Main interpretability evidence | Read side: what input makes a unit fire ([[feature-visualization]]) | Both sides, which can disagree: probes read what is encoded; steering and the [[logit-lens]] show what a unit writes |
| Units | A channel tied to a spatial position | MLP neurons writing into one shared [[residual-stream]], plus attention heads |

**A trap in the comparison.** neuron-explorer measures what a neuron *writes*, while the classic CNN maps show what a unit *reads*. A late LLM neuron *reads* abstract context but *writes* near the vocabulary, because it sits next to the output. In the same way, CNN layer 1 reads pixels because it sits next to the input. So "early LLM layers steer topics" doesn't mean early layers *see* topics. It means that pushing an early layer moves the output along topics. Sources that measure the same side should be compared with each other.

## Gaps

- **No direct LLM counterpart to Zeiler's figures.** There is no decoder-LLM study that maps what units *read* at each depth with the same method. The read-side evidence here is Tenney's, on BERT, an encoder whose last layers don't produce vocabulary. Decoder-side read evidence (logit-lens and tuned-lens progressions, layer-probing of Llama-family models) is not ingested.
- **A single model and author.** The write-side LLM evidence is one 4B model from one author, and its ranking metric favours content words. An all-words variant kept the mid-layer dip; a lexical-free metric was never run.
- **One figure set not extracted.** The edges → textures → patterns → parts → objects figure series in Feature Visualization was not captured in text. The CNN progression above is sourced from Zeiler & Fergus and Zoom In instead.

## Related

- [[feature-hierarchy]] · [[convolutional-neural-network]] · [[transfer-learning]] · [[probing]] · [[circuits]] · [[superposition]] · [[polysemanticity]]
- [[alexnet]] · [[inceptionv1]] · [[bert]] · [[neuron-explorer]]

## References

- [[visualizing-understanding-convnets-zeiler-2013]] · [[transferable-features-deep-nets-yosinski-2014]] · [[feature-visualization-olah-2017]] · [[zoom-in-circuits-olah-2020]]
- [[bert-rediscovers-nlp-pipeline-tenney-2019]] · [[neuron-explorer-lyramakesmusic-2026]] · [[discovering-the-gems-in-early-layers-shi-2024]]
