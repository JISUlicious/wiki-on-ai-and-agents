---
title: "Harness Continual Learning: Continual Adaptation Beyond Model Parameters"
type: source
created: 2026-09-05
updated: 2026-09-29
sources: []
arxiv_id: "2608.19013"
year: 2026
authors:
  - Borui Kang
  - Jinrui Gu
  - Junhan Lv
  - Wenbin Li
  - Lei Wang
  - Yang Gao
tags:
  - 2026
  - paper
status: complete
importance: high
---

# Harness Continual Learning: Continual Adaptation Beyond Model Parameters

**Borui Kang, Jinrui Gu, Junhan Lv, Wenbin Li (corresponding), Lei Wang, Yang Gao** (State Key Laboratory for Novel Software Technology, Nanjing University; University of Wollongong) — [arXiv:2608.19013](https://arxiv.org/abs/2608.19013), cs.LG, 19 Aug 2026. Preprint; no venue given. No code link in the paper.

## Summary

[[continual-learning]] has been organized around one substrate — model weights — and one failure — catastrophic forgetting. This paper moves the learning object *outside* the model. Modern agents adapt through a harness of prompts, memories, tools, skills, and routing rules that persists across interactions while the foundation model stays frozen; the authors call studying a *sequence* of such harness updates **Harness Continual Learning (HCL)**, and name the resulting failure **harness-level forgetting**: "a memory update can change the evidence retrieved for an earlier query; a skill revision can alter [[tool-use|tool use]]; and a routing edit can break a previously successful workflow. An update that helps recent cases can therefore turn an earlier correct answer, valid tool call, or successful action trajectory into a failure without changing the foundation model."

The framework has two halves. The **state** is four jointly versioned components — Task Interface (prompts, templates, parsing rules), Experience Memory (bounded raw records plus LLM-summarized abstract entries), Capability Map (external tools plus *inner skills* consolidated from abstract memory), Adaptive Router (routing prompts, selection criteria, workflow templates) — treated as one candidate harness, so a proposal either enters whole or not at all. The **update rule** is **guarded harness evolution**: a *Continual Optimizer* (the same frozen model, prompted with the deployed harness and post-execution evidence) drafts candidates component by component, and a *Continual Evaluator* commits a candidate only if it passes three checks under identical model, decoding, tool, environment, and seed conditions — **current improvement** on the task's validation cases (Δ ≥ δ), **historical retention** on an *anchor set* of earlier cases (the count of previously solved anchors that now fail must satisfy D ≤ B), and **validity** (syntax, schema compliance, legal tool use). Anchors are used only by the Evaluator and are invisible to the Optimizer.

The evidence is that harness evolution accumulates capability *and* measurably forgets, and that the tolerance B is a dial on the stability–plasticity trade-off. On [[alfworld|ALFWorld]] with a frozen Qwen3.5-9B, the retention-strict profile reaches **61.74%** final average against **55.56%** for the best memory baseline (RAG), while the permissive profile scores 62.98% at **10.94 average forgetting** versus 2.64. In a four-task textual stream with DeepSeek-V4-Flash the permissive profile lifts the final average from 45.50 (zero-shot) to **64.70**; in a four-task multimodal stream with Qwen3.6-27B the strict profile reaches **68.92** against 42.73 for DGG. The most interesting result is the tolerance sweep: forgetting rises monotonically with B (0.39 → 3.45) but final performance does **not** — the unrestricted gate is the *worst* (60.13) and B=1 the best (63.46). The authors' reading: "without historical constraints, locally beneficial updates may overwrite reusable harness contents, weakening both retention and the experience or capabilities available for later tasks."

## Method details worth keeping

- **Bounded raw memory.** Raw Memory "retains a fixed number of interactions from each task in arrival order"; Abstract Memory is an LLM summary of it, and inner skills are a further LLM consolidation of abstract entries. This is *not* an append-only store — but the paper never evaluates the consolidation step and lists "harness-content consolidation" as unresolved in the conclusion.
- **Sequential per-component search.** When several components need revision the Optimizer takes them in a predefined order, drafts up to K alternatives for each, evaluates each by swapping only that component into the current candidate, keeps the best admissible one as the base for the next component, and leaves a component unchanged if no alternative passes.
- **Anchor construction.** At the end of each task, anchors are chosen by a predefined ratio of previously *successful* and *failed* cases (slots backfilled from the other group if one runs short); each anchor is a raw input plus a fixed binary success criterion (Appendix C: exact-match for MuSiQue/HotpotQA, IoU ≥ 0.5 for detection/grounding, sentence-CIDEr ≥ 0.5 for captioning, VQA consensus = 1.0, goal predicate within 50 steps for ALFWorld, inventory/world-state predicate for Minecraft skill tests). A RefCOCO IoU drop from 0.68 to 0.41 counts as one loss; gains on other anchors do not offset it.
- **Gate thresholds.** Main profiles require ≥1 additional correct validation case; the sweep requires ≥2 of 80 and ≥90% output-format compliance. Stability-HCL sets B=0, Plasticity-HCL sets B=∞. If several candidates pass, a composite of current performance, validity, and retention ranks them, ties broken randomly.
- **Models per setting** (the same frozen model is used for every arm within a setting): ALFWorld Qwen3.5-9B; Minecraft and the main multimodal stream Qwen3.6-27B; textual reasoning DeepSeek-V4-Flash; component ablation Qwen3.5-4B.

## Key points

**ALFWorld** (Table 2; six categories learned in order, 10 training episodes each, ≤50 steps; final success on the 134 official evaluation episodes; forgetting averaged over the first five categories). MemP and MemRL are re-implemented inside the HCL harness with common interface, capability library and environment stack, varying only memory management:

| Method | Pick | Look | Clean | Heat | Cool | Two-object | Final Avg ↑ | Avg Fgt ↓ |
|---|---|---|---|---|---|---|---|---|
| Static Harness | 95.80 | 66.70 | 25.80 | 26.10 | 9.50 | 58.80 | 47.12 | – |
| RAG baseline | 95.80 | 83.30 | 41.90 | 39.10 | 14.30 | 58.80 | 55.56 | **1.74** |
| MemP | 95.80 | 83.30 | 48.40 | 34.80 | 9.50 | 47.10 | 53.15 | 5.18 |
| MemRL | 87.50 | 66.70 | 29.00 | 60.90 | 23.80 | 41.20 | 51.51 | 5.64 |
| **Stability-HCL** (B=0) | 100.00 | 83.30 | 51.60 | 30.40 | 28.60 | 76.50 | 61.74 | 2.64 |
| **Plasticity-HCL** (B=∞) | 100.00 | 77.80 | 41.90 | 39.10 | 19.00 | 100.00 | **62.98** | 10.94 |

Plasticity-HCL solves every Two-object episode (the *last* task) and pays for it with 10.94 forgetting — the cleanest single illustration of harness-level forgetting in the paper. Note that RAG, which never revises procedures or routing, has the lowest forgetting of any adaptive method; forgetting is a cost of *editing*, not of *retrieving*.

**Minecraft** (Qwen3.6-27B, 50-task curriculum, retention-oriented profile with B=0 applied to retained skill tests): HCL completes all 50 tasks while the Static Harness plateaus at 15; cumulative environment actions 83 (HCL) vs 88 (MemRL) vs 91 (MemP). Appendix A is candid that "completed tasks are not systematically replayed after every update," so Minecraft measures skill-level, not task-level, retention.

**Textual reasoning** (Table 3; MuSiQue → ProofWriter → GSM8K → HotpotQA; 250 adaptation / 50 validation / 500 test per task; DeepSeek-V4-Flash frozen):

| Method | MuSiQue | ProofWriter | GSM8K | HotpotQA | Final Avg ↑ | Avg Fgt ↓ |
|---|---|---|---|---|---|---|
| Zero-shot | 35.00 | 42.80 | 49.40 | 54.80 | 45.50 | – |
| Stability-HCL | 27.60 | 73.00 | 50.40 | 57.80 | 52.20 | **0.00** |
| Plasticity-HCL | 29.00 | 77.00 | **92.00** | 60.80 | **64.70** | 0.07 |

Two things the prose does not dwell on. Both HCL profiles finish **below zero-shot on MuSiQue** (35.00 → 27.60 / 29.00), the first task in the stream — a regression relative to the frozen model's own baseline that the forgetting metric cannot see, because Fgt measures decline from the best *observed* score of the evolving harness, not from zero-shot. And the strict profile buys zero forgetting at the price of nearly all the GSM8K gain (50.40 vs 92.00).

**Multimodal perception** (Table 4; COCO detection → COCO captioning → RefCOCO grounding → VQAv2; same splits; Qwen3.6-27B frozen; DGG is a gradient-based multimodal continual-tuning baseline):

| Method | Detection | Caption | Grounding | VQAv2 | Final Avg ↑ | Avg Fgt ↓ |
|---|---|---|---|---|---|---|
| Zero-shot | 4.27 | 25.47 | 43.00 | **84.87** | 39.40 | – |
| DGG | 29.58 | 29.77 | 48.96 | 62.60 | 42.73 | 0.26 |
| Plasticity-HCL | 64.14 | 37.31 | 90.60 | 79.80 | 67.96 | 0.81 |
| **Stability-HCL** | **65.34** | **39.41** | **91.60** | 79.33 | **68.92** | **0.22** |

Here the strict profile wins on *both* axes, and the largest gains are on detection and grounding, where the harness must impose a spatial output schema. VQAv2 is again below zero-shot. Zero-shot detection at 4.27 for a 27B model (against 35.11 for the 4B model in the ablation stream) suggests the zero-shot rows largely measure output-format compliance — which is exactly the thing the Task Interface fixes, so these gains are partly interface engineering rather than learning.

**Stability–plasticity sweep** (Table 5; textual stream, DeepSeek-V4-Flash, 300 adaptation / 80 validation / 600 test per task, 80 anchors per earlier task, δ = 2 validation cases, **40 proposals per profile, ten per stage** — a matched proposal budget):

| Tolerance b | MuSiQue | ProofWriter | GSM8K | HotpotQA | Final Avg ↑ | Avg Fgt ↓ |
|---|---|---|---|---|---|---|
| 0 | 27.83 | 73.33 | 84.33 | 59.50 | 61.25 | **0.39** |
| 1 | 24.83 | 77.50 | **92.33** | 59.17 | **63.46** | 1.22 |
| 3 | 26.83 | 79.83 | 83.00 | 58.50 | 62.04 | 2.00 |
| ∞ | 28.33 | 71.00 | 82.00 | 59.17 | 60.13 | 3.45 |

Forgetting is monotone in b; performance is not. The residual 0.39 at b=0 is the paper's own measurement of anchor-set coverage: "Preserving all anchors currently solved by H_n cannot guarantee unchanged behavior on historical cases not represented by A_n." Compare this table with Table 3: the same b=0 configuration scores **61.25 here and 52.20 there** (GSM8K 84.33 vs 50.40), with only the data budget, δ, and proposal count changed. No seeds or error bars are reported anywhere in the paper, so a 9-point swing between protocols has no noise floor to be read against.

**Component ablation** (Table 6 / Table 10; multimodal stream, Qwen3.5-4B, balanced configuration; disabling a component freezes its initial contents while the other three evolve):

| Variant | Detection | Caption | Grounding | VQAv2 | Final Avg ↑ | Avg Fgt ↓ | Commits |
|---|---|---|---|---|---|---|---|
| Zero-shot | 35.11 | 22.98 | 0.00 | 81.27 | 34.84 | – | – |
| **Full HCL** | 53.07 | 36.09 | 87.60 | 76.87 | **63.41** | 0.45 | 18 |
| w/o Interface update | 53.45 | 33.56 | 87.80 | 74.67 | 62.37 | 0.11 | 24 |
| w/o Memory update | 55.50 | 28.95 | 88.00 | 76.67 | 62.28 | 0.83 | 46 |
| w/o Capability update | 55.11 | 34.16 | 86.40 | 76.80 | 63.12 | **0.06** | 16 |
| w/o Router update | 53.59 | 36.68 | 87.40 | 73.40 | 62.77 | 0.14 | 4 |

The full-vs-ablated spread is ~1.1 points, small enough that without variance it is hard to rank components; the authors' own reading is that Memory and Interface matter most, and that freezing Memory both hurts captioning and *raises* forgetting to 0.83. The commit counts are the more telling column: with memory frozen the loop commits 46 times against 18, i.e. it keeps churning other components to compensate. The authors caution these counts are trajectory-specific and not acceptance rates.

## Where it breaks

- **No variance.** Every number is a single run; the paper's own Table 3 / Table 5 pair shows the same gate configuration moving 9 points under a protocol change.
- **The Evaluator's reliability is never measured.** The gate's ground truth is a finite validation set (50–80 cases, δ of 1–2) and a finite anchor set (80 per earlier task); the residual forgetting at b=0 is the only evidence about coverage, and gate accuracy is not reported as a function of stage. Streams are 4–6 stages long, so the deep-trajectory regime is barely entered.
- **Retention cost scales with the stream.** Every candidate reruns all anchors for every earlier task, so the gate is O(T) executions per candidate and O(K × components × T) per stage. The conclusion names "efficient retention evaluation" and "evaluation over longer interaction streams" as open.
- **Baselines are memory systems, not search.** RAG, MemP, MemRL, DGG — no test-time-scaling arm at matched compute, and per-candidate execution cost is never totalled. [[rethinking-evaluation-harness-evolution-wang-2026|Rethinking]]'s objection — that harness evolution is a search procedure and must be compared to plain search at matched budget — applies in full. Test sets are disjoint from validation and anchors but drawn from the same tasks; there is no held-out task family.
- **Regressions below the frozen model are invisible to Fgt.** MuSiQue and VQAv2 end below zero-shot under every HCL profile; the forgetting metric only measures decline from the harness's own best.
- **Consolidation is asserted, not evaluated.** Raw → Abstract → inner skills is the mechanism that would keep the store from ratcheting, and it is never ablated or measured.

> [!success] Resolved contradiction — "compounding rather than overwriting"
> [[self-improving-agent]] states that "skills, prompts, or scaffold edits are appended; the agent's capability grows monotonically and side-steps the catastrophic forgetting of gradient-based continual learning." Two 2026 results already attack the *monotonic* half (see [[catastrophic-remembering]]). This paper attacks the *side-steps forgetting* half from a third direction: with the model frozen, committed harness edits produce **measurable loss of previously reliable behavior** — 10.94 average forgetting on ALFWorld under a permissive gate, 3.45 in the textual sweep — and the *unrestricted* gate ends with the *worst* final harness. Forgetting was not escaped by leaving the weights alone; it moved to the substrate that was supposed to be immune.
>
> **Resolved 2026-09-29:** [[self-improving-agent]]'s mechanism section was rewritten to reflect this finding.

> [!note] Three failure modes, one substrate map
> - **Catastrophic forgetting** — weights; a gradient step for a new task overwrites an old one ([[continual-learning]]).
> - **Catastrophic remembering** — harness, *append* path; the store grows without provenance and degrades ([[catastrophic-remembering]]; six-paper convergence).
> - **Harness-level forgetting** — harness, *edit* path; a committed revision to any coupled component (interface, memory, skills, router) breaks earlier behavior while the model is untouched (this paper).
>
> The [[continual-learning]] page's framing "gradient methods overwrite; appending methods hoard" is therefore incomplete: harness methods that *revise* rather than append get the overwriting failure back. The two harness-side failures are duals of each other — the fix for remembering is deletion and consolidation, and deletion is precisely the operation that triggers forgetting. HCL's answer is replay-shaped (an anchor set is a rehearsal buffer for behavior) and it explicitly imports that vocabulary in §3.4.

> [!note] Relation to Rehearse's confidence cliff
> [[rehearse-confidence-cliff-autoresearch-ji-2026|Rehearse]] found that the *pre-execution LLM judge* deciding which modification to trial falls from 82.8% to 56.9% selective accuracy as successes accumulate, while committing more often. HCL's Continual Evaluator is a different object: it is **not an LLM judgment but an execution** — validation cases and anchors are rerun under both the deployed and candidate harness with the seed held fixed, and success is a fixed binary predicate. So the confidence-cliff mechanism does not apply to HCL's gate by construction; HCL pays the execution cost Rehearse was built to avoid, and its gate cannot drift in the way a judge does. What HCL does *not* do is measure its own gate: the proxy can still fail through coverage (residual forgetting at b=0), through small validation sets (δ of 1–2 cases), and through the Optimizer — the same frozen LLM Rehearse studied — whose per-stage proposal quality is never reported. Rehearse's observation that helpful-modification rate falls 70% → 43% with depth would surface in HCL as a declining per-stage commit rate; the paper reports only totals. One structural echo survives: relaxing the gate (b=∞) commits more and ends worse, which is the same shape as Rehearse's "coverage rises as accuracy falls," reached by a different mechanism.

## Related

- [[continual-learning]] — the parent concept; HCL relocates its object from θ to the harness and maps replay/representation/architecture/regularization onto Memory/Capability Map/Router/Evaluator in §3.4.
- [[catastrophic-remembering]] — the append-side failure; HCL's bounded Raw Memory and LLM consolidation are a counter-design that is never tested against it.
- [[self-improving-agent]] — the "compounding rather than overwriting" bullet this paper contradicts.
- [[agentic-harness-engineering]] — HCL's stated difference from the AHE cluster: retention across *committed* updates as an explicit objective, not next-configuration quality. Its anchor gate is a direct answer to that page's "regression blindness" open problem.
- [[agentic-harness-engineering-lin-2026]] · [[self-harness-harnesses-that-improve-themselves-zhang-2026]] · [[harness-updating-is-not-harness-benefit-lin-2026]] — cited as the harness-optimization line whose "repeated improvement alone does not provide a general retention criterion."
- [[harness-r1-learning-to-edit-runtime-harnesses-shao-2026]] — the other 2026 loop that reruns *successes* to count regressions; Harness-R1 does it within a batch, HCL across the whole history via anchors.
- [[rethinking-evaluation-harness-evolution-wang-2026]] — the matched-budget-search objection HCL does not address.
- [[evo-bench-can-language-models-improve-agent-harness-huang-2026]] — measured the same pathology from the evolver side: two of three low scorers froze a revision *worse* than one they had already found.
- [[rehearse-confidence-cliff-autoresearch-ji-2026]] — see the callout; execution-gate vs judge-gate.
- [[code-as-harness]] — the four components are the standard harness anatomy (interface, memory, capability registry, router) made jointly versioned.
- [[memory-management]] · [[memory-consolidation]] · [[procedural-memory]] · [[skill-library]] — Raw/Abstract Memory and inner skills are the consolidate-and-curate rows of the memory lifecycle.
- [[alfworld]] · [[voyager]] — the two open-world settings; the Minecraft curriculum is Voyager's.
- [[retrieval-augmented-generation]] — the RAG baseline: lowest forgetting of any adaptive method, because it never revises.
- [[react]] · [[toolformer]] · [[memgpt]] · [[reflexion]] — the systems §2.1 cites as developing individual harness functions.
- [[harness-vs-model-attribution]] — every gain here is with the model frozen; a controlled datapoint for the attribution question.
- [[agent-evaluation]] — the Avg_T / Fgt_T protocol (Eq. 15) borrowed from model-centric continual learning.
- [[llm-as-a-judge]] — contrast: HCL's gate is executed, not judged.
- **No wiki page yet:** harness-level forgetting (the concept this paper names); MemP, MemRL, DGG (baselines); MuSiQue, ProofWriter, HotpotQA, RefCOCO, VQAv2 (task sources).

## References

- [arXiv:2608.19013](https://arxiv.org/abs/2608.19013)
