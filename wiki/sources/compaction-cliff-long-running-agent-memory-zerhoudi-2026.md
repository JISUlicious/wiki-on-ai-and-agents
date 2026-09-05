---
title: "The Compaction Cliff in Long-Running AI Agent Memory"
type: source
created: 2026-09-05
updated: 2026-09-05
sources: []
arxiv_id: "2608.22752"
venue: CIKM 2026
authors:
  - Saber Zerhoudi
  - Jelena Mitrović
  - Michael Granitzer
year: 2026
status: complete
importance: high
tags:
  - 2026
  - paper
  - safety
---

# The Compaction Cliff in Long-Running AI Agent Memory

**Saber Zerhoudi, Jelena Mitrović, Michael Granitzer** (University of Passau; Granitzer also IT:U Linz) — [arXiv:2608.22752](https://arxiv.org/abs/2608.22752), cs.AI / cs.IR, 24 Aug 2026, v1, CC BY 4.0. Published at **CIKM '26** (Rome, 7–11 Nov 2026), DOI `10.1145/3799682.3840567`. Corpus at `huggingface.co/datasets/searchsim/AgentArtifactCorpus`, code at `github.com/searchsim-org/cikm26-knowledge-triage`.

## Summary

The opening sentence is the whole thesis: *"A safety rule and an episodic log compete for the same tokens in an AI agent's context. When the budget overflows, both are summarized at the same rate; only the rule needs exact wording to remain enforceable."* Compaction is **type-blind** — it applies one fidelity target to heterogeneous knowledge — and a safety constraint is the one item type with **zero distortion tolerance**. Paraphrasing "never prescribe contraindicated drugs" into a summary that says the agent should "be careful with prescriptions" loses the qualifier that makes the rule enforceable, and nothing in the transcript signals it.

The measured failure is the **Compaction Cliff**: on 20 production agent configurations, Claude Code's `/compact` prompt on Sonnet 4.6 — invoked with the production instruction *"compress to N tokens, keep every safety rule and procedural command verbatim"* — preserves **53% of safety rules after one round and 10% after five** (50% compression per round). The decay reproduces across four LLM compactor families (gpt-5.4-nano, gpt-5.4-mini, Sonnet 4.6, Opus 4.7), three structural baselines, and the strongest non-LLM compressor, so it is a property of type-blind retention rather than of one prompt or one model.

**Knowledge Triage** is the fix, and it is explicitly an *architectural* one: classify every item **once, at indexing time**, into five types, then route each type through its own retention policy across all three context-management operations. The five types and their distortion tolerance:

| Type | Definition | Distortion tolerance | % of AAC items |
|---|---|---|---|
| **Constraint (C)** | Rule bounding agent behavior; violation is a safety failure | **Zero** — intact or lost | 12.3 |
| **Procedural (P)** | Step-by-step task instruction | Semantic equivalence iff execution preserved | 28.7 |
| **Belief (B)** | Factual assertion treated as true | Bounded by embedding distance | 31.4 |
| **Preference (F)** | Soft guideline | High; may be summarized or merged | 14.2 |
| **Episodic (E)** | Past event or observation | Free; may be discarded | 13.4 |

Three **deterministic** operators consume the labels — the classifier runs once at indexing and is *never re-invoked* by the operators, so per-item cost is constant in the number of compaction rounds, partitions, and queries:

- **TypeCompact** — three fidelity lanes (C and P at full, B and F compressed, E to placeholder), plus a deterministic verifier that extracts a canonical form (negation + object phrase) from each hard-lane constraint, checks it appears in the output, and restores the original or escalates to `Unsafe`. Feasible only above a **minimum safe budget** `B_min = Σ|i|` over constraints and procedures.
- **TypeDecompose** — partitions a too-large topic while **replicating each constraint into every partition its scope intersects**, enforcing *constraint locality*.
- **TypeRetrieve** — pins in-scope constraints ahead of relevance, then fills the residual budget by score, enforcing *priority*. Pinning is a database lookup, so it costs **zero LLM tokens per query**.

The default classifier, **SafetyMargin**, is the paper's sharpest idea. Instead of asking "is this sentence grammatically a rule?", it asks a counterfactual: `margin(i) = max_a [P(unsafe | a, K∖{i}) − P(unsafe | a, K)]`, scored 0–1 by one gpt-5.4-mini call, with >0.5 marking a Constraint. This exists because **declarative safety text has no imperative marker**: "the patient is allergic to penicillin" is a constraint that reads like a belief. And declarative phrasing is not a corner case — it accounts for **49.8% of 564 openFDA BOXED WARNING / CONTRAINDICATIONS sentences (30 drugs) and 61.1% of 36 LegalBench contract-NLI clauses**, against 22.5% / 22.2% imperative (Cohen's κ=0.88 four-class, 0.92 binary on a 100-sentence re-labelled subsample).

## Key points

**The cliff, and TypeCompact against it** (Table 6, mean across 20 AAC configurations; C = constraint preservation, P = procedural, time/tok = mean latency and LLM tokens per config):

| Strategy | C @50% | C @25% | C @10% | time @50% | tok @50% |
|---|---|---|---|---|---|
| **TypeCompact** | **1.00** | **0.95** | **0.80** | 0.0s | **0** |
| MaRS-FL (reimpl.) | 0.99 | 0.75 | 0.43 | 0.5s | 0 |
| Sonnet 4.6 (`/compact`) | 0.53 | 0.39 | 0.24 | 73.9s | 48K |
| gpt-5.4-nano (single-shot) | 0.52 | 0.26 | 0.14 | 9.9s | 56K |
| gpt-5.4-mini (single-shot) | 0.48 | 0.29 | 0.16 | 7.3s | 57K |
| Opus 4.7 (single-shot) | 0.45 | 0.26 | 0.17 | 31.1s | 50K |
| LLMLingua-2 | **0.21** | **0.08** | **0.01** | 0.5s | 0 |

Two things stand out beyond the headline. **[[llmlingua-2-pan-2024|LLMLingua-2]] is a near-total loss on constraints** (0.01 at 10%) because it weights tokens by entropy — the paper's example is that "Patient is allergic to penicillin. Never prescribe…" compresses at rate 0.5 *with the negation dropped*. And Sonnet 4.6's `/compact` gets **slower as it compresses harder** (73.9s → 166.2s → 420.3s), while TypeCompact is 0.0s and 0 tokens at every ratio because all the intelligence was spent at index time.

> [!warning] The "2–4×" claim is generous at the loosest ratio
> The abstract says TypeCompact "preserves 2–4× more safety rules than the strongest single-shot LLM compactor **at every ratio**." Recomputing from Table 6: against the best *single-shot* LLM the ratios are **1.9× / 3.3× / 5.0×** (50 / 25 / 10%); against Sonnet `/compact` they are **1.9× / 2.4× / 3.3×**. So the band holds at 25% and 10% but the 50% ratio is **below 2× on either reading**, and the 10% ratio exceeds 4× against a true single-shot baseline. The direction is unambiguous; the interval is not.

**By type at 50% compaction** (Table 5, 50 AAC configurations) — this table is the argument, because it shows the tradeoff is *deliberate*:

| Strategy | C | P | B | F | E | All |
|---|---|---|---|---|---|---|
| Hierarchical truncation | 0.50 | 0.82 | 0.75 | 0.63 | 0.77 | 0.69 |
| Temporal windowing | 0.58 | 0.80 | 0.64 | 0.78 | 0.91 | 0.74 |
| Aggressive pruning | 0.51 | 0.75 | 0.67 | 0.66 | 0.84 | 0.69 |
| **TypeCompact** | **1.00** | **1.00** | 0.50 | 0.51 | 0.84 | **0.77** |

TypeCompact *buys* full constraint and procedural retention by spending belief and preference fidelity down to ~0.50. Type-blind strategies spread the same budget uniformly and land at 0.50–0.58 on exactly the class that cannot tolerate it.

**Multi-round.** Five rounds at 50% per round drove `/compact` from **0.53 to 0.10**; TypeCompact **stabilizes at 0.96 from round two onward** and holds there. This is the wiki's [[catastrophic-remembering]] problem inverted — not accumulation without compounding, but *iterated loss*: the same single-round failure applied five times.

**The three ablations** (each removes one TypeCompact component; all three are load-bearing):

1. *Indexing-time labels.* [[llmlingua-2-pan-2024|LLMLingua-2]] plus a regex stage that re-inserts detected constraints **after** compression recovers 0.83 / 0.65 / 0.60. That closes about half the gap. The other half closes only when the label is assigned **before** compression — because declarative rules carry no marker for a regex to find.
2. *Verifier.* 27 restoration events across the run (mean 0.46 per call, max 1); never escalated to `Unsafe` at 50%.
3. *Budget boundary.* On the **1.4%** of configurations whose constraint-plus-procedural density exceeded 40%, `B_min` approached the budget: TypeCompact escalated to `Unsafe` on 28% and hit 1.00 on the remaining 72%, while the same runs **without** the verifier reported an apparent 1.00 recall while silently dropping a mean **57%** of the constraints they should have kept. Silent success is the failure mode the verifier exists to convert into a loud one.

**Decomposition** (Table 7, 200 AAC configurations, 25%-of-total partition budget):

| Strategy | Mean violation | Configs with ≥1 violation | Token overhead |
|---|---|---|---|
| `chunk_by_tokens` | 32% | 93% | 0% |
| `chunk_by_topic` | 13% | 40% | 0% |
| `chunk_by_paragraph` | 32% | 93% | 0% |
| **TypeDecompose** | **0%** | **0%** | **+14.5%** |

The 14.5% mean is driven by extremes: **median 0%** (74% of configurations have no global-scope constraints at all), 90th percentile 21%, **worst case 219%**. A separate scaling test on a 3,000-item topic (recursive split at T=200, KMeans over embeddings) produced 29 leaves of mean size 103; held-out retrieval at k=5 reached **94% recall against a 98% flat-scan bound while examining 92 of 3,000 items — a 33× reduction**.

**Retrieval** (Table 8; 1,000 BEIR scifact distractors + 33 τ-bench retail-policy chunks, of which the cascade flags 22 as constraints; 50 hand-written queries, ~12 in-scope constraints per query):

| Retriever | @5 | @10 | @20 | @50 | tok/query |
|---|---|---|---|---|---|
| [[bm25\|BM25]] | 32% | 38% | 43% | 49% | 0 |
| Dense (octen-embedding-8b) | 34% | 45% | 67% | 91% | 0 |
| Reranked dense | 25% | 36% | 55% | 91% | 0 |
| **TypeRetrieve (on dense)** | **40%** | **57%** | **96%** | **100%** | **0** |
| gpt-5.4-nano (one-shot) | 34% | 43% | 51% | 53% | 5,776 |
| gpt-5.4-mini (one-shot) | 34% | 46% | 64% | 70% | 5,800 |
| Sonnet 4.6 (one-shot) | 34% | 47% | 61% | 73% | 6,741 |

At k=5 everything is pinned to the structural bound min(5/12, 1) ≈ 42%. The gap opens at k=20–50, where score-only retrievers keep spending residual budget on high-similarity *non*-constraints while TypeRetrieve fills the in-scope set deterministically — **for zero tokens against 5,776–6,741 for the LLM rerankers**.

**Classifier cost** (Table 9, 200-item AAC test set; reference labels come from gpt-5.4-mini, so gpt-5.4-mini rows are self-consistency upper bounds):

| Variant | Constraint recall | C-F1 | Latency |
|---|---|---|---|
| Regex only | 0.60 | 0.63 | <1 ms |
| Encoder only | 0.27 | 0.32 | 387 ms |
| Distilled MiniLM+LR | 0.77 | 0.61 | ~5 ms |
| gpt-5.4-nano | **0.93** | 0.74 | 684 ms |
| gpt-5.4-mini | 0.87 | **0.84** | 667 ms |
| Sonnet 4.6 | 0.83 | 0.77 | 3,452 ms |
| Opus 4.7 | 0.73 | 0.80 | 3,853 ms |
| Qwen3-Next-80B | 0.87 | 0.83 | 606 ms |
| Selective cascade | **0.93** | 0.80 | 1,096 ms |

Cost-recall is **not monotone in model size**: gpt-5.4-nano matches the full cascade at 0.93, and Opus 4.7 drops to 0.73 because "its reasoning trace overflows the single-word output format" — a formatting artifact, not a capability one. Per-cell standard error is 3–5 points at N=200, so only large gaps are reliable.

**The adversarial-phrasing test is where SafetyMargin earns its place.** Rewriting 50 author-written safety rules in four grammatical forms (imperative, declarative, conditional, passive) gives 200 items with known labels. The LLM classifiers handle imperative, conditional and passive at **1.00 recall each** but lose **38–48 absolute points on declarative** phrasing: "the patient is allergic to penicillin" is recognized as a constraint by gpt-5.4-mini **62%** of the time, by the cascade **52%**, by regex **0%**. SafetyMargin lifts declarative recall to **0.90** (+28 points) at a cost of 4–10 points on the easy forms; mean recall across the four forms rises 0.91 → 0.93, at 880 ms vs 667 ms per item.

**Human verification.** Two annotators on a 100-cell sample: κ=0.77 four-class (85% agreement), **κ=0.92 binary preserved-vs-lost** (96%). Consensus agrees with the automated key-token metric on 79% of cells; disagreement concentrates on a "Weakened" class (28% of automated-Preserved cells, mostly at 10% compression where key tokens survive but a qualifier is dropped). Crucially, **humans are stricter and this costs the type-blind methods far more**: TypeCompact falls 5 points (91% → 86%) while temporal windowing falls 15 (65 → 50), hierarchical truncation 23 (64 → 41), and aggressive pruning 30 (71 → 41). The artifact-level gap *widens* under human judgment.

**Downstream behavior** — three benchmarks, paired McNemar exact two-sided:

| Benchmark | Full policy | Type-blind baseline | TypeCompact | p |
|---|---|---|---|---|
| **SafetyMed** (N=200) pass / preservation | 96.5% / 100.0% | Sonnet 92.5% / 81.0%; hierarchical 98.0% / 93.5% | 97.0% / 95.5% | vs Sonnet: pass p=0.022, preservation p=3.7×10⁻⁹ (14.5-pt lead) |
| **τ-bench retail** (115 tasks, 1,035 rollouts, 329 paired) | 28.6% | Hierarchical 29.2% | **37.7%** | p=0.003 vs full, p=0.005 vs hier. |
| **τ-bench airline** (50 tasks, 117 paired) | **34.2%** | Hierarchical 15.4% | 26.5% | p=0.024 vs hier.; ties full at p=0.14 |

Retail by model (Table 10): gpt-5.4-nano 22.5 → 36.0 (p=0.011), gpt-5.4-mini 27.3 → 35.5 (p=0.176), qwen3-next-80b 36.1 → 41.7 (p=0.345). **The entire significant gain is on the smallest model** — pinning the relevant rules helps most when the model is least able to locate them inside a full policy.

## Where it breaks

Stated limitations, plus what is visible from outside:

- **The guarantee is exactly the classifier's recall.** "A constraint is lost only when the classifier misses it." At SafetyMargin's 0.93 operating recall the residual miss rate is 0.07 — and that is precisely the nine SafetyMed items behind the 4.5-point gap to the full-policy ceiling (p=0.0039). *Per-type retention preserves no more constraints than the classifier detects.*
- **The retail win is not token-matched.** TypeCompact retained **1,136 tokens against hierarchical truncation's 669** on the same 1,338-token policy, so part of the 8.5-point gain may be retained context rather than typing. The authors flag this and offer two controls (the §4.3 artifact experiments are budget-matched; on airline the footprint *reverses* — 647 vs 718 tokens — and TypeCompact still leads by 11.1 points). A token-matched behavioral control is left open.
- **TypeCompact does not beat the full policy everywhere.** On airline it lands at 26.5% against the full policy's 34.2%, and on SafetyMed pass rate hierarchical truncation (98.0%) nominally beats it (97.0%). The honest claim is narrower than the abstract's framing: TypeCompact beats *the compactor a production system would actually deploy*, not compaction-free operation.
- **Reference labels are gpt-5.4-mini's own.** Table 9's gpt-5.4-mini-derived rows are self-consistency upper bounds. Against an independent human annotator, five-class Cohen's κ is only **0.45** (0.79 on the binary constraint-vs-other split), so per-type results away from the constraint boundary are weakly grounded.
- **Multi-round evidence is thin.** The five-round rollout ran on only **two of the four** LLM families.
- AgentArtifactCorpus is public GitHub only (396,934 artifacts from 54,628 repos across eight platforms; median 1,201 bytes; TypeScript 31.5% / Python 18.7% / Go 18.2%), so closed enterprise type distributions may differ; the authors recommend re-fitting before deployment.
- MaRS has no public implementation; Table 6's MaRS-FL is the authors' own reimplementation of the design they argue against.

> [!warning] Two internal number conflicts
> **(1)** §5 states "single-round survival is a consistent **19–42%** at 50% compression across all four [LLM families]," but Table 6 reports 0.45–0.53 constraint preservation at 50% for those same four compactors. The two ranges do not overlap and the paper does not reconcile them.
> **(2)** The ablation text says "LLMLingua-2 alone reached **0.55 / 0.18 / 0.02**" at 50/25/10%, while Table 6 reports **0.21 / 0.08 / 0.01** for LLMLingua-2 at the same ratios. Plausibly a 50-config vs 20-config difference, but it is not labelled as such.

## Reconciliation with [[lost-in-compaction-side-constraint-loss-wang-2026]]

These two papers are direct siblings — independent groups, three weeks apart, both measuring compaction destroying rules and both proposing a plug-in module rather than a better compactor prompt. They **agree on the fix and on the mechanism; the headline numbers are not comparable, and they conflict on one sub-mechanism.**

**Where they agree (strongly).**

| | Lost in Compaction (Wang et al.) | Compaction Cliff (Zerhoudi et al.) |
|---|---|---|
| Object destroyed | **Side/session constraints** — transient, user-stated, session-scoped ("show me the draft before sending") | **Safety constraints** — durable, authored config text in `CLAUDE.md` / `AGENTS.md` / system prompts |
| Root cause | Compactor prompts are written for **task continuity**; an SC is not the task | Compaction is **type-blind**; one fidelity target over heterogeneous knowledge |
| Non-LLM compressors | LLMLingua-2 retains **0.0%** on all three datasets | LLMLingua-2 preserves **0.21 / 0.08 / 0.01** |
| Prompt engineering as a lever | Real but weak — "strict" framing helps, combining framings adds ~1.3 pp; adding an explicit SC-preservation instruction still leaves WildChat below 40% | Real but weak — the production prompt *already says* "keep every safety rule verbatim" and still yields 0.53 |
| The fix | Separate SLM extractor over user turns; append verbatim: `H̃ = C(H) ⊕ S` | Separate classifier at index time; pin constraints at zero distortion in a hard lane + verifier |
| Fix's measured retention | **90.3–95.6%** | **1.00 @50%, 0.96 stable over five rounds** |
| Framing of the fix | "**architectural separation** — a strategy-layer module, not a better compactor prompt" | "**before delegating compaction to a frontier LLM, classify items by safety role** and retain the safety-critical class without modification" |

Both fixes are the same architectural move — a cheap model outside the compactor that identifies the constraint class and reinstates it verbatim — and both land in the same 90–96% band. That convergence from two different constructs, two different corpora and two different metrics is the strongest joint result here.

**Why the headline numbers (17% vs 53%) do not conflict.** They measure different things under different pressure:

- **Different construct.** Wang's SCs are by definition *not durable* — stated once, session-scoped, stored nowhere. Zerhoudi's constraints are *installed* configuration text. Wang's paper explicitly notes SCs are "not memory," which is why nothing else catches them.
- **Different compression pressure.** Wang compacts a **100K-token stitched context to ~182× shorter**. Zerhoudi compacts a small config artifact at **50% (i.e. 2×)**. Retention should be far higher at 2×, and it is.
- **Different metric.** LLM-as-a-judge presence-in-summary vs a deterministic key-token test on a canonical constraint form (κ=0.92 against humans).
- **They converge under matched pressure.** Wang's ~6% average over the six open/non-LLM compactors, and 17% over all 24 cells, sits right next to Zerhoudi's **round-five 10%** and his 10%-ratio **0.24**. Wang also reports retention around ~90% at a 10K context on Hermes Agent — i.e. at low pressure, closer to Zerhoudi's 53%. The two curves look like one curve sampled at different points.

**Where they genuinely conflict.** On *why* rules are lost positionally. Zerhoudi's §2 grounds the failure partly in **positional bias / lost-in-the-middle**: "Attention to long inputs is uneven: positional bias leaves middle content under-attended." Wang measured this directly and found the opposite shape — retention rises **monotonically with injection position** (later is better), explicitly **diverging from the U-shaped lost-in-the-middle pattern**, and attributes it to proximity between the constraint and the compaction instruction that follows the context. Zerhoudi invokes a mechanism Wang's data rules out for this task. It does not affect either paper's fix, since both pin or re-inject rather than reposition — but a practitioner reading Zerhoudi and concluding "put safety rules at the top and bottom, not the middle" would be following the wrong advice.

**One more asymmetry worth keeping.** Wang's fix reintroduces uncompacted text (`C(H) ⊕ S` grows without bound over a long session) and is evaluated on retention and latency but **not** on false positives. Zerhoudi's fix has a bounded footprint (constraints are ~12.3% of items), *does* measure classifier precision (0.91 constraint precision on AAC), and adds a verifier that converts silent loss into a loud `Unsafe`. Zerhoudi additionally covers decomposition and retrieval, which Wang does not touch. Conversely, Wang covers what Zerhoudi does not: constraints that arrive *mid-session in a user turn*, which have no indexing time at which to be classified. Zerhoudi concedes exactly this gap — "items whose constraint status depends on session context need an online τ."

## Related

- [[lost-in-compaction-side-constraint-loss-wang-2026]] — the direct sibling; see the reconciliation section above. Agreement on the fix, complementary coverage, one mechanistic conflict on positional bias.
- [[prompt-compression]] · [[llmlingua-2-pan-2024]] — LLMLingua-2 is the non-LLM baseline here and is a near-total loss on constraints (0.01 at 10% compression) because entropy weighting prunes `never` alongside filler.
- [[context-assembly]] · [[agentic-context-engineering]] — the layer the classifier is inserted into; "place a type classifier upstream of compaction, retrieval, and decomposition."
- [[memory-management]] · [[tiered-memory]] — the three fidelity lanes are a tiering scheme over knowledge *type* rather than over recency or access frequency.
- [[memory-consolidation]] — compaction is consolidation, and this paper measures what consolidation destroys.
- [[catastrophic-remembering]] — the round-five 0.10 is the iterated form: five applications of a per-round loss.
- [[procedural-memory]] — Procedural is one of the five types, retained at full fidelity beside Constraint (`B_min` is the sum of both).
- [[virtual-context]] · [[long-context-llm]] — compaction exists because the window is finite; `B_min` names when no compaction is safe and decomposition must take over.
- [[why-does-claude-md-keep-growing-chakrabarti-2026]] — the other half of the `CLAUDE.md` lifecycle: that paper measures the file growing, this one measures what happens when the runtime is forced to shrink it.
- [[context-as-an-environment-scroll-lin-2026]] — the alternative architecture: Scroll never compresses at all, keeping an append-only log so nothing needs to be triaged. Knowledge Triage instead accepts compression and protects one class inside it. Both reject "decide what survives before you know what is needed," from opposite directions.
- [[acm-agentic-context-management-li-2026]] — attacks *when* compaction fires; this paper attacks *what fidelity* it applies to which type.
- [[memharness-memory-is-reconstructed-not-replayed-wu-2026]] · [[zero-mem-zero-token-memory-operations-xiao-2026]] — neighbouring memory-architecture positions; TypeRetrieve's zero-token pinning is the same instinct as zero-token memory ops (do the work at index time, not query time).
- [[memgpt-packer-2023]] · [[a-mem-xu-2025]] — both appear in the paper's positioning table (Table 1) as untyped systems with no per-operator safety guarantee.
- [[memory-construction]] · [[memory-evolution]] — the classifier runs once at construction; the operators are deterministic thereafter, which is what keeps cost constant across rounds.
- [[guardrailing]] · [[agentic-misalignment]] — a paraphrased safety rule is a silently disabled guardrail; the SafetyMed penicillin scenario is the concrete harm.
- [[constitutional-ai-bai-2022]] — cited as writing safety preferences as a fixed rule set, and flagged as vulnerable to the same context-manipulation pressure.
- [[claude-code]] · [[anthropic]] — Claude Code's `/compact` on Sonnet 4.6 is the production system measured; the corpus spans eight harness platforms (Claude, Cursor, Copilot, Windsurf, Continue, Aider, Codeium, Universal).
- [[agentic-harness-engineering]] — the compaction prompt is a harness artifact, and its retention rate is a correctness variable.
- [[longmemeval]] — 500 LongMemEval turns are the cross-domain check on the five-type taxonomy (macro-F1 0.84 vs 0.87 on AAC; constraint F1 holds at 0.88 on both).
- [[bm25]] · [[dense-retrieval]] · [[retrieval-augmented-generation]] — the retrieval baselines TypeRetrieve overlays; the paper's point is that relevance scoring has no channel for constraint membership.
- [[agent-benchmark]] · [[agent-evaluation]] — SafetyMed (200 scenarios from MedQA + openFDA) and AgentArtifactCorpus are both released artifacts.
- [[llm-as-a-judge]] — SafetyMargin is an LLM judge scoring a counterfactual safety margin rather than grammatical form.
- [[knowledge-graph]] — noted as complementary: a graph can host typed items as labelled nodes with no extraction step, since rules already live as natural-language strings.

> [!note] Concept gaps this page could not link
> No wiki page yet for **τ-bench** (used here for both retail and airline behavioral evaluation), **context compaction** as a first-class concept distinct from [[prompt-compression]], **constraint preservation / typed memory**, or **rate-distortion theory for composite sources** (the paper's formal frame, after Liu et al.). AgentArtifactCorpus and SafetyMed are new released datasets with no page.

## References

- [arXiv:2608.22752](https://arxiv.org/abs/2608.22752) · CIKM '26 · DOI `10.1145/3799682.3840567`
- Dataset: `huggingface.co/datasets/searchsim/AgentArtifactCorpus` (396,934 artifacts, 54,628 repos, 2,000 per-type annotations)
- Code: `github.com/searchsim-org/cikm26-knowledge-triage`
- Pan et al. 2024, *LLMLingua-2* — the non-LLM compactor baseline.
- Packer et al. 2023, *MemGPT*; Xu et al. 2025, *A-MEM*; Edge et al. *GraphRAG*; *MemOS*; Alqithami 2025, *MaRS* — the positioning table (Table 1).
- Liu et al., rate-distortion for composite sources under subsource-dependent fidelity — the formal frame instantiated with knowledge types as subsources.
- Yao et al., *τ-bench* (retail, airline); *MedQA*; openFDA BOXED WARNING / CONTRAINDICATIONS; *LegalBench* contract-NLI; *BEIR* scifact; *LongMemEval* — evaluation corpora.
- Liu et al. 2024, "lost in the middle" positional bias; *LongBench*; *RULER* — cited as explanations for compaction loss (but see the conflict with Wang et al. 2026 above).
