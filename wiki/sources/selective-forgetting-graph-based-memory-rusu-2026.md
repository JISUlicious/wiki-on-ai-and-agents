---
title: "Selective Forgetting: A Graph-Based Memory Framework for Long-Term LLM Agents"
type: source
created: 2026-09-07
updated: 2026-09-07
sources: []
arxiv_id: "2608.28978"
authors:
  - Theo Rusu
  - Sourena Khanzadeh
  - Manar Alalfi
year: 2026
status: complete
importance: high
tags:
  - 2026
  - paper
---

# Selective Forgetting: A Graph-Based Memory Framework for Long-Term LLM Agents

**Theo Rusu, Sourena Khanzadeh, Manar Alalfi** (Department of Computer Science, Toronto Metropolitan University; Khanzadeh also Flybits / The Creative School) — [arXiv:2608.28978](https://arxiv.org/abs/2608.28978), cs.AI, submitted 29 Aug 2026, v1. CC BY 4.0; no venue given. Code: `github.com/skhanzad/Selective-Amnesia`.

## Summary

This is a **negative result on graph-structured agent memory, published by authors who built the graph system**. The stated purpose is to test an assumption rather than to advance a system: knowledge graphs have been proposed as the structured alternative to flat [[retrieval-augmented-generation|RAG]] for long-term memory "on the assumption that representing conversations as entities and relations improves recall. We evaluate that assumption directly."

The framework is a conventional three-stage pipeline — retrieve a two-hop subgraph, update it by LLM extraction from each conversational turn, and periodically prune low-importance nodes. The paper then runs two experiments on [[longmemeval]] that point in **opposite directions**, and the value of the paper is entirely in that split:

1. **Graph *retrieval* loses.** At a matched candidate-generation budget of five retrieval roots, the graph scores token F1 **0.417 against 0.468** for a flat vector baseline — a paired bootstrap over all 500 questions puts the gap at **Δ = −0.050, 95% CI [−0.085, −0.016]**. It loses on five of six question categories.
2. **Graph *forgetting* works.** Applied once to a persistent 27,021-node graph, the importance-based pruner removes **9.8% of nodes and 9.5% of stored bytes with token F1 statistically unchanged** (+0.001, 95% CI [−0.015, +0.016]).

The mechanism the authors propose for (1) is the paper's most transferable idea: **extraction destroys surface form**. The deficit is widest precisely on questions that need a specific prior assistant turn, where judged correctness falls from **0.911 to 0.607** — the flat baseline can return the original assistant turn verbatim, while graph extraction has already decomposed that turn into typed entities and lost which item was emphasized.

The authors bound their own claim explicitly and repeatedly: a single small extractor (GPT-4o-mini) on a single benchmark "characterise[s] this extraction-based pipeline rather than graph-structured memory in general."

## Mechanism

**Schema.** Typed nodes and attributed edges over a fixed nine-label ontology: `Person, Organization, Location, Event, Concept, Artifact, Preference, Goal, Skill`. Each node carries a label, title, natural-language content description, a flat attributes dictionary, and retention bookkeeping (`created_at`, `access_count`, `last_accessed_at`, `turns_at_creation`, `importance_score`). Edges carry a typed predicate plus their own attributes, so relational properties (duration, confidence, quantity) do not spawn extra nodes.

**Update.** One GPT-4o-mini extraction call per conversational turn, with turns prefixed `[Role: user]` or `[Role: assistant]` so the extractor treats them differently — user turns yield facts about the user; assistant turns yield both user-related facts the assistant confirmed *and* factual claims/recommendations the assistant stated. Output is strict JSON; a validation layer rejects unparseable output, undefined ontology types, or invalid node-edge references. Filler-only turns produce empty output.

**De-duplication** is two checks in sequence: an O(1) `title_index` lookup on normalized (label, title), and — only on a miss — a linear cosine scan against all stored `nomic-embed-text` (Ollama) vectors with a merge threshold of **0.92**. The threshold is deliberately conservative because the errors are asymmetric: a false merge "collapses two distinct entities irreversibly and corrupts every edge incident to them," while a missed merge only leaves redundancy.

**Retrieval.** A lightweight extraction call pulls the entities referenced by the question, embeds their descriptors, and runs an exhaustive cosine scan with a floor of **0.75**; the **top-5** nodes seed a BFS expanded to **two hops** and capped at **15 nodes**, serialized into the answer prompt.

**Retention.** Every **400 turns**, each node is scored

`Score = 0.35·recency(t) + 0.25·frequency(c) + 0.20·centrality(d) + 0.20·turns_decay(k)`

with recency an exponential decay on last-access with a **90-day half-life**, frequency the log-normalized access count, centrality log-scaled edge degree, and turns_decay an exponential decay from creation turn with a **1,000-turn half-life**. Nodes scoring below **0.10** are pruned along with every incident edge.

**The fairness control that makes the result load-bearing.** Retrieval roots are set to 5 *by convention, to match* the five chunks retrieved by the flat baseline, "so that the two systems are compared at an equal candidate-generation budget and any difference in performance is attributable to the representation rather than to the number of retrieval hits." Both systems use the same generator; neither sees anything outside the question's own haystack. This is what turns the comparison from a tuning artifact into a claim about representation.

## Setup

[[longmemeval]]: **500 questions**, each paired with a multi-session haystack; sessions carry real historical timestamps spanning roughly **33 months (June 2021 – February 2024)**. Six categories probe distinct memory demands (single-session user / assistant / preference, knowledge-update, multi-session, temporal reasoning).

- **Experiment 1** builds a *fresh* graph per question from that question's haystack alone. Baseline stores the same turns as raw text chunks, top-5 retrieval.
- **Experiment 2** ingests all 500 haystacks into **one persistent graph**, simulating a long-running deployment, then compares no-forgetting against forgetting-applied-once.
- Metrics: token precision/recall/F1 plus a binary LLM-as-judge correctness score ([[llm-as-a-judge]]). GPT-4o-mini serves as extractor, generator, *and* judge. Temperature 0.
- Statistics: nonparametric bootstrap, 10,000 resamples over questions, percentile method; paired bootstrap matched by question ID; Wilson intervals additionally reported for the bounded judge proportion.
- Hardware is modest and disclosed: an AMD Radeon RX 7800 XT, 16 GB system RAM, Intel Core i5-9400F, with [[openai-api|OpenAI API]] for GPT-4o-mini. Budget admitted **four complete runs total** (two per experiment).

## Results

### Experiment 1 — graph vs. flat vector, matched at five retrieval roots

| Question type | n | Graph P | Graph F1 | Graph J | Base P | Base F1 | Base J |
|---|---|---|---|---|---|---|---|
| Single-session (user) | 70 | 0.743 | 0.737 | 0.771 | 0.823 | 0.819 | **0.929** |
| Single-session (assistant) | 56 | 0.680 | 0.575 | 0.607 | 0.916 | 0.774 | **0.911** |
| Single-session (preference) | 30 | **0.312** | 0.083 | 0.233 | 0.274 | 0.114 | 0.367 |
| Knowledge-update | 78 | 0.507 | 0.456 | 0.513 | 0.553 | 0.511 | 0.590 |
| Multi-session | 133 | **0.384** | 0.326 | 0.398 | 0.381 | 0.342 | 0.436 |
| Temporal reasoning | 133 | **0.468** | 0.328 | **0.293** | 0.416 | 0.334 | 0.278 |
| **Overall** | **500** | 0.505 | **0.417** | 0.454 | 0.532 | **0.468** | 0.536 |

**Paired bootstrap, Graph − Baseline** (∗ = significant at α = 0.05):

| Metric | Δ | 95% CI |
|---|---|---|
| F1 | **−0.050**∗ | [−0.085, −0.016] |
| Precision | −0.028 | [−0.067, +0.011] |
| Recall | **−0.073**∗ | [−0.107, −0.039] |
| Judge accuracy | **−0.082**∗ | [−0.128, −0.034] |

Three of four metrics move significantly against the graph. Precision is the lone non-significant one, and the per-type table shows why: the graph's precision is *higher* on preference, multi-session, and temporal questions — it returns fewer, tighter tokens — but its **recall collapses** because the entities it returns no longer contain the answer's surface form.

**The single win is temporal reasoning**, and only on the judge metric (0.293 vs. 0.278). The authors' reading is the charitable and probably correct one: events become typed nodes with temporal attributes wired to their participants, which flat chunk retrieval does not represent at all. Note the win is small and its own F1 is still down (0.328 vs. 0.334).

**The single largest deficit is single-session (assistant): judge 0.607 vs. 0.911, F1 0.575 vs. 0.774.** These questions require recalling a specific recommendation or factual statement the assistant made. This is the paper's core diagnosis, quoted in full because it generalizes past this pipeline: *"structured abstraction can improve relational organization while simultaneously discarding information required for precise or verbatim recall."*

Second-order finding on **knowledge-update** (F1 0.456 vs. 0.511): failure inspection blames the conflict-resolution policy, which can retain an earlier attribute value rather than the newer one when no explicit confidence score is available. The authors propose last-write-wins for factual and numeric attributes. This is a fixable bug, not a representational limit — worth separating from the surface-form finding.

### Experiment 2 — forgetting on a persistent 27,021-node graph

| Variant | Nodes | Edges | Size | F1 | Judge |
|---|---|---|---|---|---|
| No forgetting | 27,021 | 46,538 | 440.6 MB | 0.292 | 0.300 |
| Forgetting | 24,368 | 43,978 | 398.6 MB | 0.293 | 0.284 |
| **Change** | **−9.8%** (2,653 nodes) | −5.5% (2,560 edges) | **−9.5%** | **+0.001** | −0.016 |

**Paired bootstrap, Forgetting − No forgetting — nothing is significant:**

| Metric | Δ | 95% CI |
|---|---|---|
| F1 | +0.001 | [−0.015, +0.016] |
| Precision | −0.004 | [−0.023, +0.014] |
| Recall | −0.001 | [−0.017, +0.016] |
| Judge accuracy | −0.016 | [−0.038, +0.006] |

The judge drop of 1.6 points is the only one with a suggestive sign, and the interval **bounds any loss at 3.8 points**. That the pruner strips a tenth of the store for at most that is the paper's positive claim, and the authors are careful to call the non-significance of Experiment 2 "the key evidence for the no-quality-loss half of our main claim," while explicitly labelling the Experiment 1 comparison as "included for completeness but... not load-bearing for our main claim." That framing is worth noticing: the negative result on graph retrieval is presented as the *secondary* finding, though it is the more consequential one.

Absolute scores in Experiment 2 are much lower than Experiment 1 (F1 0.29 vs. 0.42) because merging 500 independent haystacks into one graph introduces **cross-conversation retrieval interference**. The experiment is a within-condition comparison, not an accuracy claim — a distinction the paper states outright.

Removed nodes are characterized as low re-reference frequency, limited structural connectivity, reduced recency: "the importance function preferentially removes peripheral information."

**Complexity.** Two exhaustive vector scans dominate — one at de-duplication `O(kNd)`, one at retrieval `O(qNd)` — giving per-turn `O(Nd + M)`. Forgetting is `O(N+M)` every 400 turns, amortized `O((N+M)/T)`. The authors point out the scan terms are incidental (an ANN index removes them), "leaving the forgetting module as the mechanism that bounds N itself." That is the honest architectural case for the pruner: it is the only component that bounds the thing that grows.

> [!warning] Contradiction
> This paper contradicts a claim currently made on the [[graph-memory]] concept page. That page states: *"The trade-off versus dense-vector memory is retrieval **simplicity** for relational **expressiveness** — graph traversals can answer multi-hop queries that flat embeddings struggle with, at the cost of a heavier ingest pipeline and more complex retrieval."*
>
> Two things are wrong with that framing in light of these measurements.
>
> **(a) The multi-hop advantage does not appear.** On LongMemEval's multi-session category — the category that *is* the multi-hop test — the graph scores F1 0.326 and judge 0.398 against the flat baseline's 0.342 and 0.436. It loses. The only category where the graph wins at all is temporal reasoning, on one of two metrics, by 1.5 points.
>
> **(b) The stated cost is understated.** The page prices graph memory at "a heavier ingest pipeline and more complex retrieval" — i.e. an *engineering* cost. This paper shows the cost is also an **information** cost: extraction irreversibly discards surface form, and that loss is worth 30 points of judged correctness on verbatim-recall questions (0.911 → 0.607). No amount of engineering effort recovers it, because the destruction happens at write time.
>
> **Scope of the refutation — be precise.** This refutes *extraction-based graph retrieval as a general default*, at this pipeline's fidelity. It does **not** refute [[hipporag-2]], whose design differs in exactly the way that matters: HippoRAG 2 keeps **passage nodes as first-class citizens** linked to extracted phrases, so the original text remains reachable and the surface form is never discarded. Reading these two papers together, the compatible synthesis is that the failure lies in *replacing* text with entities, not in *indexing* text with entities. The authors themselves also decline the general claim — one small extractor, one benchmark.
>
> **The forgetting half survives untouched, and points the other way.** Nothing here counts against graph-structured *retention*: the importance function pruned 9.8% of a 27,021-node store with no significant loss on any of four metrics. Whatever else is true of graph memory, the fact that a graph exposes degree centrality and incident-edge structure is what makes principled pruning available at all — a flat store has no centrality term to score with.

## Limitations

The paper's own limitations are unusually candid, in an appendix section addressed to reviewers:

- **Scope is bounded by budget, and stated as such.** Four complete runs, no sweep. Each configuration requires re-ingesting all 500 haystacks at one extraction call per turn plus one generation and one judge call per question.
- **No matched-compression control.** The authors name this as their top priority given more budget: prune the *same fraction of nodes at random* to isolate whether the four-term importance function beats a trivial retention rule. Without it, Experiment 2 shows that *removing 9.8% of this graph* is cheap, not that *this scoring function* is good. This is the most important caveat on the positive result and the paper does not hide it.
- **Retention parameters were never swept.** Weights, threshold s_min = 0.10, and the 400-turn interval were fixed a priori — the weights "encode a prior, drawn from the memory-stream and forgetting-curve literature" ([[generative-agents|Park et al. 2023]], [[memorybank|Zhong et al. 2024]]). Experiment 2 is therefore **one point on the compression–quality trade-off, not the curve**. The ~10% figure is an artifact of a conservative threshold, not a discovered optimum.
- **One extractor, one benchmark.** GPT-4o-mini is a small model; a stronger extractor might preserve more surface form. Priority list for future work is explicitly: random-pruning control, second benchmark, stronger extractor.
- **The judge is the extractor is the generator.** GPT-4o-mini plays all three roles. Not discussed in the paper, but it means judge bias cannot be separated from generator behavior.
- Unremarked by the authors: Experiment 2's absolute F1 of ~0.29 is low enough that a pruning-induced regression could be masked by floor effects. Non-significance at that operating point is weaker evidence than non-significance would be at Experiment 1's level.

## Related

- [[graph-memory]] — the concept this paper directly tests and partly refutes; see the Contradiction callout above.
- [[knowledge-graph]] — the representation under test: typed nodes, attributed edges, a nine-label fixed ontology.
- [[retrieval-augmented-generation]] — the flat vector baseline that wins here, matched at five retrieval candidates.
- [[dense-retrieval]] — the baseline's substrate; the paper's finding is that flat embedding retrieval keeps verbatim text that extraction throws away.
- [[longmemeval]] — the sole benchmark; 500 questions, ~33 months of timestamps, six categories. This is the first ingested source in the wiki to report full per-category LongMemEval numbers.
- [[long-term-memory]], [[memory-management]] — the problem domain: unbounded growth of a persistent conversational store.
- [[hipporag-2]] — the strongest counterexample in the wiki, and architecturally exempt: passage nodes remain first-class, so the surface form that killed this pipeline stays reachable.
- [[a-mem]], [[zettelkasten-memory]] — graph topology from LLM-chosen links rather than triple extraction; the same surface-form question applies but is untested here.
- [[mem0]] — cited as representative of graph memory that addresses consistency but not unbounded growth; this paper's stated gap.
- [[memory-evolution]] — A-MEM's rewrite-on-insert is the *update*-side answer to store growth; this paper's pruner is the *delete*-side answer. They are complementary and neither has been compared to the other.
- [[memory-consolidation]] — pruning-by-importance is a consolidation policy; the four-term score (recency, frequency, centrality, age) is its concrete instantiation.
- [[catastrophic-remembering]] — this is the same ratchet, and a rare paper that *measures the cost of deletion* rather than the cost of accumulation. It belongs in that concept's evidence table: 9.8% of nodes removed for at most 3.8 judge points.
- [[ebbinghaus-forgetting]] — the 90-day recency half-life is drawn from this lineage via [[memorybank]].
- [[memorybank]], [[memgpt]], [[generative-agents]], [[memory-stream]] — the flat-store predecessors the paper positions against; the memory-stream's recency/importance/relevance triple is the direct ancestor of its scoring function.
- [[memharness-memory-is-reconstructed-not-replayed-wu-2026]] — **the sharpest complement.** MemHarness argues verbatim replay is destructive and memory must be reconstructed; this paper finds the opposite failure, that abstraction destroys the verbatim detail some questions need. Both are right about different question types, and together they bracket the design space: neither raw replay nor pure abstraction is safe.
- [[zero-mem-zero-token-memory-operations-xiao-2026]] — independently reaches the same conclusion from the provenance angle, keeping original traces as the source of record and building only non-generative indexes over them. Zero-Mem is arguably the constructive version of this paper's negative finding.
- [[recuris-experiential-working-memory-evolution-yu-2026]] — contrast in retrieval key: Recuris selects experience against a verified task state rather than by embedding similarity over extracted entities.
- [[episodic-memory-llm]] — single-session (assistant) recall is an episodic demand, and it is exactly where extraction fails hardest.
- [[continual-learning]] — the paper's framing borrows selective retention from this literature (Kirkpatrick et al.); note the inversion, since here forgetting is the *goal*.
- [[llm-as-a-judge]] — binary judge correctness is one of the two headline metrics, and GPT-4o-mini judges its own generations.
- [[locomo]] — the sibling long-term-conversation benchmark, cited in related work but not evaluated; it is the obvious second benchmark the authors say they lack.
- [[llm-agent]] — the deployment setting: a personal assistant interacting over prolonged periods.

**Concept gaps noted (not created):** there is no page for *machine unlearning* / deliberate removal (cited here via Bourtoule et al.), none for *entity extraction* or *OpenIE* as a memory-write primitive, and none for *surface-form vs. abstraction fidelity* — the paper's actual contribution, which currently has no home. `toronto-metropolitan-university` has no entity page (`university-of-toronto` is a different institution and must not be substituted). FadeMem (Wei et al. 2026), the closest prior work on biologically-inspired agent forgetting, is not ingested.

## References

- [arXiv:2608.28978](https://arxiv.org/abs/2608.28978) — Rusu, Khanzadeh & Alalfi, *Selective Forgetting: A Graph-Based Memory Framework for Long-Term LLM Agents*, 29 Aug 2026, v1, cs.AI.
- Code: `github.com/skhanzad/Selective-Amnesia`
- Benchmark: LongMemEval (Wu et al., ICLR 2025) — not independently ingested in this wiki.
