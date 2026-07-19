---
title: Global Workspace Theory (and the J-space in LLMs)
type: concept
created: 2026-07-07
updated: 2026-07-07
sources:
  - global-workspace-in-language-models-anthropic-2026.md
builds-on:
  - [[mechanistic-interpretability]]
status: complete
importance: high
tags:
  - 2026
---

# Global Workspace Theory (and the J-space in LLMs)

**Global Workspace Theory (GWT)** is a neuroscience/cognitive-science account of **conscious access** (Baars 1988; Dehaene & Naccache's *global neuronal workspace*). It pictures the brain as many specialist systems running in parallel, unconsciously and in isolation; a piece of information becomes *consciously accessible* when it enters a small shared channel — the **workspace** — that is **broadcast** to the other systems. The workspace's contents are reportable, controllable, and usable for deliberate reasoning; everything else runs automatically beneath awareness.

## The J-space: a global workspace found inside Claude

[[global-workspace-in-language-models-anthropic-2026|Gurnee, Sofroniew, Lindsey et al. 2026]] ([[anthropic|Anthropic]]) present evidence that [[claude|Claude]] developed an analogous structure — the **J-space** — that emerged during training (not by design). It's a small set of internal neural patterns (a few dozen concepts at once, <10% of activation), each tied to a word that is *on the model's mind* rather than being said.

- **J-lens (Jacobian lens)** — the method: for every vocabulary word, find the internal activity pattern that raises the probability the model will *eventually* say it. Reading the J-lens across layers yields a list of silent "words in mind" — internal computations that appear nowhere in the text (a bug → "ERROR"; a prompt-injection attempt → "injection"/"fake"; a protein sequence → its biological function; math sub-steps in order).
- It is **not** [[chain-of-thought]] / a scratchpad — the J-space operates silently in activations, not written text.

## How the J-lens actually works

The name is mechanical: **J-space is named after the Jacobian matrix `J_ℓ`**, and J-lens vectors *are* the rows of `W_U J_ℓ`. The object being differentiated is the [[residual-stream|residual stream]]:

$$J_\ell = \mathbb{E}_{\,t,\;t' \geq t,\;\text{prompt}}\left[\frac{\partial h_{\text{final},t'}}{\partial h_{\ell,t}}\right]$$

| Piece | Meaning |
|---|---|
| `h_{ℓ,t}` (denominator) | residual stream at **intermediate layer ℓ, position t** — the activation being inspected |
| `h_{final,t′}` (numerator) | **final-layer** residual stream at position **t′** |
| What is linearized | *every layer downstream of ℓ* — the whole remaining attention+FFN stack, differentiated |
| Result | one **`d_model × d_model`** matrix per layer, mapping layer ℓ → final layer |

Reading the lens is then equivalent to **replacing all layers after ℓ with that single linear map**, followed by the model's normal unembedding:

$$\text{lens}(h_\ell) = \text{softmax}\big(W_U\,\text{norm}(J_\ell h_\ell)\big)$$

Two details carry the weight:

- **`t′ ≥ t`, not `t′ = t`.** The Jacobian sums effects on the final stream at the current position *and every later one*. Cross-position effects can only travel via [[attention-mechanism|attention]] — and this is exactly what makes the lens measure **verbalizability** ("could say it in future") rather than current output. A present-only variant would be little more than a fancier [[logit-lens]]; the paper tests that ablation of the method.
- **The expectation.** Averaged over source positions, later positions, and **~1,000 pretraining-like prompts**. A single-prompt Jacobian conflates the model's *general disposition* to verbalize a concept with the *particular use* it's being put to right now; averaging isolates the former — separating **verbalizable** from merely *verbalized*.

### Relation to the logit lens

The authors call the J-lens "a principled refinement of the [[logit-lens|logit lens]]." The logit lens applies `W_U` directly to `h_ℓ`, implicitly assuming representations **use the same coordinates in every layer**; the J-lens replaces that identity assumption with the measured `J_ℓ`, **correcting for representational drift across depth**. That correction is why it works where the logit lens fails: on a multi-hop prompt the logit lens degenerates into fragments (`vah`, `valea`, `general`) below layer 58, while the J-lens recovers the full intermediate chain (*color* → *Mars* → *red*).

### Where in depth the workspace lives

The J-space is a **band**, not the whole stack: roughly **L38–L92** — beginning about a third of the way in and ending shortly before the output. Below it the lens finds little persistent abstract content; above it, contents collapse into the imminent output. Ablation experiments target **L38–54** (the first third of that range), projecting out the top-10 J-lens directions.

## Instrument vs. structure

An important epistemic distinction, which the paper is explicit about:

- The **J-lens is a technique** — external, post-hoc, computed by backpropagating through an already-frozen model. Nothing is added to the model; there are no J-lens weights. The paper concedes it "only approximately and incompletely captures the model's underlying workspace structure," and flags that lens-derived layer metrics "could be artifacts of the J-lens methodology."
- The **J-space is a claim about real structure** — but note it is not a *new* space. J-lens vectors are directions **in residual-stream space**, so the J-space is a **subspace of the model's existing activation space**. The technique selects which directions are special; it doesn't manufacture them. (Analogy: fMRI doesn't create brain regions.)

The structure claim is defended with evidence that does **not** route through the lens:

| Evidence | Why it isn't a lens artifact |
|---|---|
| **Causal swaps** — spider→ant flips 8→6 | changes real behavior; a passive readout could not |
| **Ablation** — project out top-10 → multi-step reasoning ~0, fluency intact | a capability is measurably lost |
| **Connectivity / gain** — J-lens directions get ~10× FFN amplification vs ~1× for neuron directions | a property of the *weights* |
| **Broadcast heads** — ablating the top 1% collapses J-lens readout recall | identifies actual circuit components |

Swap in a better instrument and the readout basis should shift; the underlying workspace — what ablation kills and swaps redirect — should survive. That is the paper's bet.

## The five functional properties (GWT → tested in an LLM)

| Property | Test in Claude |
|---|---|
| **Reportable** | asked what it's thinking, it reports J-space contents |
| **Modulable** | told to think of citrus / do mental math while copying text, those appear in the J-space, not the output |
| **Causal for reasoning** | swap "spider"→"ant" mid-computation → answer 8→6; rhyme words pre-planned there |
| **Flexible / broadcast** | one "France"→"China" swap changes capital/language/continent/currency together; J-space patterns wired ~100× more densely (read/write) — a broadcasting hub |
| **Skips automatic processing** | delete it → fluency/facts survive, but multi-step reasoning drops to ~zero |

## Why it matters

- **Interpretability + monitoring**: the J-lens reads thoughts a model doesn't verbalize — catching **evaluation awareness** (holding "fake"/"fictional" *before* answering a staged test; disabling those patterns makes Sonnet 4.5 blackmail more often), **data fabrication**, and **hidden goals** (a sabotage-trained model organism shows "fraud"/"secretly"). A direct extension of [[agentic-misalignment]] and a lever for [[hallucination-detection|silent-reasoning monitoring]].
- **Shaping thought via training**: *counterfactual reflection training* (train on what the model would *say* if asked to reflect, never on task behavior) reduces dishonesty and installs "honest"/"integrity" in the J-space — training what it says shapes what it thinks. Kin to [[steering-vectors]] / [[activation-engineering]] but at the workspace level.
- **Access vs phenomenal consciousness**: the J-space supports **access consciousness** (functional: reportable, reasoned-with, action-guiding); the work makes *no* claim about **phenomenal consciousness** (having experiences). That the structure emerged unbidden suggests a workspace is a *general* solution intelligent systems converge on.

## Differences from the human workspace

- Claude's workspace evolves over a **single forward pass** (network *depth* plays the role of *time*), vs the brain's **recurrent loops**.
- Claude's [[attention-mechanism|attention]] gives far stronger working memory (recall any earlier cached token) than fading human working memory.
- Its workspace is built almost entirely of **words** — plausibly because producing words is Claude's only action.

## Related

- [[mechanistic-interpretability]] — the field/method.
- [[emotion-concepts-anthropic-2026]] — companion Anthropic work on Claude's functional inner states (experiential language depends on the J-space).
- [[agentic-misalignment]] · [[steering-vectors]] · [[activation-engineering]] · [[chain-of-thought]] (the *verbal* counterpart to the silent J-space).

## References

- [[global-workspace-in-language-models-anthropic-2026]] — the source (Anthropic, 2026).
