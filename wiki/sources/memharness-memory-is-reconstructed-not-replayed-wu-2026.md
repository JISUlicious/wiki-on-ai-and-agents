---
title: "MemHarness: Memory Is Reconstructed, Not Replayed"
type: source
created: 2026-08-17
updated: 2026-08-17
sources: []
arxiv_id: "2607.28272"
authors:
  - Rong Wu
  - Daocheng Fu
  - Licheng Wen
  - Xuemeng Yang
  - Shu Zou
  - Jianbiao Mei
  - Yuxin Wang
  - Hairong Zhang
  - Yu Yang
  - Tao Hu
  - Cong Zhang
  - Botian Shi
  - Pinlong Cai
year: 2026
status: complete
importance: high
tags:
  - 2026
  - paper
---

# MemHarness: Memory Is Reconstructed, Not Replayed

**Rong Wu, Daocheng Fu, Licheng Wen, Xuemeng Yang, Shu Zou, Jianbiao Mei, Yuxin Wang, Hairong Zhang, Yu Yang, Tao Hu, Cong Zhang, Botian Shi, Pinlong Cai** — [arXiv:2607.28272](https://arxiv.org/abs/2607.28272), 30 Jul 2026. Preprint; no venue given.

## Summary

Memory-augmented agents retrieve a past experience and paste it into the context **verbatim**, regardless of whether it still applies. The paper's diagnosis: stored experience is abstract and general, decision-time state is concrete and shifting, and closing that gap by concatenation produces **negative transfer** — the retrieved memory actively makes the agent worse. Humans, the authors note, do not recall episodes verbatim; they reorganize and adapt them to the present.

MemHarness inserts a reconstruction step between retrieval and action. Each memory entry is a tuple `m_i = (e_i, o_i^src)` — an abstract strategy **plus the source observation it was distilled from**. That second field is the load-bearing design choice: it gives the policy something concrete to compare the current state against. The pipeline per decision step:

1. **Retrieval** — the policy itself decides whether to query the bank `B` from the history window `h_t`, and emits the query. Top-k retrieval returns `E_t`.
2. **Contextual reconstruction** — the same policy sees `x_recon = T ⊕ h_t ⊕ ⋃(e_{t,i}, o_{t,i}^src)`, compares each source observation against the current context, and emits guidance `g_t` that retains, revises, or rejects the retrieved experience. If nothing applies it outputs `<EMPTY>`, and guidance falls back to a self-reasoning prompt `p_self`.
3. **Action** — the same policy acts on `(T, h_t, g̃_t)`.

Critically there is **one unified policy** for all three roles, and `g_t` has no ground-truth supervision. It is treated as a [[latent-reasoning|latent reasoning]] variable and trained end-to-end with [[grpo]] on sparse task reward (10 for success, 0 for failure) plus a 0.1-weighted format bonus that enforces the agentic protocol: exactly one `<think>`/`<action>` block per step, 1–5 `<retrieve_memory>` calls per episode, English-only output. The trajectory-level advantage is broadcast to every token, so thinking, reconstruction, and action share credit. Setup: Qwen2.5-7B-Instruct on the verl-agent framework, brief cold start (200 multi-turn CoT trajectories + 200 trajectory→memory summarization QA pairs per benchmark) purely for format alignment, BGE-M3 retriever, top-3 principles, history window w=3. The memory bank **starts empty** and is populated during RL from the policy's own trajectories.

## Key points

**Main results** (ALFWorld macro-averaged success rate over six categories; WebShop reports both task score and success rate):

| Method | ALFWorld Avg SR | WebShop Score | WebShop SR |
|---|---|---|---|
| GPT-4o | 49.2 | 31.8 | 23.7 |
| Gemini-2.5-Pro | 62.1 | 42.5 | 35.9 |
| Qwen2.5-7B-Instruct (base) | 14.5 | 26.4 | 7.8 |
| ReAct | 27.9 | 46.2 | 19.5 |
| Reflexion | 39.9 | 58.1 | 28.8 |
| Mem0 | 33.5 | 23.9 | 2.0 |
| ExpeL | 45.3 | 30.9 | 11.2 |
| MemP | 41.0 | 25.3 | 6.4 |
| SimpleMem | 27.9 | 33.2 | 8.6 |
| RLOO | 75.9 | 80.3 | 65.7 |
| GRPO | 76.4 | 79.3 | 66.1 |
| MemRL | 24.0 | 29.5 | 9.2 |
| Mem0 + GRPO | 52.0 | 58.1 | 37.5 |
| SimpleMem + GRPO | 54.5 | 67.8 | 46.9 |
| EvolveR (reproduced) | 70.1 | 84.1 | 72.6 |
| **MemHarness** | **85.2** | **87.4** | **75.6** |

The 7B model beats Gemini-2.5-Pro by **+23.1 pp on ALFWorld and +39.7 pp on WebShop SR**, and plain GRPO by **+8.8 / +9.5 pp**. Per-category ALFWorld: Pick 87.0, Look 78.6, Clean 97.0, Heat 87.5, Cool 71.4, Pick2 **90.0** — the Pick2 (two-object) column is the standout, where GRPO manages 64.7 and every prompt-based method is under 24.

**The negative-transfer claim is empirically the paper's strongest result.** Bolting memory onto RL *hurts*: Mem0+GRPO 52.0 and SimpleMem+GRPO 54.5 both land ~22–24 pp **below** plain GRPO (76.4) on ALFWorld. MemRL at 24.0 is worse than the untrained base model at some categories. So this is not a case of memory helping a little — the standard replay paradigm is destructive here, and that is what makes the reconstruction step worth its complexity.

**Ablations** (ALFWorld / WebShop Avg SR):

| Variant | Test-time memory | Reconstruction | ALFWorld | WebShop |
|---|---|---|---|---|
| Base model | ✗ | ✗ | 14.5 | 7.8 |
| Cold start only | ✗ | ✗ | 7.6 | 17.6 |
| RL only (GRPO) | ✗ | ✗ | 76.4 | 66.1 |
| RL + raw memory | ✓ | ✗ | 70.1 | 72.6 |
| **MemHarness** | ✓ | ✓ | **85.2** | **75.6** |
| − generic LLM reconstruction | ✓ | ✓ | 77.7 | 71.8 |
| − w/o reconstruction | ✓ | ✗ | 79.6 | 74.6 |
| − w/o memory | ✗ | ✗ | **83.0** | 73.6 |

Three things fall out. (a) Cold start alone is *worse than the base model* on ALFWorld (7.6 vs 14.5) — format alignment without RL destroys capability, so the RL is doing all the work. (b) Handing reconstruction to a generic Qwen2.5-7B-Instruct instead of the trained policy drops ALFWorld to 77.7: **zero-shot rewriting is not enough**, the adaptation has to be learned against environment dynamics. (c) **The most interesting row is `w/o memory` at 83.0** — disabling retrieval entirely at test time still beats RL-only by +6.6 pp. The reconstruction objective works as latent guidance *during training* and improves the agent's intrinsic reasoning; the memory itself only contributes the last +2.2 pp on ALFWorld (+2.0 on WebShop).

**Out-of-distribution ALFWorld** (unseen room layouts and object placements):

| Method | OOD Avg SR |
|---|---|
| **MemHarness** | **85.9** |
| − w/o reconstruction | 82.4 |
| − w/o memory | 83.0 |
| RL + raw memory | 76.3 |

Verbatim replay in unseen environments is *worse than having no memory at all* (82.4 < 83.0), which is negative transfer shown directly. Reconstruction recovers it and adds a margin.

**Does reconstruction actually compare states?** Two probes:

*(a) Source-state ablation* — RR = per-memory reject rate, SR = task success rate:

| Source state | ALFWorld RR | ALFWorld SR | WebShop RR | WebShop SR |
|---|---|---|---|---|
| Correct `o^src` | 8.7 | 85.2 | 56.0 | 75.6 |
| No `o^src` | 7.8 | 80.0 | 55.9 | 73.0 |
| Random `o^src` | 13.3 | 84.3 | 63.3 | 73.6 |

Removing the source state leaves the reject rate flat but costs 5.2 pp of success — the agent accepts misaligned guidance it can no longer detect. Swapping in a *random* source state spikes rejection (8.7→13.3, 56.0→63.3). Together: the policy is genuinely comparing historical against current state, not just reacting to a missing field.

*(b) Counterfactual probes* — 1,000 offline probes where a strong LLM minimally edits the current state so the retrieved memory no longer applies, scoring only the reconstruction output:

| Task | State | Unchanged | Adapted | Reject |
|---|---|---|---|---|
| ALFWorld | Match | 46.0 | 53.4 | 0.6 |
| ALFWorld | Edited | 37.3 | 56.3 | 6.4 |
| WebShop | Match | 0.0 | 27.9 | **72.1** |
| WebShop | Edited | 0.0 | 21.2 | **78.8** |

Both shift in the right direction under minimal edits. But note the raw levels: **on WebShop the policy rejects 72% of retrieved memories even when they match**, and never passes one through unchanged. The two environments elicit completely different policies — ALFWorld converges to sparse retrieval (2–3 per trajectory) with near-zero rejection; WebShop retrieves constantly and filters aggressively. The authors attribute this to observation format (concise state descriptions vs. long webpage content).

**Training dynamics** — trajectories containing an *accepted* reconstruction consistently succeed more often than those containing a *rejected* one, in both environments, which is the paper's evidence that the accept/reject decision is calibrated rather than arbitrary.

## Where it breaks

- **No limitations section.** The conclusion only offers "future work will explore scaling to larger models and open-ended environments."
- **[[test-time-memory|Test-time memory]] contributes surprisingly little.** The headline framing is about reconstructing retrieved experience, but the ablation shows most of the gain over RL-only is a *training-time* effect (76.4 → 83.0 with no memory at inference), and live memory adds ~2 pp. The honest reading is that the reconstruction objective is a good auxiliary training signal that happens to also support a memory system — closer to a curriculum result than a memory-architecture result.
- **Memory hurts on some categories.** ALFWorld Pick: `w/o memory` scores **100.0** and generic-LLM reconstruction 97.1, while full MemHarness gets 87.0. Plain GRPO also beats MemHarness on Pick (90.8). On Cool, MemHarness (71.4) trails both RLOO (71.9) and the generic-LLM variant (80.0). The averages hide real per-task regressions.
- **OOD score exceeds in-distribution** (85.9 vs 85.2), which is unusual enough to suggest the OOD split is not harder in the way the framing implies, or that the two evaluations are not directly comparable. The paper does not comment.
- **WebShop rejects the majority of what it retrieves.** A 72–79% reject rate means the retriever is mostly surfacing inapplicable material and the reconstruction step is functioning primarily as a filter. That is a working system, but it undercuts "harnessing past experience" as the description of what happens on WebShop.
- **Single backbone, single scale.** Everything is Qwen2.5-7B-Instruct. No evidence the reconstruction ability emerges at other scales, and no cost accounting for the extra generation pass per decision step.
- **Two benchmarks, both classic and saturating.** ALFWorld and WebShop, with the strongest baselines already at 70–85. No long-horizon multi-session or open-ended setting, despite the memory framing.
- **`RL + Raw Memory` is numerically identical to the reproduced EvolveR** across every ALFWorld category and WebShop, so that ablation row and that baseline are the same run — worth knowing when reading the two tables together.

## Related

- [[experiential-learning]] — the core subject: distilling trajectories into reusable natural-language experience, then adapting rather than replaying it.
- [[agentic-rl]] · [[grpo]] · [[reinforcement-learning]] — the whole pipeline is trained end-to-end with GRPO on sparse task reward; RLOO is the other RL baseline.
- [[outcome-based-reward]] — reward is 10/0 on task outcome plus a 0.1 format bonus; reconstruction gets no direct supervision.
- [[rl-memory-policy]] — closest existing framing: learning *when and how* to use memory as a policy decision rather than a fixed rule.
- [[memory-management]] · [[memory-construction]] — the bank is self-populated during training with semantic dedup, utility tracking, and low-utility pruning.
- [[procedural-memory]] — the entries are abstract strategies ("memory principles"), i.e. how-to knowledge rather than episodes.
- [[episodic-memory-llm]] — the contrast the paper draws with human recall: reconstruction over verbatim retrieval.
- [[reflection-mechanism]] · [[self-reflection]] · [[reflexion]] — Reflexion is a prompt-based baseline (39.9 ALFWorld); MemHarness's critique step is the learned analogue.
- [[react]] — the base agentic protocol and a baseline (27.9 ALFWorld).
- [[mem0]] — baseline both standalone (33.5) and combined with GRPO (52.0), where it drops *below* plain GRPO.
- [[memory-r1]] — adjacent line: RL-trained memory operations.
- [[embodied-agent]] · [[llm-agent]] · [[agent-benchmark]] · [[agent-evaluation]] — task setting and evaluation.
- [[alfworld]] · [[webshop]] — the two benchmarks (**no wiki pages yet**).
- [[dense-retrieval]] — BGE-M3 retriever over the experience bank, top-3.
- [[context-assembly]] · [[agentic-context-engineering]] — what the reconstruction step replaces: instead of assembling retrieved text into context, the policy rewrites it into state-specific guidance first.
- [[self-improving-agent]] · [[skill-library]] — the memory bank as a growing, pruned repository of learned strategies.
- [[chain-of-thought]] — `g_t` is treated as a latent reasoning variable, trained without ground truth like a CoT trace.

## References

- [arXiv:2607.28272](https://arxiv.org/abs/2607.28272)
- Shridhar et al., *ALFWorld*; Yao et al., *WebShop* — the two benchmarks.
- Yao et al., *ReAct*; Shinn et al., *Reflexion*; Zhao et al., *ExpeL* — prompt-based baselines.
- Ahmadian et al. 2024, *Back to Basics: Revisiting REINFORCE-style Optimization* (ACL) — RLOO.
- DeepSeek-R1 — the cold-start-then-RL recipe followed here.
- EvolveR — the reproduced memory-evolution baseline, identical to this paper's "RL + Raw Memory" ablation.
- Qwen2.5-7B-Instruct (policy backbone); BGE-M3 (retriever); verl-agent (training framework).
