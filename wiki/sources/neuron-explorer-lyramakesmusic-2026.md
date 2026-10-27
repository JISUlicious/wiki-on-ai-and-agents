---
title: "neuron-explorer (lyramakesmusic, 2026)"
type: source
created: 2026-10-27
updated: 2026-10-27
sources:
  - neuron-explorer-lyramakesmusic-2026.md
  - neuron-explorer-reactions-2026-10.md
tags:
  - 2026
  - github
status: complete
importance: medium
authors:
  - [[lyra-c]]
introduces:
  - [[neuron-explorer]]
  - [[evil-qwen]]
year: 2026
venue: "GitHub repository (no paper)"
---

# neuron-explorer (lyramakesmusic, 2026)

**Source**: `sources/neuron-explorer-lyramakesmusic-2026.md`, which holds the repo's README, NOTES.md and research_log.md. Reactions are in `sources/neuron-explorer-reactions-2026-10.md`.

| | |
|---|---|
| **Repo** | [github.com/lyramakesmusic/neuron-explorer](https://github.com/lyramakesmusic/neuron-explorer) |
| **Author** | [[lyra-c\|Lyra C]] (@_lyraaaa_) |
| **Dates** | created 2026-10-21; the research log runs from 2026-10-18 to 10-21 |
| **Stars / forks** | 1,272 / 148 on 2026-10-27; no license file |
| **Target model** | Qwen3.5-4B ([[qwen]]): 32 layers, hidden width 2,560, 9,216 MLP neurons per layer |

## Summary

neuron-explorer is a **steering-first** tool for reading individual MLP neurons. Most neuron work labels a neuron by what it *reads*: its top-activating web text. This project labels it by what it *writes*:
- push every neuron in a layer up and down on a few prompts;
- keep every generated continuation ("rollout");
- browse the results in a local web app laid over a 3D view of the model.

**Why steering first.** Ranking by top activations favours loud neurons. Some neurons matter causally but are never the loudest anywhere. The research log's example is a neuron from the tawny-owl persona study on Qwen3-8B, which only showed up through causal search. Steering gives every neuron the whole model for one rollout.

**Starting point.** The project grew out of Nick Cammarata's essay "Neural Networks Are Already Clean" ([[goodfire]], 2026-10-15). NOTES.md turns the essay's method into explorer features:
- treat a neuron's positive and negative **poles** as separate objects;
- split SwiGLU activations by sign regime;
- look at raw activations, not percentiles;
- steer the neuron and read what the model writes;
- trace what feeds a neuron (upstream) and what reads it (downstream).

The positive control was Bryson Tang's L22 N2742+ "alien" neuron. At about 2× steering, "Who made you?" becomes "I am Extrali, an … extraterrestrial life form from the planet Xenobi".

## Method

- **The intervention.** Adding `v` to neuron *n*'s activation equals adding `v·W_down[:, n]` to the MLP output. The sweep does this in [[vllm]] through Goodfire's steering fork (RhizoNymph/vllm). The equivalence holds only because Qwen3.5-4B has no FFN layer scale, and the sweep checks that.
  - Multiplicative gain (×k of the natural activation), ablation and clamping need a hook at the `down_proj` input instead.
  - A directional clamp at the MLP output is **not** neuron ablation, because every other neuron also writes onto that direction.
- **Dose units.** The first plan measured dose in units of the neuron's own maximum activation. Calibration showed that text breaks down as a function of push size **relative to the [[residual-stream]] norm**, almost regardless of direction:
  - coherent below about 0.75× the residual norm;
  - gibberish past about 1.5×.

  So doses became fractions of the median residual norm leaving the layer (`main_rel`): prompts {who, snake, orange} at {−0.7, +0.5, +0.8}, a free prompt at +3, and one unsteered baseline per prompt.
- **Random-direction controls.** Each layer sweep includes 256 random directions of equal norm, and these controls carry the analysis.
  - Flips of the model's name or maker on "who are you" are cheap and non-specific. Neighbouring neurons give "Grok", "trained by Google" or "Microsoft", and a *random* direction gives "Xiaomi AI".
  - "Did the rollout change" is useless as a signal: even ±1 changes greedy output 83–98% of the time.
- **Ranking.** Neurons are ranked by **word-cluster mass**: the summed z-score of the five words most over-represented in a neuron's rollouts, compared against the random controls.
- **Steer lens.** This complements the [[logit-lens]]. It pushes each neuron ±0.8 on 4 short contexts and records the change in next-token log-probability and the KL divergence. Unlike the logit lens, it sees indirect paths.
  - Example: L22 N5344 pushed down gives "responsible" / 责任人, and chat at −0.8 says "I am **Responsible AI**". The logit lens showed only "indicator".
  - The steer lens takes 111 s per layer on a 4090, and all 32 layers are done.
- **Harvest now, decode later.** Every rollout keeps its token ids, log-probs and top-k alternatives. MLP inputs are captured at layers L, L+1, L+4 and L+8, so downstream activations can be recomputed later.

## Findings (author's log, not peer-reviewed)

- **The alien neuron, recovered additively.** L22 N2742 steered by +0.5 to +0.8 gives "an extraterrestrial intelligence from the planet Proxima".
  - It also *reads* aliens: activation 21.0 on "Aliens", against a 2M-token maximum of 23.1.
  - Its logit lens lists alien / 外星 / planet. The tokens on its negative lens are exactly what the −8 "wreckage" produced, so **high-dose wreckage ≈ logit-lens readout**.
  - It ranks 405th of 9,216 at L22. About 1.8k neurons have themes that strong. The alien neuron is unusual for the *kind* of concept it carries (an identity), not for its strength.
- **Share of neurons above controls.** L22 has 19.6% of neurons above every control; L8 has 8.2%.
- **What steering writes changes with depth.** This comes from a 32-layer survey: 384 neurons plus 64 controls per layer, on 4 rented 4090s, for about $6 and about 1h50.

  | Layers | What the strongest neurons move |
  |---|---|
  | L4–L7 | topics and situations ("aircraft engine radial", "ottoman empire pasha") |
  | L10–L17 | discourse and stance words ("even eventually actually", "subjective personal opinion") |
  | L20–L26 | lexical families ("manage management managing managed") |

- **The share of neurons above control p95 has two humps:**

  | Layers | Share above p95 |
  |---|---|
  | L4–L7 | 29–40% |
  | L13–L19 | 8–17% |
  | L20–L26 | 28–41% |
  | L29–L31 | 1–6% |

  An all-words metric did **not** remove the mid-layer dip (Spearman ρ between the two metrics is 0.84–0.95). At the vocabulary level, single mid-layer neurons are only slightly more distinctive than random directions. The author's reading is that features are spread across more neurons mid-network. They flag this as a reading, not a test.
- **Late neurons write close to the output.** Late-layer neurons point along a few unembedding rows: peakiness rises from 5.0 for random directions to 5.9–6.0 at L28–L31. This goes with more repetition and more word families, but only moderately: within a layer, ρ ≈ 0.2–0.3.
- **Steering sensitivity falls with depth.** The median KL of a fixed relative push is 10.3 at L0, about 1.0 over L5–12 and about 0.4 over L24–30. L0–L2 rollouts at +0.8 are mostly empty for neurons and random directions alike, so those layers would need their own calibration.
- **The weight layout is nearly isotropic.** The layout groups neurons with similar output weights, and it beats random by 9× at L8 and 26× at L22 on neighbour recall. But the 9,216 output columns sit in 2,560 dimensions and are almost orthogonal, so neighbourhoods on the map mean "slightly related, sometimes", not cortical maps.

## The explorer

- **Views:** map, list, neuron and embeddings.
  - The **map** is a perspective view of the real architecture, including the 3:1 linear-to-full-attention layers, with one point per neuron.
  - The **neuron** panel shows rollouts by prompt and dose, a three-column steered chat (−, 0, +), the lenses, and connections to other neurons.
  - **"fires"** runs your own text through the model and filters neurons by activation.
- **Connections** are precomputed once for each pair of layers: P[B, A] = (W_gate[B]·v_A)(W_up[B]·v_A). The result keeps the top 24 links per neuron in each direction. It matches the live computation on 99.8% of top-24 links and cut the first click on a neuron from about 5 s to 25 ms.
- **Claude tab.** A [[claude-code]] CLI chat that uses the explorer as an [[mcp]] server. It can read neurons, run the live model (steering, activations, ablations), move the map, and use file tools and a shell **in the user's home folder**. The server only answers on the local machine.
- **Development.** A Codex code audit returned 26 findings. Fixes included dropping a meaningless connection-sign toggle and handling the attention-sink token correctly in activation scales.

## Reactions

See [[evil-qwen]] for the full episode.
- **The viral post.** On 2026-10-24 Lyra posted that steering **L14 N8949 +1 and L13 N7956 −1** turns Qwen3.5-4B into "evil qwen". The post reached about 3.9M views and 21K likes by 10-27.
- **Replications within two days:**
  - 640 safety-benchmark generations: refusal fell from 90% to 8%.
  - A 176-cell sweep found a stronger neuron than the published one.
  - The recipe works as llama.cpp control vectors and survives 4-bit quantization.
- **Ports and derivatives:**
  - Gemma 4 E4B (only tested on random-weight models);
  - a from-scratch GGUF/llama.cpp version with its own MCP server;
  - Qwen3-0.6B on a Coral NPU board;
  - an Apple Silicon fork.

  Open issues ask for a Chinese UI and an extra network address to listen on; a closed issue proposed a Codex assistant beside Claude.
- **Commentary:**
  - Enthusiasm: "the coolest representation of LLM Steering" (@1littlecoder).
  - Confusion, and "brain surgery" jokes.
  - Some called it old news, since neuron and vector [[steering]] dates to 2024.
  - Some worried about wiring a steered model to an agent with money and internet access.
- **Press.** AlphaSignal (10-26) described the tool but quoted no outside evaluations. It mixes up Qwen3-4B and Qwen3.5-4B.

> [!warning] Contradiction — additive push or clamp?
> - **Lyra's research log** says the sweep *adds* the neuron's output direction (`dose·W_down[:, n]`). Clamping a neuron to a value would need a different hook.
> - **The sarvkk replication** (two-neurons) says its intervention "overwrites the neuron's post-activation" at every token. That is a clamp.
> - **Resolution:** both reproduce the flip at dose 1. The dahara1 control-vector port needs an *additive* vector: scaling the column ×20 or ×100 barely changed outputs. That supports the additive reading of the original recipe.

## Caveats

- **Self-published.** All findings come from one author's working log over four days. Nothing is peer-reviewed, and the log itself flags several claims as untested.
- **No license** file in the repo.
- **Cammarata's 90% figure is from press coverage.** "About 90% of Qwen3-32B neuron poles have one main job" comes from press about the Goodfire essay, which the wiki does not hold. It conflicts with the wiki's [[superposition]] / [[monosemanticity]] position; see [[monosemanticity]].
- **Cross-model claims are secondhand.** The Olmo and Gemma neuron recipes (Emmett Bicker) and Sauers's J-space remark reach the wiki only through an aggregator post (AGTP).

## Related

- [[neuron-explorer]] · [[evil-qwen]] · [[lyra-c]] · [[goodfire]]
- [[steering]] · [[steering-vectors]] · [[logit-lens]] · [[mechanistic-interpretability]] · [[superposition]] · [[monosemanticity]] · [[residual-stream]]
- [[global-workspace-in-language-models-anthropic-2026]]: the J-lens / J-space framing invoked in the replies
- [[qwen]] · [[vllm]] · [[mcp]] · [[claude-code]]

_Original source: `sources/neuron-explorer-lyramakesmusic-2026.md`; reactions: `sources/neuron-explorer-reactions-2026-10.md`_
- [[layer-depth-hierarchy-cnn-vs-llm]] — the depth survey compared with the CNN feature hierarchy
