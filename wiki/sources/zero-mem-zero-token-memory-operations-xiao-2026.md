---
title: "Zero-Mem: Zero-Token Memory Operations for LLM Agents"
type: source
created: 2026-08-17
updated: 2026-08-17
sources: []
arxiv_id: "2607.29377"
authors:
  - Yilin Xiao
  - Zhehan Zhu
  - Yujing Zhang
  - Jin Chen
  - Zijin Hong
  - Luyao Zhuang
  - Qinggang Zhang
  - Shengyuan Chen
  - Xiaocao Ouyang
  - Lingfei Ren
  - Xiao Huang
year: 2026
status: complete
importance: high
tags:
  - 2026
  - paper
---

# Zero-Mem: Zero-Token Memory Operations for LLM Agents

**Yilin Xiao, Zhehan Zhu, Yujing Zhang, Jin Chen, Zijin Hong, Luyao Zhuang, Qinggang Zhang, Shengyuan Chen, Xiaocao Ouyang, Lingfei Ren, Xiao Huang** — [arXiv:2607.29377](https://arxiv.org/abs/2607.29377), cs.CL, 31 Jul 2026. Preprint; no venue given. Code promised at `github.com/TheMoon0815/Zero-mem` after peer review. License CC BY-NC-SA 4.0.

## Summary

Most [[llm-agent]] memory systems spend LLM calls *on the memory itself* — summarizing, reflecting, extracting triples, evolving notes ([[mem0]], [[a-mem]], MemoryOS, Zep). That has two costs the paper wants to attack at once: a recurring token/latency bill, and a **provenance** problem — once a generated abstraction mediates retrieval, omitted details, merged subjects, and blurred temporal updates are no longer traceable to what was actually said. Recent work (SimpleMem, LightMem) *reduces* generative overhead; Zero-Mem asks whether it can be driven to exactly zero.

The paper defines **zero-token memory operations** as an operating regime, not just an implementation: memory construction, organization, routing, retrieval, evidence closure, pre-reader evidence calibration, and post-reader answer calibration all invoke no LLM and consume no LLM input or output tokens. Encoder computation and final-QA inference are accounted for **separately** — this is stated explicitly, and matters (see limitations).

Zero-Mem keeps the **original interaction traces as the source of record** and builds two non-generative indexes over them:

- **Entity–context graph** `G = (V_d ∪ V_e, E_de ∪ E_dd)` — context nodes and entity nodes, built by running a non-generative NER model (spaCy) over each unit. Entity–context edges carry a normalized co-occurrence weight `w(d_i,e) = c(e,d_i) / Σ_{e'∈E(d_i)} c(e',d_i)`; adjacency edges connect neighboring context units. The graph records **observed** co-occurrence, never inferred semantic triples.
- **Temporal hierarchy** `T(H) = U_turn ∪ U_window ∪ U_episode ∪ U_local` — turns (atomic utterances), windows (short-range context), episodes (adjacent windows grouped by semantic continuity and session boundaries), and local spans (immediate neighborhood of a candidate turn).

Both views resolve to the same provenance-bearing units, and both are indexed with BM25 lexical statistics plus BGE-M3 dense embeddings (for scoring only — no rewriting).

At query time a lightweight deterministic profile `φ(q) = {subject, keywords, answer-type, temporal-cues, boundary}` picks a route ∈ {relational, local}. **Both views always run**; routing only sets the fusion weight ρ (primary view gets ρ, secondary 1−ρ). The graph view seeds entity activations by cosine match, propagates them through query-relevant co-occurrence sentences, and runs **Personalized PageRank** `π_q = (1−γ)r_q + γP^⊤π_q`; the hierarchical view searches coarse-to-fine `episode → window → turn → local`. Rankings are min-max normalized per query and fused, evidence closure adds relational bridges and neighboring turns, then deterministic calibration filters provenance/boundary violations and ranks by subject, temporal, and answer-type compatibility. **The final-QA reader is the only LLM call**; post-reader calibration applies evidence-support/type/format checks with no second model.

## Key points

**LoCoMo, average over four question types** (F1 / BLEU-1; GAM is the strongest baseline):

| Reader | Method | Avg F1 | Avg BLEU-1 |
|---|---|---|---|
| GPT-4o-mini | Mem0 | 45.10 | 35.90 |
| GPT-4o-mini | A-Mem | 39.65 | 32.31 |
| GPT-4o-mini | LightMem | 38.44 | 34.36 |
| GPT-4o-mini | SimpleMem | 48.02 | 41.68 |
| GPT-4o-mini | CompassMem | 52.18 | 44.09 |
| GPT-4o-mini | GAM | 53.75 | 47.51 |
| GPT-4o-mini | **Zero-Mem** | **59.15** | **52.96** |
| Qwen2.5-14B | GAM | 52.70 | 46.55 |
| Qwen2.5-14B | **Zero-Mem** | **57.57** | **51.41** |

Gains over GAM: **+5.40 F1 / +5.45 BLEU-1** (GPT-4o-mini), **+4.87 / +4.86** (Qwen2.5-14B). With Qwen2.5-14B Zero-Mem ranks first on **every** question type and metric; with GPT-4o-mini it leads on single-hop (66.65 vs 57.75), temporal (61.97 vs 59.45), and open-domain (35.52 vs 33.30) but **loses multi-hop to GAM** (41.61 vs 42.29 F1; 32.92 vs 34.44 BLEU-1) — the single cell where it is not best.

**HotpotQA** (F1; MemAgent's curated memory-evaluation variant, distractors scaled to three context lengths):

| Method | 4o-mini 56K | 224K | 448K | Qwen 56K | 224K | 448K |
|---|---|---|---|---|---|---|
| LONG-LLM | 56.56 | 54.29 | 53.92 | 49.75 | 46.82 | 43.17 |
| RAG | 52.71 | 51.84 | 54.01 | 51.81 | 46.72 | 48.36 |
| Mem0 | 32.58 | 31.74 | 27.41 | 30.12 | 32.44 | 26.55 |
| GAM | 63.22 | 64.56 | 59.81 | 64.07 | 55.99 | 57.87 |
| **Zero-Mem** | **72.07** | **66.43** | **65.04** | **68.58** | **65.47** | **61.02** |

Best in all six cells; **+5.52 F1 averaged** over the strongest baseline. Note how badly the *generative* memory systems (Mem0 27–33, MemoryOS 23–30, A-Mem 23–34) do here — they fall far below plain RAG and even plain long-context, which the paper reads as abstraction destroying the distributed supporting evidence.

**Efficiency** (unified config: 4 concurrent threads, GPT-4o-mini backbone, identical hardware; overhead is memory-operation only, outside the shared final-QA stage):

| Method | F1 | BLEU-1 | Tokens | Tokens/query | Time (s) | Time/query (s) |
|---|---|---|---|---|---|---|
| SimpleMem | 48.02 | 41.68 | 14,096,246 | 9,153.41 | 8,365.38 | 5.43 |
| LightMem | 38.44 | 34.36 | 877,086 | 569.54 | 788.76 | 0.51 |
| GAM | 53.75 | 47.51 | 28,570,674 | 18,552.39 | 9,237.25 | 6.00 |
| **Zero-Mem** | **59.15** | **52.96** | **0** | **0** | **334.77** | **0.22** |
| Relative | +10.0% | +11.5% | −100% | −100% | **−57.6%** | −57.6% |

The **57.6%** headline is measured against **LightMem** (the fastest baseline, 788.76 s), while the +10.0%/+11.5% quality gains are against **GAM** (the best-quality baseline) — two different reference rows in the same table. Zero-Mem is simultaneously the most accurate and the cheapest, which is the paper's real claim: removing generative memory calls does not push the cost into a slower non-generative pipeline.

**Ablation** (HotpotQA 56K, GPT-4o-mini; F1 / BLEU-1):

| Variant | F1 | BLEU-1 |
|---|---|---|
| Full | 72.07 | 69.66 |
| Graph view only | 62.50 | 59.90 |
| Hierarchical view only | 54.88 | 51.40 |
| − evidence closure | 67.90 | 65.43 |
| − evidence calibration | 70.13 | 66.45 |

Both single-view variants land far below the full model, so the two structures are genuinely complementary rather than redundant. Graph-only beats hierarchy-only by 7.6 F1 here, which the authors attribute to HotpotQA's cross-document relational emphasis — the ranking would plausibly invert on a locality-heavy benchmark, but that ablation is not run.

**Retrieval budget** (LoCoMo, GPT-4o-mini): top-1 gives 52.59 F1 / 46.79 BLEU-1, top-5 gives 59.15 / 52.96, and top-10 is the best overall — but top-5 trails top-10 by only **0.65 F1 and 0.83 BLEU-1** at half the primary candidates. Larger budgets only fluctuate. Top-5 is used in the main tables to match every baseline's setting.

**Hyperparameters**: damping factor γ = 0.6 and dual-view routing coefficient ρ = 0.6, both fixed. NVIDIA RTX 4090s. Readers are GPT-4o-mini and Qwen2.5-14B-Instruct, identical across all methods with equivalent context budgets.

## Where it breaks

- **"Zero-token" is an accounting boundary, not zero cost.** The paper is upfront: encoder inference, memory organization, retrieval, and calibration still incur processing. spaCy NER over the whole history plus BGE-M3 embedding is real GPU work. The claim survives because the wall-clock number (334.77 s total) *includes* that work and still wins — but a reader who takes "zero" as free is misreading it.
- **No explicit limitations section.** The paper does not state its own failure modes; everything below is inferred from what is and isn't measured.
- **The deterministic machinery is hand-built.** Routing keys off "question form, temporal or aggregation requirements, and the availability of subject anchors"; calibration applies type/format/list-pruning rules. No robustness study on queries whose surface form misleads the router, and no sensitivity sweep on γ or ρ — both are simply set to 0.6.
- **Only two benchmarks, both pure QA.** Despite the "LLM Agents" framing there are no tool-use, action, or trajectory evaluations, even though the preliminaries define trace units to include actions and tool observations. [[longmemeval]] is not run. HotpotQA is a Wikipedia multi-hop dataset repurposed as a memory eval, not a real interaction history.
- **Losing multi-hop on LoCoMo/GPT-4o-mini** to GAM is the visible edge of the approach: with no generative abstraction, evidence that needs *synthesis* across sessions has to be reachable by graph propagation alone.
- **Cost is shifted, not removed, for indexing.** Building the entity–context graph and four-granularity hierarchy over a growing history is an ongoing non-LLM cost; no incremental-update or scaling analysis is given.

## Related

- [[memory-management]] — the paper's central target: the whole memory lifecycle re-implemented without generation.
- [[memory-construction]] — construction here is NER + graph assembly + hierarchical chunking, with no LLM in the loop.
- [[graph-memory]] — the entity–context graph, restricted to *observed* co-occurrence rather than inferred relations.
- [[tiered-memory]] — the turn/window/episode/local hierarchy is a granularity tier stack, contrasted with MemoryOS's short/mid/long-term OS-style paging.
- [[long-term-memory]] · [[episodic-memory-llm]] — episodes group adjacent windows into coherent event regions; provenance is preserved to the raw trace.
- [[locomo]] — primary long-term conversational memory benchmark; four question types.
- [[mem0]] · [[a-mem]] · [[memorybank]] — generative-memory baselines; Mem0 and A-Mem both collapse on HotpotQA (27–34 F1) relative to plain RAG.
- [[memgpt]] · [[virtual-context]] — the OS-paging lineage that MemoryOS extends.
- [[retrieval-augmented-generation]] · [[dense-retrieval]] · [[bm25]] — the RAG baseline and Zero-Mem's own lexical/dense access signals (BM25 + BGE-M3).
- [[hipporag-2]] — HippoRAG (the 2024 v1 paper) is a baseline here; both systems use Personalized PageRank over an entity graph, but Zero-Mem drops the LLM-based triple extraction.
- [[knowledge-graph]] — explicitly *not* what the graph view builds; no semantic triples are generated.
- [[long-context-llm]] — the LONG-LLM baseline and the 448K-token HotpotQA setting.
- [[prompt-compression]] · [[context-assembly]] — the alternative cost-reduction lever; Zero-Mem instead removes the generation step entirely.
- [[agentic-context-engineering]] — same problem space, opposite bet: evolve a written context vs. never write one.
- [[reflection-mechanism]] · [[memory-evolution]] · [[zettelkasten-memory]] — the generative operations Zero-Mem removes ([[a-mem]]'s note linking, reflection-based consolidation).
- [[offline-consolidation]] — LightMem's separation of offline consolidation from online retrieval; Zero-Mem's answer is to have no consolidation stage at all.
- [[llm-agent]] · [[agent-evaluation]] — framing and evaluation context.
- [[open-domain-qa]] — one of the four LoCoMo question types.
- [[reflexion]] · [[react]] · [[generative-agents]] — cited as the reflective/acting agent lineage motivating generative memory.

## References

- [arXiv:2607.29377](https://arxiv.org/abs/2607.29377)
- Maharana et al. 2024, *Evaluating Very Long-Term Conversational Memory of LLM Agents* (ACL) — LoCoMo.
- Yang et al. 2018, *HotpotQA* (EMNLP); Yu et al. 2026, *MemAgent* (ICLR) — the curated memory-evaluation variant used here.
- Yan et al. 2025, *General Agentic Memory via Deep Research* ([arXiv:2511.18423](https://arxiv.org/abs/2511.18423)) — GAM, the strongest baseline.
- Fang et al. 2026, *LightMem* (ICLR); Liu et al. 2026, *SimpleMem* (ICML) — the efficiency-oriented baselines.
- Kang et al. 2025, *Memory OS of AI Agent* (EMNLP); Rasmussen et al. 2025, *Zep* ([arXiv:2501.13956](https://arxiv.org/abs/2501.13956)).
