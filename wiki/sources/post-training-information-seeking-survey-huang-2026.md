---
title: "From Search to Decision: How Post-Training Shapes Information-Seeking Capabilities in Large Language Models"
type: source
created: 2026-11-19
updated: 2026-11-19
sources:
  - post-training-information-seeking-survey-huang-2026.md
arxiv_id: "2611.14149"
year: 2026
authors:
  - Haohang Huang
  - Xuan Lu
  - Mingjin Kuai
  - Yuxuan Zhang
  - Wei Ji
  - Haiyang Zhang
tags:
  - 2026
  - survey
status: complete
importance: medium
---

# From Search to Decision: How Post-Training Shapes Information-Seeking Capabilities in LLMs

**Haohang Huang, Haiyang Zhang** (Xi'an Jiaotong-Liverpool University), **Xuan Lu** (Shanghai Jiao Tong University), **Mingjin Kuai, Wei Ji** (Nanjing University), **Yuxuan Zhang** ([[university-of-british-columbia|University of British Columbia]]) — [arXiv:2611.14149](https://arxiv.org/abs/2611.14149), cs.IR, 10 Nov 2026. Short survey (9 pages + 2 appendix tables).

## Summary

Retrieval is no longer a fixed preprocessing step. A model now decides whether to search, what to ask, where to look, what the evidence supports, and when to stop. This survey organizes post-training work on those decisions around **six capabilities** rather than around architectures or optimizers. For each, it asks three questions: what to learn (Sec. 3), how it is learned (Sec. 4: signals, optimizers, update locus, credit assignment), and what was actually learned (Sec. 5: measurement, attribution, generalization).

The task is framed as a sequential policy: at step t, an action is drawn from π(· | q, h_t) over the history of actions and observations, and a final answer is generated from the full history. This covers single-step [[retrieval-augmented-generation|RAG]] and iterative [[agentic-search]] alike.

The paper reports no new experiments and no aggregate numbers. Its value is the taxonomy and an evaluation checklist.

## Key points

**Six capabilities** (Fig. 1 draws them as a loop, with exploration as cross-cutting):

| Capability | Decision it governs | Example methods |
|---|---|---|
| Epistemic awareness / retrieval utility | Is acquiring information worth its cost? | CoBRA (tool/no-tool counterfactual margins), s3, Calibrate-then-Act |
| Information-need formulation | What query expresses the gap? | StepSearch, QREF (refiner only) |
| Information-space exploration | Where to look next? | MARAG-R1 (multi-retriever coverage), diversity-driven PO |
| Evidence evaluation | Support, sufficiency, source reliability | [[corrective-rag|CRAG]], RLSeek, S2G-RAG, AuthorityBench |
| Evidence integration / belief revision | How evidence updates conclusions | RAFT, GuarantRAG (DPO), CARE, [[memory-r1|Memory-R1]] |
| Adaptive search control | Continue, reformulate, verify, switch, or stop | [[self-rag|Self-RAG]], [[search-r1|Search-R1]], AutoSearch, AnySearch |

**How it is learned** (Table 2 compares 9 representative methods):
- Signals range from demonstrations ([[supervised-fine-tuning|SFT]]) to terminal [[outcome-based-reward|outcome rewards]] (Search-R1) to behavior-specific rewards (CoBRA margins, StepSearch information gain plus redundancy penalty) to preferences (GuarantRAG via [[dpo|DPO]]).
- Optimizers: SFT, DPO, [[ppo|PPO]], [[grpo|GRPO]], RLOO, and hybrids (SFT warm start then RL).
- Update locus: a modular component (QREF trains only a query refiner; RLSeek only a verifier) vs. the whole policy (Search-R1). MemRL updates external memory utilities with frozen weights; Agentic-R co-optimizes retriever and agent.
- Credit assignment for long trajectories: intermediate answers (AutoSearch), retrospective critics (CriticSearch), judged round contributions (CW-GRPO), gold-conditioned state-value differences from a frozen reference model (TRACE), teacher–student disagreement weighting inside GRPO (SSPO).

**What was actually learned:**
- Routing and budget control are measured fairly directly (action-level). Evidence integration is mostly measured downstream (answer accuracy), which says less about whether evidence causally drove the decision.
- A controlled comparison (Zhao et al. 2026a) finds corpus coverage strongly affects results, and plain outcome rewards stay competitive with more elaborate credit assignment.
- Transfer evidence exists but is narrow: unseen budgets (AnySearch), other agent backbones (QREF), closed-to-open-web (TRACE). Retriever-aware query learning loses effectiveness on a different retriever (Yuan et al. 2026a).

**Evaluation proposal.** Report target behavior, intervention and updated component, behavioral measures, controlled factors, generalization setting, and untested claims. Example: hold task, trajectory prefix, and budget fixed; vary only the latest retrieval observation; check whether the next action changes appropriately.

**Open problems:** (1) predicting the value of evidence *before* retrieving it; (2) learning to coordinate the six decisions jointly, tested against outcome-only baselines; (3) adapting search control to unfamiliar retrieval environments.

## Limits

- No quantitative synthesis. The paper reports no effect sizes, so it cannot rank methods.
- Coverage is selective; most cited work is from 2025–2026. Capability assignments in Table 2 are "primary targets", not exclusive.
- The evaluation protocols in Appendix B are proposals, not results.

## Concepts & entities

- [[agentic-search]] — the setting the six capabilities decompose.
- [[retrieval-augmented-generation]] — single-step special case of the formulation.
- [[search-r1]] — the reference end-to-end outcome-rewarded search policy.
- [[self-rag]], [[corrective-rag]] — early adaptive-retrieval and evidence-evaluation methods.
- [[memory-r1]] — cross-turn persistence via RL-trained memory management.
- [[outcome-based-reward]], [[process-reward-model]] — the sparse-vs-intermediate credit trade-off in Sec. 4.4.
- [[grpo]], [[ppo]], [[dpo]], [[supervised-fine-tuning]], [[agentic-rl]] — the optimizers surveyed.
- [[hallucination-detection]] — RLSeek's out-of-domain evaluation target.
- [[university-of-british-columbia]] — co-author affiliation.

## References

- [arXiv:2611.14149](https://arxiv.org/abs/2611.14149)
- [[search-r1-jin-2025]], [[self-rag-asai-2023]], [[crag-yan-2024]], [[memory-r1-yan-2025]] — surveyed methods with existing source pages.
