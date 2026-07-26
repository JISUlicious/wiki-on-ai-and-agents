---
title: "Always-On Agents: A Survey of Persistent Memory, State, and Governance in LLM Agents"
type: source
created: 2026-07-22
updated: 2026-07-22
sources:
  - always-on-agents-survey-ding-2026.md
arxiv_id: "2606.30306"
year: 2026
authors:
  - Tianyu Ding
  - Aditya Nannapaneni
  - Bingfan Liu
  - Ling Zhang
tags:
  - 2026
  - survey
status: complete
importance: high
---

# Always-On Agents: A Survey of Persistent Memory, State, and Governance in LLM Agents

**Tianyu Ding, Aditya Nannapaneni, Bingfan Liu, Ling Zhang** (affiliations not stated in the preprint) — [arXiv:2606.30306](https://arxiv.org/abs/2606.30306), cs.MA, 29 Jun 2026. 136 pages.

## Summary

A large survey (136 pages, 435 coded works) arguing that "always-on" LLM agents should be analyzed as **persistent-state systems**, not merely memory-augmented ones. The defining property is *not* continuous execution but **persistence**: an agent is always-on when its behavior at a given moment depends on durable state accumulated before that moment, beyond the current prompt. The survey's central move is to widen the unit of analysis from retrievable memory to everything durable that can *authorize* a future action — task ledgers of open commitments, permissions and consent, tool and credential state, provenance and audit trails, shared/social state across users and agents, trigger state governing proactive action, and externally committed effects. Its slogan for the risk: "state becomes authoritative" — a cached credential grants access after the user intended it to lapse; a months-old summary overrides fresh evidence because it retrieves more strongly.

The literature is read through **six diagnostic axes** per state item (authority, scope, mutability, provenance, recoverability, actionability) and a **ten-stage lifecycle** (observe, write, validate, organize, retrieve, act, update, forget, audit, rollback). Coding all 435 works against this grid exposes a sharp asymmetry: the field is heavily invested in the *forward arc* (accumulate and recall) and thin on the *return arc* (govern, relinquish, repair). Retrieve appears in 269/435 works and write in 200, but audit in 88, forget in 66, and **rollback in only 27** — and no work in the corpus reports recovery success or cost after corruption. Authority is the rarest axis at 72/435.

Because existing benchmarks score answer quality, they cannot detect this gap, so the survey proposes the **Always-On Evaluation Protocol (AOEP-v0)** — a pilot evaluation contract that scores state-mutation and recovery obligations instead. A pilot harness ran seven systems through nine fault patterns and found that recency, full-context, and dense-retrieval memory stores score *identically* (7/15 obligations), failing the same governance subset — evidence that the missing thing is a structured governance envelope, not better retrieval. The authors are careful to frame the counts as a "scoped map rather than an exhaustive census."

## Key points

**Taxonomy of persistent state (Sec. 3)** — two tiers:

| Tier | Types |
|---|---|
| Memory types (classical core) | episodic traces, semantic facts, preferences, procedural skills |
| Control-relevant state (beyond memory) | workflow / task-ledger state, policy & permission state, provenance & audit state, shared & social memory, trigger state, tool & credential state, external commitments & side effects |

- **Six state axes:** authority (who permits this state to influence an action), scope, mutability, provenance, recoverability, actionability.
- **Ten lifecycle stages:** forward arc = observe, write, validate, organize, retrieve, act; return arc = update, forget, audit, rollback.
- **Corpus skew (N = 435, coded):** retrieve 269, write 200 vs. audit 88, forget 66, **rollback 27**. Axis coverage: mutability 160 (most common), provenance 153, recoverability 112, **authority 72 (rarest)**.
- **Corpus by part:** P1 Foundations 27, P2 Substrates 117, P3 Lifecycle 33, P4 Continual adaptation 52, P5 Governance 53, P6 Multi-agent 23, P7 Evaluation 25, P8 Failure modes 25, P9 Applications 80.
- **Coding reliability:** pooled inter-coder agreement 0.82 on lifecycle stages, 0.74 on state axes over a blind 236-work sample.
- **Governance-targeted search rounds** grew the corpus more than fourfold yet only lifted the governance fraction from ~1/6 to ~1/3; restricting to work submitted through 2024 makes the skew *worse*, so it is not an artifact of immature 2026 preprints.
- **AOEP-v0 pilot** (nine fault patterns, frozen Qwen2.5-7B reader, greedy decoding), obligations passed / negative-invariants / total:

| System | Obligation | Neg.-invariant | All |
|---|---|---|---|
| Governed reducer (upper bound) | 15/15 | 41/41 | 92/92 |
| No-memory (floor) | 0/15 | 41/41 | 77/92 |
| Naive append | 7/15 | 40/41 | 83/92 |
| Full context | 7/15 | 40/41 | 83/92 |
| Vector-RAG (MiniLM) | 7/15 | 40/41 | 83/92 |
| Mem0-style (extracted facts) | 4/15 | 38/41 | 77/92 |
| Mem0 (actual mem0ai) | 3/15 | 36/41 | 73/92 |

- The three raw-storage systems score **identically** because they pass obligations reducible to semantic recall and fail those needing maintained governance state (current permission epoch after revocation, blocking a stale-permission task, surfacing a conflict, logging a rollback). Extracted-fact stores score *below* raw storage because extraction flattens the structured envelope (permission epoch, deletion link, trust tier) into free text; the local mem0ai setup leaked a deleted billing address and an untrusted exfiltration address on two episodes.
- **Reader robustness:** full-context system across Qwen2.5-3B (7/15), Qwen2.5-7B (7/15), Mistral-7B-Instruct-v0.3 (4/15), Llama-3.1-8B (7/15) — the larger reader does no better. Under sampled decoding (T=0.7, top-p=0.95, 5 seeds) obligation pass held in a 6–8/15 band (mean 6.6, sd 0.8).
- **"Baseline-beats-memory" finding:** surveyed controlled studies find purpose-built memory systems do not reliably beat naive in-context use of recent history, and continuously LLM-consolidated textual memory has a utility curve that rises then **degrades below the no-memory baseline** as consolidation accumulates. Conclusion: "an ungoverned memory system can be strictly worse than having no memory at all once the horizon is long enough."
- **Persistence-induced failure modes:** write poisoning/injection, retrieval distraction and tool-drift, stale commitment and belief drift, deletion failure and the rollback gap, cross-agent propagation, and rollback itself as a fresh attack surface.

## Concepts & entities

- [[memory-management]] — the survey's core object, but explicitly widened: memory managers are only one of six mechanism families, and paging/indexing leaves authority and recoverability implicit.
- [[memory-systems-anatomy-2026]] — closely adjacent wiki query on memory-system anatomy; this survey supplies the governance half that memory-form taxonomies omit.
- [[episodic-memory-llm]] — episodic traces are one of four "classical core" memory types in the taxonomy.
- [[long-context-llm]] — treated as a *boundary* class: long-context QA assumes the context is given, so it is out of scope unless the agent chooses what to keep.
- [[retrieval-augmented-generation]] — another boundary class (read-only curated corpus); also the most-studied lifecycle stage, and the pilot shows dense retrieval fails governance obligations identically to recency.
- [[agent-evaluation]] — AOEP-v0 is the survey's contribution here: score state mutation and recovery obligations, not answer quality.
- [[mirix]] / [[a-mem]] — agentic memory systems in the surveyed mechanism family; A-Mem is cited as the agentic note system whose entries link and refactor each other.
- [[hipporag-2]] — graph-backed structured memory store, part of the "memory managers and graph memory" family.

## References

- [arXiv:2606.30306](https://arxiv.org/abs/2606.30306)
