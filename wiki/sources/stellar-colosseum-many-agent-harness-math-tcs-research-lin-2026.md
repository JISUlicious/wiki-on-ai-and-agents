---
title: "Stellar Colosseum: A Many-Agent Harness for Long-Horizon Research in Mathematics and Theoretical Computer Science"
type: source
created: 2026-09-30
updated: 2026-09-30
sources:
  - stellar-colosseum-many-agent-harness-math-tcs-research-lin-2026.md
arxiv_id: "2609.15983"
authors:
  - Honghao Lin
  - David P. Woodruff
  - Yuan Deng
  - Jieming Mao
  - Song Zuo
  - Vahab Mirrokni
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# Stellar Colosseum: A Many-Agent Harness for Long-Horizon Research in Mathematics and TCS

**Honghao Lin\*, David P. Woodruff\*, Yuan Deng, Jieming Mao, Song Zuo, Vahab Mirrokni** ([[google-research|Google Research]]; [[cmu|CMU]]) — [arXiv:2609.15983](https://arxiv.org/abs/2609.15983), v2, cs.AI, 15 Sep 2026. Preprint; no venue. Integrated into Google Antigravity's Teamwork framework as the "Long Proof" pattern.

## Summary

A model-agnostic harness that allocates inference across a **five-stage research pipeline**: strategy exploration → a *readiness gate* that decides when a route is concrete enough → decomposition into a sectioned proof with an explicit dependency graph → parallel subproblem solving with local retries → global verification. A rejected proof either gets patched section by section or reopens exploration. Inside any hard stage the same pattern repeats. Colosseum generates a population of candidates, attacks each with a targeted **falsifier**, and reduces them through **overlapping random-sample tree aggregation**: each synthesis node merges a random subset of candidates *together with their critiques*, so objections survive aggregation. The exploration tree for TCS-Bench is (32, 16, 8, 5, 1) wide with fan-in 5. The verifier aggregation "is not a majority vote: a concrete fatal defect is sufficient to reject the proof". A shared knowledge directory carries partial results and failed attempts across rounds.

It extends the parallel-exploration / iterative-verification setup of Woodruff et al., *Accelerating scientific research with Gemini: case studies and common techniques* (arXiv:2602.03837, not ingested). That paper should not be confused with the similarly titled Co-Scientist report [[accelerating-scientific-research-gemini-real-world-schmidgall-2026]].

## Key points

- **TCS-Bench** (300 theorem-proving tasks from FOCS/STOC/SODA 2020–2026; graded by a reference-assisted automated grader reported at >90% accuracy on 100 expert-labeled proofs):

| Method | Accuracy |
|---|---|
| Gemini 3.1 Pro (direct) | 30.3% |
| Gemini 3.1 DeepThink | 52.0% |
| GPT-5.6 Pro (max) | 68.0% |
| Colosseum + Gemini 3.1 Pro | 54.0% |
| Colosseum + Gemini 3.7 Flash | 55.0% |
| **Cross-model selection** (8 Flash critiques of the Pro proof; submit it if ≥5 say correct) | **71.0%** |
| Oracle best-of-two (upper bound) | 77.3% |

  The headline 71.0% is **two full Colosseum runs plus a critique selector**. Either run alone scores 54–55%, below GPT-5.6 Pro used directly. The critique signal separates correct from incorrect proofs at AUC 0.896.
- **Codeforces** (222 problems rated above 1500, contests 2084–2162, full hidden tests): **218/222 with an execution probe** (corpus-level rating 4263) and 213/222 without (3918). The probe only compiles and stress-tests on public and generated inputs.
- **Research contributions** (proofs are in companion papers, not this one): a strong coreset for ℓp subspace approximation (p > 2) that improves the ε-dependence from ε⁻ᵖ to ε⁻²; a conditional hardness barrier for sparse least squares; dimension lower bounds for maximum-inner-product embeddings; a single-stage Hadamard quantizer; and a near-optimal Ω(log^{3/2} n/(log log n)^{3/2}) lower bound for prefix-matrix factorization.
- **Case studies.** A 46-page and a 75-page proof *draft* for Knuth's even-m cycles. With internet off, a 22-page draft that independently reaches the "central architecture" of OpenAI's Erdős unit-distance counterexample over 15 exploration rounds.

## What it does not show

- **No matched-compute search baseline.** The paper cites fixed-budget comparisons (REBASE) but does not run one. There is no best-of-N or self-consistency arm for Gemini 3.1 Pro at Colosseum's call count, and the call count is never reported ("also depends on the number of proof sections, local retries, and global revision rounds"). The DeepThink and GPT-5.6 Pro rows differ in both compute and model.
- The grader is an automated LLM grader from TCS-Bench, a benchmark built by an overlapping author group (Lin, Woodruff, Mirrokni). The case-study drafts are not reported as refereed or formally checked, and the level of human involvement in the research results is left to the companion papers.

## Concepts & entities

- [[multi-agent]] — "many-agent" is used here in the sense of many parallel candidates and falsifiers, not persistent agents with identity.
- [[verifier]] · [[llm-as-a-judge]] — falsifiers and the 8-critique selector. The critique-routing AUC is the most reusable number here.
- [[inference-time-compute]] · [[self-consistency]] · [[tree-of-thoughts]] — the test-time-scaling family Colosseum structures. It explicitly rejects flat voting.
- [[agentic-harness-engineering]] — a hand-designed, not evolved, harness; relevant to the harness-vs-search question only by counter-example.
- [[leap-formal-mathematics-agentic-kung-2026]] · [[numina-lean-agent-liu-2026]] · [[mathematics-in-the-age-of-ai-tao-2026]] — adjacent math-agent work. Colosseum works in natural language, and formal checks are optional evidence.
- [[google-research]] · [[cmu]].

## References

- [arXiv:2609.15983](https://arxiv.org/abs/2609.15983)
- TCS-Bench: Cohen-Addad et al., arXiv:2608.09538.
