---
title: "Metis: Memory Foundation Model"
type: source
created: 2026-08-17
updated: 2026-08-17
sources: []
arxiv_id: "2607.26760"
authors:
  - Zeyu Zhang
  - Ziliang Guo
  - Yihang Sun
  - Xichong Zhang
  - Xixuan Hao
  - Zehao Lin
  - Yang Zhang
  - Xiaoyan Zhao
  - Tong Shen
  - Bo Tang
  - Zhi-Qin John Xu
  - Junchi Yan
  - Haofen Wang
  - Xu Chen
  - Feiyu Xiong
  - Zhiyu Li
  - Tat-Seng Chua
year: 2026
status: complete
importance: high
tags:
  - 2026
  - paper
---

# Metis: Memory Foundation Model

**Zeyu Zhang, Ziliang Guo, Yihang Sun, Xichong Zhang, Xixuan Hao, Zehao Lin, Yang Zhang, Xiaoyan Zhao, Tong Shen, Bo Tang, Zhi-Qin John Xu, Junchi Yan, Haofen Wang, Xu Chen, Feiyu Xiong, Zhiyu Li, Tat-Seng Chua** — [arXiv:2607.26760](https://arxiv.org/abs/2607.26760), 29 Jul 2026. Preprint; no venue given. Project and checkpoints released at `github.com/MemTensor/Metis`.

## Summary

The paper's thesis is an analogy: agents internalized reasoning (large reasoning models absorbed [[chain-of-thought]] into the forward pass) and modality (multimodal foundation models absorbed vision), but **memory is still an external module bolted onto a frozen backbone**. The authors argue that costs three things — (1) memory and backbone optimize different targets, so retrieval may not surface what the backbone actually needs; (2) end-to-end optimization is impossible because gradients cannot pass through discrete store/retrieve operations (RL patches this at high cost); (3) explicit out-of-model storage operations add online latency.

They define a **memory foundation model** along two axes:

- **Native memory state** — the model is *stateful across inferences*. Part of θ is dynamic: `y_{t,k} ~ P(y | X_t, Y_{t,<k}; θ_t)`, with no context `C_t` in the conditioning at all. Notably the paper adopts a **strict definition of memory**: only information acquired *during online interaction* counts; anything available before the interaction is "knowledge", not memory.
- **Native memory procedure** — remembering, forgetting, and updating happen *autonomously inside the forward computation*, driven by the semantic intent of the instruction ("Bob moved from London to Boston" implies an update), not by rule-based insert/delete/modify dispatch.

**Metis** is the prototype. Inspired by Fast Weight Programming, it inserts **Metis blocks** into Transformer blocks, each pairing:

- a **local memory block** holding a dense memory network `M_t^(l) ∈ R^{d_k × d_v}` plus a query-key normalization vector `S_t^(l) ∈ R^{d_k}`, both initialized to zero;
- a **hyper memory block** of *static* mid-trained parameters (importance vector `w̃_agg`, memory projections `W̃_K, W̃_V, W̃_Q`) that computes the update.

Storage scores each token with the learnable importance vector, keeps the top-ρ nucleus of that distribution (adaptive aggregation), projects the survivors, and writes `M_{t+1} = λM_t + ((1−λ)/L') · (K̃^⊤/√d_k) Ṽ`. In the shipped model the linear write is replaced by a **Gated Delta Network-based update (GDU)**. Utilization is a **memory attention** branch `Ã_t = diag(Q̃_t S_t)^{-1} Q̃_t M_t`, fused into the main attention as `A_t = γ·Softmax(...)V + (1−γ)·Norm(Ã_t)` — an additive residual alongside full attention, explicitly *not* a replacement for it.

Two properties make this "gradient-free at inference": all learned weights (backbone and hyper blocks) are frozen; the memory update is a single forward pass. The backbone is frozen during mid-training too — only the memory parameters train, initialized from the corresponding backbone K/V projections.

## Key points

Everything below is the **no-context** setting: the model gets nothing in its prompt but the question, so any correct answer must come from the memory state. Backbones are Qwen3.5 at 4B/9B/27B; judge is gpt-4.1-mini, median of three repeats.

**Memory operation tasks** (MemOps and the authors' own test set; micro-average):

| Setting | Method | MemOps (Gold) Avg | Metis Test Set Avg |
|---|---|---|---|
| *Full context* | *Qwen3.5-27B* | *87.90* | *78.87* |
| *Partial context (RAG)* | *Qwen3.5-27B* | *28.01* | *65.40* |
| No context | Qwen3.5-4B / 9B / 27B | 1.65 / 1.88 / 1.69 | 16.96 / 18.64 / 16.87 |
| No context | Temp-LoRA-4B / 9B / 27B | 8.85 / 13.51 / 9.70 | 19.34 / 19.51 / 23.86 |
| No context | δ-Mem | 4.38 | 15.03 |
| No context | **Metis-4B / 9B / 27B** | **17.84 / 19.63 / 24.76** | **56.72 / 57.92 / 73.77** |

**Memory-based QA** (LoCoMo gold-session; NextMem contextual generation = SQuAD + HotpotQA + LoCoMo + LongMemEval):

| Setting | Method | LoCoMo (Gold) Avg | NextMem Avg |
|---|---|---|---|
| *Full context* | *Qwen3.5-27B* | *65.03* | *78.80* |
| *Partial context (RAG)* | *Qwen3.5-27B* | *23.17* | *—* |
| No context | Qwen3.5-4B / 9B / 27B | 0.18 / 0.07 / 0.07 | 11.86 / 15.93 / 17.75 |
| No context | Temp-LoRA-4B / 9B / 27B | 9.99 / 11.72 / 4.24 | 24.20 / 28.12 / 30.97 |
| No context | δ-Mem | 10.79 | 20.42 |
| No context | **Metis-4B / 9B / 27B** | **16.31 / 16.81 / 26.74** | **41.69 / 43.39 / 50.82** |

The bare Qwen3.5 baselines score ~0 on LoCoMo (Gold) without context, which is the control that matters: the answers genuinely are not recoverable from pretrained knowledge, so Metis's 26.74 is real stored information. Metis-27B NextMem sub-scores: SQuAD 43.42, HotpotQA 66.54, LongMemEval 39.71, LoCoMo 60.41 — against full-context Qwen3.5-27B at 91.80 / 89.18 / 48.43 / 64.47. **The LoCoMo and LongMemEval subsets are where compression costs least** (60.41 vs 64.47; 39.71 vs 48.43), while SQuAD loses more than half. Scaling is lumpy: 4B→9B is nearly flat, 9B→27B jumps.

**Out-of-distribution transfer** (benchmarks not used to build training data, no-context):

| Method | ATM-Bench (Gold) Avg | MemDaily (Gold) Avg |
|---|---|---|
| δ-Mem | 2.27 | 39.84 |
| Temp-LoRA-4B / 9B / 27B | 2.57 / 2.86 / 2.57 | 44.92 / 51.42 / **59.45** |
| Metis-4B / 9B / 27B | 10.22 / 16.49 / **18.56** | 52.54 / 47.29 / 59.04 |

ATM-Bench is a clean win, and it holds on the *deterministically scored* number questions (Metis-27B 31.39 vs 0.00 for every Temp-LoRA size), so it is not a judge artifact. **MemDaily is not a win**: Temp-LoRA-27B edges Metis-27B (59.45 vs 59.04), and Metis-9B (47.29) is *worse than Metis-4B* (52.54). The paper says so plainly ("more mixed").

**Ablations on Metis-4B** (ΔAvg. vs the full model, overall):

| Variant | Overall Avg | ΔAvg |
|---|---|---|
| Full model | 33.14 | — |
| w/o adaptive aggregation (last-token instead) | 12.93 | **−60.98%** |
| w/o query-key normalization | 23.72 | −28.44% |
| w/o auxiliary data (Multi-fact + Memory Pollution) | 26.74 | −19.31% |
| w/o optimizable memory query | 29.09 | −12.23% |
| w/o Multi-fact Scenario data only | 29.68 | −10.43% |
| w/o GDU (linear update instead) | 32.95 | −0.58% |

**Adaptive aggregation is the whole ballgame** — writing the last-token hidden state instead of the learned top-ρ nucleus costs 61% of performance. Conversely the GDU, the component the paper's architecture section builds up to, is worth almost nothing on average at 4B: linear update is actually *better* on both memory-operation benchmarks (+3.32%) and only loses on LoCoMo (Gold).

**General-capability regression** (Metis-4B minus Qwen3.5-4B):

| Benchmark | Initial stage (empty memory) | Active stage (irrelevant info stored) |
|---|---|---|
| MMLU-Pro | −0.80 | −5.10 |
| IFEval (strict) | +0.55 | **−22.18** |
| GSM8K | −1.06 | −5.61 |
| MMMLU | −0.80 | −3.30 |

With an empty memory state the architecture is essentially free. Once **task-irrelevant** content is stored, strict instruction-following falls off a cliff (−22.18). This is arguably the paper's most important negative result and it is reported without spin.

**Low-rank compressibility** — SVD the memory state (full dim 1024) and keep rank k. Overall recovery: k=1 → 43.5%, k=4 → 68.9%, k=16 → 94.2%, **k=64 → 99.9%**, k=128/256 → flat. Useful information lives in a ~64-dimensional subspace, so the state is heavily redundant. Sensitivity varies by task: MemOps (Gold) recovers fully at k=16, while the Metis test set keeps improving to k=64.

**Training** — 8×H100, backbone frozen, memory parameters initialized from the backbone's K/V projections. AdamW, lr 2×10⁻⁴, constant after 200 warmup steps, weight decay 0.01, grad clip 1.0, BF16, seed 42. Metis-4B: 14,000 steps ≈ 1 epoch; Metis-27B: same 14,000 steps ≈ 0.4 epochs; Metis-9B: 8,000 steps ≈ 0.57 epochs, chosen by early stopping.

## Where it breaks

The paper is unusually candid; most of this is self-reported.

- **Metis is nowhere near full context.** On MemOps (Gold) the best Metis (24.76) is under a third of full-context Qwen3.5-27B (87.90); on LoCoMo (Gold) it is 26.74 vs 65.03. Every "best" claim is *within the no-context group*. The authors say outright that native memory "cannot be viewed as a complete replacement for external memory."
- **Fixed-size state → long-horizon decay.** The memory-capacity study (20 synthetic personas × 40 atomic persona domains) probes two axes. *Step-level*: as a single update grows, accuracy falls fast, and past a few hundred words all three probe positions (first / middle / last fact) are low. *Trajectory-level*: with 40 statements written in groups of 5, first-fact accuracy declines almost monotonically **and middle/recent facts also destabilize** — new writes interfere across the whole state rather than only evicting the oldest.
- **Semantic confusion in the shared latent state.** Named as a core limitation: similar facts blend.
- **Forgetting is the hardest operation and generalizes worst.** On external MemOps (Gold), Metis-27B scores 10.91 on Forget vs 28.27 Remember / 31.02 Update — and this despite scoring 77.50 on Forget on the authors' *own* test set, a gap that reads as overfitting to in-distribution forget phrasing. The case study shows the mechanism half-working: after a forget instruction the state is correctly updated, but the model's *immediate reply* still repeats the deleted fact.
- **Stored noise degrades unrelated tasks** (IFEval −22.18 at active stage). Whatever occupies the memory state is always in the forward pass; there is no gating to ignore it.
- **No measured efficiency numbers.** Section 3.6 argues latency analytically — the three branches (original attention, memory attention, memory storage) have no sequential dependency, so `T_parallel = max(T_orig, T_util, T_store) + T_fuse` — and notes utilization cost depends on state size rather than history length. But **no wall-clock, throughput, or memory-footprint measurement is reported anywhere**, which is a conspicuous gap for a paper whose motivation includes inference latency.
- **Training data is synthesized from the same public benchmarks used for evaluation.** Seed entities are extracted from LoCoMo and LongMemEval; the authors state exact QA behaviors are not leaked, and the OOD section (ATM-Bench, MemDaily) is the response to this. Take the in-distribution LoCoMo numbers accordingly.

## Related

- [[memory-management]] — the paper's target: replace the whole external memory lifecycle with in-architecture state.
- [[long-term-memory]] · [[memory-construction]] — construction here is a forward-pass write into `M_t`, not a text record.
- [[neural-long-term-memory]] · [[titans]] — the closest architectural relatives; Titans is cited in the fast-weight/TTT lineage, and Metis's contrast is that it targets *multi-step interaction* memory rather than long-context modeling within one sequence.
- [[test-time-memory]] — Temp-LoRA (temporary LoRA trained on prior chunks at inference) is the TTT-style baseline; Metis beats it but the paper's distinction is that TTT lacks *instruction-driven* semantic operations.
- [[lora]] — the mechanism behind the Temp-LoRA baseline.
- [[linear-attention]] · [[state-space-model]] — the fast-weight family the memory attention derives from (linear transformers as fast weight programmers; RetNet, RWKV, S4, Mamba, Gated Delta Network).
- [[attention-mechanism]] · [[transformer-architecture]] — memory attention is fused as a weighted residual into standard causal attention, not a substitute.
- [[kv-cache]] — the contrast: a fixed-size `M_t ∈ R^{d_k × d_v}` instead of a cache growing with history.
- [[locomo]] · [[longmemeval]] — both appear, and unusually both as *training-seed* and *evaluation* sources.
- [[squad]] · [[mmlu]] — SQuAD is a NextMem subset; MMLU-Pro/MMMLU carry the general-capability regression test.
- [[retrieval-augmented-generation]] · [[dense-retrieval]] — the partial-context baseline (top-5 by cosine similarity) and the paradigm being argued against.
- [[memorybank]] — cited as the representative textual-memory system with hierarchical storage and dual-tower retrieval.
- [[in-context-learning]] — what external textual memory ultimately relies on, and what native memory removes.
- [[knowledge-distillation]] · [[fine-tuning]] — contrast: mid-training installs the *capacity* to memorize, not the memories themselves.
- [[llm-agent]] · [[agent-evaluation]] — framing; memory as a native agent capability alongside reasoning.
- [[chain-of-thought]] — the explicit analogy: as CoT was internalized into large reasoning models, memory should be internalized into memory foundation models.
- [[quantization]] — adjacent in spirit to the low-rank SVD result (99.9% recovery at rank 64 of 1024): the state is highly compressible post hoc.

## References

- [arXiv:2607.26760](https://arxiv.org/abs/2607.26760) · project: `github.com/MemTensor/Metis`
- Ba et al. 2016, *Using Fast Weights to Attend to the Recent Past* — the FWP foundation.
- Yang et al. 2025, *Gated Delta Networks* — the GDU update rule.
- Maharana et al. 2024, *Evaluating Very Long-Term Conversational Memory of LLM Agents* (ACL) — LoCoMo.
- Wu et al. 2025 (LongMemEval); Rajpurkar et al. (SQuAD); Yang et al. 2018 (HotpotQA) — NextMem components.
- Behrouz et al., *Titans*; Sun et al., *TTT layer*; Gu & Dao, *Mamba* — the fast-weight / test-time-training lineage surveyed in §7.2.
- Meng et al., *ROME* — knowledge editing as a parametric-memory precedent.
