---
title: "SKILL.state: Scalable Long-Horizon Agent Skills"
type: source
created: 2026-09-07
updated: 2026-09-07
sources: []
arxiv_id: "2608.26263"
venue: EMNLP (accepted)
authors:
  - Sanket Badhe
  - Priyanka Tiwari
  - Jonghyun Chung
year: 2026
status: complete
importance: high
tags:
  - 2026
  - paper
  - harness
---

# SKILL.state: Scalable Long-Horizon Agent Skills

**Sanket Badhe, Priyanka Tiwari** ([[google|Google LLC]]), **Jonghyun Chung** ([[purdue-university|Purdue University]]) — [arXiv:2608.26263](https://arxiv.org/abs/2608.26263), cs.AI / cs.MA, v1 26 Aug 2026, **v3 2 Sep 2026** (the version summarized here). **Accepted at EMNLP.** CC BY 4.0. Corresponding author Sanket Badhe (`sanketbadhe@google.com`), also first author of the agent-skills survey and the agent-skill-security paper this cites — cf. [[agent-skills-survey-xu-2026]] for the wiki's coverage of that literature.

## Problem

Every mainstream agent runtime — [[react|ReAct]]-style loops, summarizing memory systems, LangGraph-style stateful graphs — executes a long procedure by **appending**. At step `t` the model receives the original skill spec plus a transcript of every prior reasoning trace, action, observation, and tool output. Two consequences:

1. **Prompt size grows with horizon.** Per-step prompt is `O(t)`, so cumulative token cost over a horizon `T` is `O(T²)`.
2. **Obsolete facts stay resident.** Historical observations and superseded reasoning remain in context long after they stop being true, and the model must re-derive "what is the case now" from "what was said when" at every step.

The paper's framing is that memory systems (summarization, retrieval — [[memgpt]], [[mem0]], [[generative-agents-park-2023]]) *alleviate* the first problem but preserve the same **execution semantics**: future decisions are still conditioned on a textual reconstruction of the past rather than on an explicit representation of the present. Even LangGraph, which carries a structured state block, carries it *alongside* the full rolling transcript.

## Mechanism

At each step the model receives exactly three things:

```
A_t = (P, Σ_t, O_t)
```

`P` = the immutable procedural specification. `Σ_t` = the structured execution state. `O_t` = the latest observation. **The model never receives previous observations, previous actions, or previous reasoning traces.**

It emits a triple:

```
(R_t, ΔΣ_t, a_t)
```

`R_t` = a multi-step chain-of-thought trace, `ΔΣ_t` = a structured state update (a JSON dictionary of key mutations and deletions), `a_t` = the action to execute. The runtime deterministically **validates** `ΔΣ_t`, applies `Σ_{t+1} = Σ_t ⊕ ΔΣ_t` where `⊕` is a dictionary merge with null-deletion semantics, executes `a_t`, and loops.

Two design details carry the weight:

- **Within-step reasoning is fully intact.** [[chain-of-thought]] is not truncated or budgeted during generation; complex deductive planning happens normally. It is *after* the transition is validated that `R_t` is **discarded permanently and never appears in any subsequent prompt**. Transient reasoning is projected into persistent structured state, and only what survives the projection survives at all.
- **Schema ownership lives in the runtime, not the model.** Schemas are authored **once per domain**, not per task — across all 100 InterCode CTF challenges the agent reuses a single static five-field schema (`discovered_flags`, `tested_hypotheses`, `active_files`, `working_dir`, `cmd_summary`). Because validation is deterministic and external, a malformed patch cannot corrupt `Σ_t`; an invalid patch triggers a rollback-retry. See [[json-schema]] and [[structured-outputs]].

**Complexity.** Conversational runtimes: `|C_t| = O(t)`, cumulative `Σ|C_t| = O(T²)`. SKILL.state: `|P_t| = O(|P| + |Σ| + |O|)`, independent of `t` — i.e. **`O(1)` per step** — so cumulative cost is `O(T)`.

The paper distinguishes itself from **Dialogue State Tracking** on exactly this point: DST maintains structured slots *alongside* full transcripts in quasi-static dialogue, whereas SKILL.state treats the state as a **sufficient statistic** and throws the transcript away.

## Measured results

**Experiment 1 — Warehouse Management long-horizon scaling** (Gemini-3-Flash, 500 independent shelves, mean ± SD over 5 seeds). This is the quadratic-growth result:

| T | Runtime | Score | Avg prompt (chars) | Total tokens |
|---|---|---|---|---|
| 10 | Prompt (ReAct) | 0.90 ± 0.02 | 3,249 | 9,438 |
| | Memory (Summary) | 1.00 | 3,300 | 9,972 |
| | Stateful (LangGraph) | 1.00 | 3,430 | 10,337 |
| | **SKILL.state** | **1.00** | **1,775** | **5,870** |
| 50 | Prompt (ReAct) | 0.88 ± 0.04 | 11,931 | 171,658 |
| | Memory | 0.93 ± 0.03 | 7,582 | 131,455 |
| | Stateful | 0.94 ± 0.00 | 11,594 | 170,992 |
| | **SKILL.state** | **0.96 ± 0.01** | **1,773** | **30,151** |
| 100 | Prompt (ReAct) | 0.84 ± 0.07 | 36,362 | 1,245,413 |
| | Memory | 0.87 ± 0.05 | 29,607 | 1,082,154 |
| | Stateful | 0.91 ± 0.02 | 31,354 | **1,062,387** |
| | **SKILL.state** | **0.94 ± 0.01** | **1,905** | **65,408** |
| 200 | Prompt (ReAct) | 0.74 ± 0.14 | 48,007 | 2,608,755 |
| | Memory | 0.84 ± 0.09 | 84,364 | **6,175,509** |
| | Stateful | 0.88 ± 0.03 | 72,305 | 5,041,164 |
| | **SKILL.state** | **0.94 ± 0.02** | **1,811** | **122,384** |

The two quoted figures verify exactly: at T=100 the Stateful baseline burns **1,062,387 tokens against SKILL.state's 65,408 — 16.2×**; at T=200 SKILL.state holds **0.94 accuracy on 122k tokens** while the Memory baseline inflates to **6.1M**. The `O(1)` claim is visible in the prompt column: **1,775 chars at T=10 and 1,811 at T=200**, flat across a 20× horizon increase, while Stateful's prompt goes 3,430 → 72,305. Baseline accuracy *falls* monotonically with horizon (ReAct 0.90 → 0.74) while SKILL.state's degrades far less (1.00 → 0.94). Differences at T ≥ 50 are reported as statistically significant by paired t-test over the 5 seeds.

**Does accuracy really improve while tokens drop?** Yes, but with real caveats the abstract does not carry:

| Setting | Accuracy verdict |
|---|---|
| Warehouse, Gemini-3-Flash, T ≤ 25 | **Tie** at 1.00 with Memory and Stateful (only ReAct is beaten) |
| Warehouse, Gemini-3-Flash, T ≥ 50 | **Clear win** (0.96/0.94/0.94 vs 0.94/0.91/0.88 best baseline) |
| Software Repository, T=25 | **Loss**: SKILL.state 0.88 vs Stateful 0.94, Memory 0.89 |
| Software Repository, T ≥ 50 | **Clear win**: 0.86 vs 0.74; 0.78 vs 0.63 at T=100 (with 90,200 tokens vs 2,308,000) |
| Warehouse, Gemma-4-31B-it, T=100 | **Tie**: 0.42 vs Stateful 0.42 — but at 65,480 vs 557,968 tokens (8.5×) |
| All three public benchmarks | **Win** on both axes |

So the honest statement is: **the token reduction is universal and large; the accuracy gain is horizon-dependent and appears from roughly T ≥ 50.** In the entangled-graph Software Repository environment at medium horizon, discarding history actually costs accuracy — which is precisely what you would expect when a single action (merging a PR) mutates the state of dependent objects.

**Experiment 2 — noise robustness** (Warehouse, T=50, distractor events injected per turn):

| Noise / turn | Prompt | Memory | Stateful | SKILL.state |
|---|---|---|---|---|
| 5 events | 0.68 | 1.00 | 1.00 | 1.00 |
| 20 events | 0.61 | 1.00 | 0.98 | 0.97 |
| 50 events | 0.53 | 0.96 | 0.98 | 0.98 |

ReAct collapses (0.68 → 0.53); SKILL.state stays ≥ 0.97 because distractors are filtered during patch generation and **never enter a subsequent prompt**. Note that Memory and Stateful hold up here too — this experiment separates SKILL.state from raw appending, not from the better baselines.

**Experiment 3 — state recovery under silent external drift** (an outside actor mutates the world without the agent acting):

| Scenario | Prompt / Memory / Stateful | SKILL.state |
|---|---|---|
| A: Secret Audit | recovers, **5–8 turns** | recovers, **0 turns** |
| B: Secret Barcode | recovers, 6–8 turns | recovers, **0** |
| C: Secret Move | recovers, 5–8 turns | recovers, **0** |
| D: Canceled Order | **fails** | **fails** |

This is the cleanest result in the paper. History-based runtimes hallucinate for five to eight consecutive turns because obsolete facts in the transcript **overpower the contradictory new observation**; SKILL.state corrects instantly because the state, not the history, is what the decision reads. Scenario D fails for every runtime, including SKILL.state — the paper reports it without spin.

**Experiment 4 — public interactive benchmarks** (Gemini-3-Flash):

| | InterCode CTF pass@1 | tokens | τ-Bench Retail | tokens | τ-Bench Airline | tokens |
|---|---|---|---|---|---|---|
| Prompt (ReAct) | 43.2% | 977k | 48.2% | 4.48M | 21.8% | 4.85M |
| Memory | 46.4% | 1.03M | 29.9% | 4.24M | 23.6% | 4.65M |
| Stateful | 41.8% | 1.13M | 51.7% | 3.92M | 28.1% | 5.28M |
| **SKILL.state** | **54.2%** | **387k** | **58.3%** | **3.47M** | **32.4%** | **2.88M** |

CTF: **+7.8 points over the strongest baseline** and +12.4 over Stateful, at **−60.4% tokens vs ReAct and −65.9% vs Stateful** — the paper credits `tested_hypotheses` in the schema for preventing the model from repeating failed commands. Airline: baseline prompts peak **above 11,000 tokens/step** on complex database responses while SKILL.state stays flat at ~2,800. One honest wrinkle: in **τ-Bench Retail SKILL.state's average prompt (3,325 chars) is the *largest* of the four runtimes** and yet its total tokens are lowest — bounded is not the same as small; it is flat.

**Experiment 5 — budget-matched controls** (Warehouse, T=100, all pinned to ~1,800 tokens). This is the ablation that matters, because it separates "shorter prompt" from "structured state":

| Runtime | Score | Avg prompt | Total tokens |
|---|---|---|---|
| Full ReAct (unbounded) | 0.84 | 36,362 | 1,245,413 |
| Truncated (sliding window) | **0.18** | 1,800 | 62,100 |
| Summary-capped | 0.52 | 1,840 | 63,400 |
| ReAct + LLMLingua | **0.22** | 1,810 | 62,350 |
| **SKILL.state** | **0.94** | 1,905 | 65,408 |

At an identical budget, sliding-window truncation lands at **0.18** and [[llmlingua-jiang-2023|LLMLingua]] perplexity compression at **0.22** — both *far worse than the unbounded ReAct they compress*. The diagnosis is specific: entropy-based filtering deletes "seemingly redundant slot identifiers that are semantically vital," and truncation evicts early inventory allocations that are still live. Across horizons the collapse is progressive:

| T | SKILL.state | Summary-capped | Truncated | LLMLingua | ReAct (full) |
|---|---|---|---|---|---|
| 10 | 1.00 | 0.92 | 0.90 | 0.88 | 0.90 |
| 25 | 1.00 | 0.76 | 0.62 | 0.60 | 0.92 |
| 50 | 0.96 | 0.64 | 0.35 | 0.38 | 0.88 |
| 100 | 0.94 | 0.52 | 0.18 | 0.22 | 0.84 |

This is a strong negative result for statistical [[prompt-compression]] as a long-horizon execution strategy, and it is the paper's best evidence that the win comes from **structure**, not from **brevity**.

**Error taxonomy on open-weight models** ([[gemma|Gemma]]-4-31B-it at T=100, score 0.42):

| Failure mode | Share |
|---|---|
| Premature state overwrite / deletion (omits existing keys instead of merging in place) | **68%** |
| Schema comprehension / type coercion (nested lists vs dicts) | 20% |
| JSON syntax / formatting slips | 12% |

The authors' reading — that degradation is **structured-output adherence, not reasoning capacity** — motivates [[constrained-decoding]] as the fix. Note the dominant mode is the model *deleting too much*, which is the mirror image of the append-only failure the architecture was built to escape.

## Relation to catastrophic remembering

SKILL.state is the most direct engineered counter to [[catastrophic-remembering]] in the corpus so far, and the comparison is worth making precisely, because it does **not** apply the fix that concept page identifies as load-bearing.

**What it does attack.** The ratchet in [[catastrophic-remembering]] runs on three assumptions: additions are stochastic, outcomes are censored, and instructions are redundant (which makes honest deletion cost `O(2^|D|)` subset probes). SKILL.state removes the ratchet by **removing the append channel entirely**. There is no accumulating store of reasoning traces or observations to audit, so the closed form `|D_∞| = A/(ρ̄·s) → ∞` never starts: the addition rate into a persistent substrate is zero for everything except validated state fields, and those are capped by a fixed schema. The empirical signature is the flat prompt column — 1,775 → 1,811 characters across a 20× horizon. That is a genuine structural fix, not a better-pruning heuristic, and it is the reason it beats summarization and compression rather than joining them.

**What it does not do.** The concept page's through-line is that *the ratchet is caused by missing provenance, so the fix is to supply provenance* — Chakrabarti's stripped prompt comments carrying rationale, [[memharness-memory-is-reconstructed-not-replayed-wu-2026|MemHarness]]'s reconstruct-or-reject, Rehearse's candidate-relevant retrieval. Every one of those **keeps the underlying material and adds the metadata needed to prune it safely**. SKILL.state does the opposite: it **keeps only the pruned result and destroys the material**. The deletion decision is made eagerly, at write time, by the model, and it is irreversible.

The authors are admirably explicit that this is the trade. Their Limitations section names three settings where the sufficient-statistic assumption fails, and the second and third are exactly the provenance problem:

> (1) when no fixed schema is known in advance and the relevant state structure must be discovered dynamically during execution; (2) **when a correct state update depends on an earlier observation whose relevance was not recognized when first observed, and was therefore never committed to state**; and (3) **when the task objective is defined over the historical trajectory itself (e.g., auditing, debugging provenance, or explaining past actions)**, where interaction history is the target output rather than operational overhead.

Failure (2) *is* the censored-outcome assumption, relocated rather than solved. Catastrophic remembering says you cannot safely delete because you do not know why anything was added; SKILL.state requires you to know, at step `t`, what will matter at step `t+40`. Failure (3) is sharper still: a system that discards history **cannot produce provenance**, because it no longer has any.

**The one place provenance does appear** is instructive. The InterCode CTF schema's `tested_hypotheses` field is *negative* provenance — a durable record of what was tried and failed — and the paper credits it directly with the +7.8-point pass@1 gain ("prevents the model from repeating failed commands"). So provenance survives here **when a human schema author anticipated the need for it**. That relocates the hard problem from runtime deletion policy to design-time schema design, which is a much better place for it (you decide once per domain instead of continuously, and you decide with the whole task in view) — but it does not eliminate it. Note also that the dominant open-weight failure mode is **premature state overwrite/deletion at 68%**, i.e. the model getting the write-time deletion decision wrong, which is the risk this design concentrates rather than removes.

**Verdict:** SKILL.state converts *unbounded growth with unsafe deletion* into *bounded growth with lossy, irreversible writes*. It is a real fix for the ratchet, achieved by throwing information away on a schedule the schema author chose in advance — not by supplying the provenance that would make deletion safe. It should be read as complementary to the provenance line, not as a substitute: the two answer different halves of the same problem, and the paper's own scope carve-outs say where each belongs.

Worth contrasting with [[continualskillbench-guan-2026]]'s finding (recorded on [[catastrophic-remembering]]) that maintaining an explicit *skill library* scores 0.602 vs 0.605 for plain [[in-context-learning|in-context learning]] — no aggregate advantage. SKILL.state's explicit structure is a different object (per-step execution state, not a growing library of reusable artifacts) and does show a clear advantage, which suggests the useful distinction is **mutable-and-bounded** versus **accumulating**, rather than structured versus unstructured.

## Limitations

Beyond the three sufficient-statistic failures above:

- **Single-agent only.** The authors note the abstraction extends naturally to multi-agent settings — a shared execution state as coordination substrate instead of quadratic conversational transcripts — but concurrent writes require deterministic conflict-resolution semantics in `⊕` that the single-agent setting never exercises.
- **Depends on the model emitting valid patches.** Mitigated by external validation and rollback-retry (state cannot be corrupted), but on Gemma-4-31B the retry loop is not enough: 0.42 at T=100 is a tie with Stateful, not a win.
- **Schemas are hand-authored per domain.** Two synthetic environments and two public benchmarks, four schemas. There is no experiment on schema-authoring cost or on schema transfer.
- The controlled testbed (SkillExecBench) is procedurally generated with deterministic ground truth, so its accuracy metric (successful actions / total actionable events) is a continuous execution-fidelity score, not a task success rate. Only CTF and τ-Bench report binary success.

## Related

- [[catastrophic-remembering]] — the failure this is engineered against; see the analysis above for why it is a *discard* fix rather than a *provenance* fix.
- [[agent-skills]] · [[skill-md-format]] · [[skill-library]] — the procedural-skill framing `P` comes from; `P` is the immutable spec, the innovation is what sits beside it.
- [[memory-management]] · [[virtual-context]] · [[memgpt]] · [[mem0]] — the memory-architecture line the paper positions against: these change *what is reconstructed*, SKILL.state changes *whether anything is reconstructed*.
- [[context-assembly]] — the prompt is rebuilt from `(P, Σ_t, O_t)` every step rather than extended; this is context assembly taken to its limit.
- [[prompt-compression]] · [[llmlingua-jiang-2023]] · [[llmlingua-2-pan-2024]] — beaten decisively at matched budget (0.94 vs 0.22); the paper is a strong negative result for entropy-based compression in long-horizon execution.
- [[react]] · [[react-yao-2022]] — the append-only baseline.
- [[harness-1-state-externalizing-search-agents-jiang-2026]] · [[externalization-in-llm-agents-zhou-2026]] · [[filesystem-based-memory-for-llm-agents-zhou-2026]] · [[stateless-decision-memory-for-enterprise-ai-agents-srinivasan-2026]] · [[zero-mem-zero-token-memory-operations-xiao-2026]] — the adjacent state-externalization cluster.
- [[lost-in-compaction-side-constraint-loss-wang-2026]] · [[compaction-cliff-long-running-agent-memory-zerhoudi-2026]] — what happens when you compress history instead of replacing it; the budget-matched controls here are the same result from the other direction.
- [[context-as-an-environment-scroll-lin-2026]] — the complementary framing: context as an addressable environment rather than a transcript.
- [[json-schema]] · [[structured-outputs]] · [[constrained-decoding]] — the patch format, and the proposed fix for the 68% overwrite/deletion failure mode on small models.
- [[chain-of-thought]] — intact within a step, destroyed between steps. Contrast [[trace-as-state-reasoning-traces-as-conditional-states-zou-2026]], ingested alongside this paper, which does the *opposite*: it preserves the reasoning trace and re-injects it as state. The two papers agree that traces carry state and disagree entirely about whether to keep them.
- [[tau-bench]] — the Sierra τ-Bench benchmark used in Experiment 4.
- [[agent-benchmark]] · [[llm-agent]] · [[tool-use]] · [[procedural-memory]] · [[long-term-memory]]

> [!note] Wikilink gaps
> [[tau-bench]] (Sierra τ-Bench, used here and a standard agent benchmark) and [[purdue-university]] now have pages. `jie-tang` (co-author of the companion Trace-as-State ingest) is still absent from `wiki/entities/`.

## References

- [arXiv:2608.26263](https://arxiv.org/abs/2608.26263) — v3, 2 Sep 2026; accepted at EMNLP.
- Cited within: InterCode CTF (Yang et al. 2023), Sierra τ-Bench (Yao et al. 2024), LangGraph, LLMLingua (Jiang et al. 2023) → [[llmlingua-jiang-2023]], MemGPT (Packer et al. 2023) → [[memgpt-packer-2023]], Mem0 (Chhikara et al. 2025) → [[mem0-chhikara-2025]], [[generative-agents|Generative Agents]] (Park et al. 2023) → [[generative-agents-park-2023]], ReAct (Yao et al. 2022) → [[react-yao-2022]], Badhe et al. 2026 agent-skills survey.
