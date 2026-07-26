---
title: "When is Routing Meaningful? Diversity and Robustness in Language Model Societies"
type: source
created: 2026-07-22
updated: 2026-07-22
sources:
  - when-is-routing-meaningful-huot-2026.md
arxiv_id: "2607.09197"
year: 2026
authors:
  - Fantine Huot
  - Michael Kaisers
  - Mirella Lapata
first_author: Fantine Huot
tags:
  - 2026
  - paper
status: complete
importance: medium
---

# When is Routing Meaningful? Diversity and Robustness in Language Model Societies

**Fantine Huot, Michael Kaisers, Mirella Lapata** (Google DeepMind) — [arXiv:2607.09197](https://arxiv.org/abs/2607.09197). 19 pages, cs.MA, submitted 10 July 2026; no venue listed.

## Summary

Routing among multiple models or agents is almost always evaluated on task accuracy and inference cost. This paper argues those metrics are insufficient because a router is not merely a predictor of task performance — it is the **coordination mechanism** that determines how work is distributed across a society of actors. Two structural properties, orthogonal to accuracy, determine whether routing is meaningful at all. First, the society must be **behaviourally differentiated**: if all actors respond identically, routing is vacuous no matter how good the policy is. Second, the policy must be **stable under surface-form variation**: semantically equivalent query variants must go to the same actor, otherwise actors receive inconsistent query distributions and specialisation cannot emerge. High task accuracy is compatible with violating both.

To measure the first, the authors adapt **Hierarchic Social Entropy** (HSE; Balch 2000, originally for robot swarms) to language-model societies. Each actor `r_k` is represented by a **behavioural vector** `b_k` of its scores on an evaluation set — measured from *outputs*, not weights, so it extends to agents sharing a backbone but differing in system prompt or tool access. Inter-actor distance is `1 − cosine similarity` (chosen over the original HSE's Euclidean distance because behavioural vectors differ in magnitude when actors differ in overall accuracy, and over SND's Wasserstein distance because full action distributions aren't available for LM outputs). Because simple Shannon entropy over cluster proportions is blind to *how far apart* clusters are — two societies with three identical actors and one outlier both score H = 0.811 whether the outlier is near or far — HSE integrates entropy across all single-linkage clustering thresholds (taxonomic levels). To measure the second property, they introduce a **perturbation-based robustness metric** `ρ`: the fraction of a query's perturbations routed to the same actor as the original, over a five-level taxonomy (character-level typos/spacing/punctuation; word-level synonym substitution; syntax-level passive voice/clause reordering; full paraphrase; rambling with added irrelevant context). Boundary anchors: a fixed router that always picks one actor gives `ρ = 1`; a uniform random router gives `E[ρ] = 1/N`.

The headline empirical result is that **accuracy and meaningfulness sharply diverge**. Learned KNN routers achieve their *best* accuracy on the most differentiated (specialist) societies and simultaneously their *worst* robustness — perturbed-query accuracy collapses to near-random at every specialist society. The prompted (LLM-based) router is less accurate on clean queries but far more stable, with robustness rising *monotonically* with society diversity in a way no KNN router does. The authors' explanation: the sharp behavioural boundaries that make specialist societies easy to route correctly on clean queries make them equally easy to route *incorrectly* when a surface reformulation shifts the query embedding across a domain boundary; prompted routing latches onto semantic intent rather than embedding position.

## Key points

- **Setup**: societies `R` of `N` actors, queries `Q`, routing policy `π: Q → R`. Actor `r_k` gets behavioural vector `b_k = (s(r_k, e_1), …, s(r_k, e_L)) ∈ R^L` over evaluation set `E`; stacking gives behavioural matrix `B ∈ R^{N×L}`. `HSE(R)` depends on the society alone (policy-independent); `ρ(π, Q)` depends on the policy.
- **Benchmarks**: EmbedLLM (112 models, 10 reasoning/knowledge benchmarks incl. MMLU, GSM8K, GPQA, ASDiv; 36,054 questions) and RouterBench (11 models, 36,497 queries from 8 datasets).
- **Societies compared on EmbedLLM** (HSE in parentheses): default real-world model pool (**1.62**); RD = Reasoning Depth, five non-overlapping experts across tiers of cognitive complexity (**2.32**); RD+SA combined 20-expert pool with partially overlapping expertise (**3.19**); SA = Subject Area, 15 non-overlapping domain experts (**3.91**). Synthetic societies use binary performance indicators, so oracle accuracy is 1.0 by construction.
- **Real-world pools are less diverse than assumed**: the 15-expert SA society is **more than twice as diverse** as the 112-model default pool. Size does not imply diversity — many real models occupy similar regions of behavioural space.
- **Strong diminishing returns / coreset heuristic**: normalised HSE plateaus after **approximately nine agents** on EmbedLLM's default society (the abstract states fewer than ten agents recover most available diversity; four agents on RouterBench). Synthetic specialist societies plateau even faster, at four to five agents, because their profiles are maximally orthogonal by construction.
- **HSE validation**: identical actors → HSE = 0 (routing vacuous) regardless of society size; fully orthogonal actors → normalised HSE = 1.0, and the normalised value is invariant to society size, enabling comparisons across scales. HSE is a ratio measure with absolute zero, so "Society A is twice as diverse as B" is well-formed.
- **KNN routers collapse**: task accuracy rises sharply and monotonically with HSE for KNN-1/3/10, but perturbed-query accuracy falls to near-random at every specialist society, and `ρ` stays low across the entire HSE range.
- **Prompted router**: clean accuracy on SA is lower than KNN-10, but the clean-vs-perturbed accuracy gap is **less than five points**, and robustness tracks HSE monotonically.
- **Max-HSE subsets**: switching from the full pool to the greedily selected max-HSE subset roughly **doubles** the prompted router's robustness on EmbedLLM and improves it more modestly on RouterBench. For KNN the effect depends on pool structure — on RouterBench it dramatically rescues KNN robustness at all `k`; on EmbedLLM, KNN-1 robustness actually *decreases* while KNN-3/KNN-10 improve only marginally, because the EmbedLLM pool has less structured behavioural separation even at max HSE.
- **Robustness is roughly flat across perturbation levels** — an important nuance. On RouterBench the prompted router ranges only from 0.589 (paraphrase) to 0.617 (syntax) on the full society, and 0.663 (paraphrase) to 0.744 (word) on the max-HSE subset. No router shows consistent decline as perturbations get semantically deeper; routers are no more sensitive to a full paraphrase than to a character-level typo, implying routing decisions are *not* anchored to surface features. The only hint of level-sensitivity is the prompted router's paraphrase dip on the max-HSE subset.
- **Stated limitations**: HSE is computed over a fixed evaluation set `E` and is misleading if `E` is unrepresentative; max-HSE subsets are selected greedily (true optimum needs exponential search); synthetic societies use binary indicators producing maximally sharp boundaries and oracle accuracy 1.0, so real specialist agents will have softer profiles and the observed KNN brittleness **may be attenuated in practice**. Robustness is measured at the assignment level on single-turn queries; extending to multi-step agentic settings, where perturbations propagate through state, is left as future work — as is using HSE as a *training signal* to bootstrap diverse societies from homogeneous initialisation.

## Concepts & entities

- [[learned-orchestration]] — the paper's direct target: routing policies evaluated as coordination mechanisms rather than accuracy predictors.
- [[mixture-of-experts]] — the "society of behaviourally differentiated actors" premise is the multi-model analogue of expert specialisation; the diminishing-returns result is a coreset heuristic for how many experts are worth maintaining.
- [[agent-evaluation]] — the core methodological claim: two structural axes (diversity, stability) should join accuracy and cost when evaluating routers.
- [[skill-acquisition]] — stable query assignment is framed as the *necessary condition* for specialisation to emerge (borrowed from multi-robot role-stability literature); unstable routing makes specialisation impossible by construction.
- [[self-improving-agent]] — relevant to the future-work proposal of using HSE as a training signal to bootstrap diverse societies from a homogeneous initialisation.

## References

- [arXiv:2607.09197](https://arxiv.org/abs/2607.09197) — "When is Routing Meaningful? Diversity and Robustness in Language Model Societies", Fantine Huot, Michael Kaisers, Mirella Lapata, Google DeepMind, 2026.
- Balch (2000), "Hierarchic social entropy: An information theoretic measure of robot group diversity", *Autonomous Robots* 8(3):209–238 — the metric adapted here.
