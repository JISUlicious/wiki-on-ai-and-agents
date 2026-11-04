---
title: "Sparks of Weight-Space Programming: LLMs can natively read & write neural network weights"
type: source
created: 2026-11-03
updated: 2026-11-03
sources:
  - weight-space-programming-takehi-2026.md
authors:
  - Rikiya Takehi
  - Jayden Teoh
  - Alex Zhang
  - [[omar-khattab]]
first_author: Rikiya Takehi
introduces:
  - [[weight-space-programming]]
year: 2026
venue: "Blog post (no paper)"
status: complete
importance: medium
tags:
  - 2026
  - blog
---

# Sparks of Weight-Space Programming (Takehi et al., 2026)

**Rikiya Takehi\*, Jayden Teoh\*, Alex Zhang and [[omar-khattab|Omar Khattab]]** (all [[mit|MIT CSAIL]]; \* co-first authors). Blog post, 2 Nov 2026: [rikiyat.github.io/blog/weight-space-programming](https://rikiyat.github.io/blog/weight-space-programming/). There is no paper. The authors released full logs ([github.com/RikiyaT/lm_understand-logs](https://github.com/RikiyaT/lm_understand-logs)) and the edited model ([RikiyaT/Qwen3.5-0.8B-weight-space-programming](https://huggingface.co/RikiyaT/Qwen3.5-0.8B-weight-space-programming)). News coverage: [[gpt-6-astra-edits-qwen-weights-runtimewire-2026]].

## Summary

The post claims that a general-purpose frontier LLM, [[gpt-6-astra|GPT-6 Astra]], can **read and write neural-network weights directly**, with no optimizer and no gradient descent. It names this [[weight-space-programming|Weight-Space Programming]]. Astra was not allowed to write training loops in any experiment. It reasons about what each part of a network computes or should compute, then edits the numbers.

There are eight experiments, in three groups:

1. **Reading weights** (Experiments 1–3 and 7): what was this network trained to do?
2. **Fixing weights** (Experiments 4 and 8): repair a failure by editing.
3. **Writing weights from scratch** (Experiments 5 and 6).

## Experiments

| # | Task | Network | Result |
|---|---|---|---|
| 1 | Which game was this policy trained on? | Four PPO policies of identical shape (17,253 weights); no game named, no tools | All 4 correct (Breakout, Pong, Pac-Man, Freeway) |
| 2 | What shape does this diffusion model draw? | Three 12×12 DDPMs (23,824 weights); no forward pass allowed | Right shape each time, but drawn as **outlines** although the models generate filled shapes. The authors read this as evidence it reads weight geometry rather than simulating the network. |
| 3 | Which language, and what does each head do? | Three 7.2M character-level transformers with input and output layers **removed**; Python analysis allowed, no running | Right language in all 6 attempts. Found exactly the copying (induction) heads that measurement confirms, using the method of Elhage et al. without being told it. |
| 4 | Fix a stuck car policy | Kinetix PPO actor (171,664 weights); one reply, no tools | Edited 3.5% of weights; success 0% → **100% of 1,024 episodes**. It added a small side path that backs the car up, then switches off. |
| 5 | Write game players from zero | MinAtar networks, all weights starting at 0; no data, no rollouts | Beat PPO at 100M steps in all three games. Breakout **517.4** vs 25.3 for PPO at 500M steps; Seaquest 53.1 vs 88.9; Asterix 53.2 vs 55.2. |
| 6 | Write a 9×9 Go player | 1.48M weights, too large for context; Python sandbox, 0–50 forward passes | With 0 passes, wins more than half its games against two smaller PPO nets. With 50, beats an equal-sized PPO network trained on 20M games. |
| 7 | Which of two fine-tuned [[gemma\|Gemma 3 270M]] copies is sycophantic? | Weights and tokenizer; embeddings and output layer frozen during fine-tuning | **Failed from weights alone** in all 4 attempts. Allowed to see attention patterns on texts it writes (not outputs), it picked the [[sycophancy\|sycophant]] in all 3 pairs: head 10.2 looks at the user's suggested answer. |
| 8 | Make [[qwen\|Qwen3.5-0.8B]] better at math | Python kernel over the weights, no GPU; practice problems from GSM8K/MATH train sets; 10, 25 or 50 forward passes | See below |

### Experiment 8: the Qwen edit

The base model loops: 85 of its first 96 practice answers ran to the 32,768-token limit. Qwen's own model card warns that it is prone to thinking loops. Astra traced the looping to **one direction written by the MLPs of layers 15 and 19** and subtracted it from 64 output columns in each layer: 131,072 weights, **0.016%** of the model. The authors compare this to removing a behaviour by deleting a single direction from a model's weights.

| Qwen3.5-0.8B | MATH-500 | GSM8K | IFEval | Hit token limit |
|---|---|---|---|---|
| Base | 7.9 | 13.8 | 21.0 | 93.0 |
| Edited, 10 passes | 12.3 | 25.7 | 49.0 | 87.2 |
| Edited, 25 passes | 14.1 | 25.1 | 39.9 | 86.4 |
| Edited, 50 passes | **48.1** | **67.1** | **62.3** | 33.1 |

The evaluation sets were never shown to Astra. Sampling: temperature 0.6, top-p 0.95, top-k 20, presence penalty 1.5, thinking on.

**The control: Gemma 3 270M**, which rarely loops (8.5% of answers hit its limit). Astra found only ordinary reasoning errors with no single component to switch off, and no edit moved MATH-500 by more than about a point (6.8 → at most 7.7). The authors' own conclusion: editing weights could remove an obstacle that hid an ability, but could not add one the model lacked.

## Caveats

- **A blog post, not a paper.** It has no peer review and reports no seeds or variance. Several cells are single runs.
- **The Qwen gain is unlocking, not new skill.** The authors say so themselves. Base accuracy is low because 93% of answers never finish; the edit lets the model finish answers it could already work out.
- **Only one in three budgets worked.** With 10 or 25 forward passes the loops barely moved (87% still hit the limit). The large gain appears only at 50. IFEval is not monotonic (49.0 → 39.9 → 62.3), which suggests run-to-run noise or different edits at each budget.
- **No random-direction control.** The post does not report what removing a random direction from the same 128 columns does. [[neuron-explorer-lyramakesmusic-2026]] showed that such controls matter: many single-neuron effects there turned out to be non-specific.
- **Reading weights did not reach behaviour.** The one test of reading a *behavioural* trait from a real language model (sycophancy) failed without the attention-pattern tool.
- **One model only.** Every experiment uses GPT-6 Astra. Whether weaker or other frontier models can do the same is not tested.
- **Partial outside check.** Ahsen Khaliq at Hugging Face rebuilt the edited Qwen model from the logs and re-ran its evaluation, according to the post. That checks the released edit, not whether the method finds such edits reliably.
- **The future-work list is speculative.** Auditing for backdoors without running a model, zero-data unlearning, [[recursive-self-improvement]] by editing one's own weights and grafting circuits across architectures are framed as possibilities, with no experiments.

## How it relates to other wiki pages

- **[[evil-qwen]] and [[neuron-explorer-lyramakesmusic-2026]]** (Oct 2026) change Qwen3.5-4B's behaviour through MLP output directions too, but as an *activation push* at inference time that one replicator found cannot be merged into the weights. Astra's edit is the opposite operation: it *removes* a direction from the down-projection columns, so the change is baked into the weights and needs no hook.
- **[[qwen-scope]]** found repetition features in Qwen3 with sparse autoencoders and used them to generate repetition negatives for RL. Same failure mode, located as a direction, but fixed with training rather than a direct edit.
- **[[mechanistic-interpretability]]**: Experiment 3 rediscovered the induction-head analysis of Elhage et al. unprompted; the post positions Weight-Space Programming as a new interpretability tool.
- **Refusal-direction ablation** ("abliteration") is the closest prior technique: projecting a direction out of weight matrices. The post's comparison to "deleting a single direction" points at it.

## Concepts & entities

- [[weight-space-programming]] · [[gpt-6-astra]] · [[omar-khattab]] · [[mit]]
- [[qwen]] · [[gemma]] · [[evil-qwen]] · [[qwen-scope]]
- [[mechanistic-interpretability]] · [[steering]] · [[residual-stream]] · [[sycophancy]] · [[recursive-self-improvement]]

## References

- [Blog post](https://rikiyat.github.io/blog/weight-space-programming/)
- [Logs](https://github.com/RikiyaT/lm_understand-logs) · [Edited model](https://huggingface.co/RikiyaT/Qwen3.5-0.8B-weight-space-programming) · [Hugging Face demo](https://huggingface.co/spaces/akhaliq/weight-space-programming-demo)
- [[gpt-6-astra-edits-qwen-weights-runtimewire-2026]]
