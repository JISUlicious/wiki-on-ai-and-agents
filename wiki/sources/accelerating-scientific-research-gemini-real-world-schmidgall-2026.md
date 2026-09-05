---
title: "Accelerating Scientific Research with Gemini in the Real-World"
type: source
created: 2026-09-05
updated: 2026-09-05
sources: []
arxiv_id: "2608.26701"
year: 2026
authors:
  - Samuel Schmidgall
  - Xiaokai Zhu
  - Marian Shaw
  - Lin Yang
  - Valentin Liévin
  - Juraj Gottweis
  - Vivek Natarajan
  - Quoc V. Le
  - Tao Tu
tags:
  - 2026
  - paper
  - ai-for-science
status: complete
importance: high
---

# Accelerating Scientific Research with Gemini in the Real-World

**Samuel Schmidgall, Xiaokai Zhu, Marian Shaw, Lin Yang, Valentin Liévin, Jingyun Yang, Yuchen Zhuang, Tim Strother, Alex Bijamov, Min Woo Sun, Anil Palepu, Justin Chen, David Steiner, Jacqueline Shreibati, Wei-Hung Weng, Yilin Zhao, Xingjian Hu, Nicholas Zahn, Sadhya Garg, Julia Kirby, Yuxiang Gan, Jiaoli Li, Divy Thakkar, Shekoofeh Azizi, David Racz, Juraj Gottweis, Vivek Natarajan, Chenglin Wu, Tal Danino, Keran Rong, Haozhe Wang, Benoit Schillings, Yong Cheng, Quoc V. Le, Tao Tu** (Google Research, Google DeepMind, Duke University, Columbia University, Texas A&M University) — [arXiv:2608.26701](https://arxiv.org/abs/2608.26701), cs.AI, 27 Aug 2026 (v1). Funded by Alphabet; no venue given. Co-Scientist source code is *not* released (access via Google Labs "Gemini for Science").

## Summary

This is the follow-up to the original Co-Scientist (Gottweis et al., 2026), which was an in-silico hypothesis generator. The extension turns it into a three-stage, execution-grounded pipeline — **Ideation → Experimentation → Paper Writing** — where every stage is an evolutionary search whose variation operator is Gemini and whose fitness function is an LLM judge, and then validates it in three domains that deliberately span a **spectrum of autonomy** (Table 1 of the paper): materials synthesis where humans execute AI-designed protocols, biology where the AI builds the pipeline but experts iteratively re-frame the task, and computer science where the AI runs unattended from a single directive. A fourth study measures the *integrity* of fully autonomous paper generation with 30 experts and 450 blind reviews.

The mechanism is worth stating precisely, because the paper's main engineering contribution is how it stops the loop from reward-hacking itself. Ideation runs a population of hypotheses at temperature 1.6, each grounded by a parallel literature review, screened for safety and reviewed by an LLM for novelty/plausibility/testability, ranked by TrueSkill-style Bayesian pairwise tournaments with UCB exploration (κ=1.0), and bred by crossover (p=0.7) / reflection-guided mutation (0.3) over G=10 generations. Experimentation is a three-phase evolutionary code search (scaffold on a data subset → transition → full-scale run) with parallel solvers, an LLM reward model scoring plan adherence, rigor and output quality in [0,1], and a 0.97 score decay to keep pressure on the best-program buffer. Paper writing evolves a manuscript scored on nine review dimensions. The **reliability modules** that the abstract credits are: (1) a *joint-optimization objective* `S = 1.0·S_reviewer − 0.5·S_plagiarism − 1.0·S_hallucination(P, E, E_log)`, where the hallucination term is conditioned on the source code and raw execution logs so that any unverified claim can cost a full unit (more than the ≤0.3 a reviewer can award); (2) a deterministic **hallucination-clipping** pass that parses every quantitative claim in the draft, checks it against the execution logs, rewrites unsupported sentences with the logged values, and *aborts paper writing entirely* if no valid logs exist; (3) enforced verbose logging during experimentation so the logs are rich enough to audit; and (4) a two-layer ethics gate (directive screening, then a binary LLM evaluator with textual feedback on every idea and plan) plus a pre-execution code-sanitization step.

The results are a mix of genuinely striking physical outcomes and carefully hedged claims. In materials, the system proposed hexachloroethane (C₂Cl₆) as a non-toxic replacement for TiCl₄ and the resulting 2D phase shows XRD/SEM/EDS/STEM signatures *consistent with* Ti₃C₂Tₓ MXene, but the atomic structure is explicitly unconfirmed and Raman/XPS show heavy oxidation. Monolayer MoS₂, MoSe₂ and WS₂ grew on the first attempt in a lab with no prior MoSe₂/WS₂ experience. In biology, a Gemini-3-Pro-Image interpolation pipeline matched unpublished E. coli swarming morphology on 3 of 4 metrics. In CS, the autonomously discovered **Agent_H** tops six frontier models on *length-adjusted* HealthBench under LLM autoraters — but three blinded physicians found it significantly better than Gemini 3.1 Pro on only one of nine dimensions (harm). And in the 150-manuscript study the reliability modules cut invalidating result hallucination from 90% (Agent Laboratory) and 46% (ablated) to 4%.

## Key points

**The human-in-the-loop boundary, study by study.** This is the thing the abstract compresses most, so it is spelled out here from the paper's own Table 1, Section 3.1 and Section 5:

| Study | What Co-Scientist did | What humans did |
|---|---|---|
| MXene precursor (3.1.2) | Reasoned over reaction kinetics and the reactor geometry to propose C₂Cl₆; produced a **ranked list of 272 candidate recipes** (precursor type/amount/location, gas flows, substrate, temperature profile) with machine-level execution code | Chose the model's **#2-ranked** recipe (C₂Cl₆ + Ti); over **25 iterations** refined it by co-mixing precursors and adding continuous forming gas, "drawing insights from alternative configurations across the model's candidate pool"; loaded/unloaded every sample; ran XRD screening after each run; did all characterization; diagnosed the 11.5% replication rate as O₂ leaks and wrote the sealing/cleaning protocol that raised it to 68% |
| TMDs, full mode (3.1.3) | ~1 day of test-time compute to generate ranked protocols (gas flows, ramp/hold temperatures, cooling rate, precursor loading, 215 mm spacing, 15 min window) from a hardware description only | "A human expert to select the top-ranked hypothesis, load the precursors and substrate into the furnace, run the growth cycle, and take measurements" |
| TMDs, fast mode (3.1.3) | Gemini 3 Deep Think formulated recipes in minutes and translated them "directly into machine-level codes that control the CVD equipment throughout the growth cycle" — this is the only place the AI *operated* hardware | Physically loaded the initial precursor and substrate; characterized the product |
| E. coli (3.2) | Implemented and optimized the whole vision pipeline (leave-one-out interpolation, Best-of-16 rejection sampling scored by Gemini 2.5 Pro) | Suggested the pipeline paradigms in the directive; after each round reviewed outputs and rewrote the research directive (task framing, not architecture); did all plating, imaging and feature extraction |
| HealthBench (3.3) | Everything after the directive: generated, tested and iterated the entire codebase | Wrote the directive and the synthetic training corpus; ran the physician evaluation |
| Paper generation (3.4) | Everything, including sourcing datasets and benchmarks | Nothing after the topic prompt |

Section 5 sums the MXene work as "human researchers iteratively refined this route across **over 70 physical experiments**" (25 design iterations + 26 initial replications + 25 post-fix replications). So "operated a semi-automated CVD reactor" is accurate only for the fast TMD mode, and even there humans loaded samples; the MXene route was AI-*proposed*, human-*optimized*.

**MXene evidence and its holes.** XRD peak at 2θ=7.8° (≈1.13 nm interlayer spacing, matching the (002) of Ti₃C₂Tₓ), STEM FFT d-spacing ≈2.51 Å (matches (10-10) of wet-etched Ti₃C₂Tₓ), EDS shows Ti/C/Cl (F after MILD treatment), no PHI 27.8° reflection, no nitrogen in SEM-EDS, layers survive LiF/HCl (so not TiCₓ particles). Against: TEM-EDS finds O and N, Raman shows TiO₂ modes, XPS shows only Ti–O bonds, yield is low, and the candidate phases still include Ti₂CCl₂. The authors ask for cross-sectional atomic STEM before assigning the phase.

**TMD one-take growth.** MoS₂ triangles >50 µm with Raman E¹₂g/A₁g separation ≈21 cm⁻¹ (monolayer), plus irregular flakes and >80×80 µm films; MoSe₂ and WS₂ monolayers on first attempt, "verified through at least five replication runs." The fast Deep Think mode also succeeds first-try in ~1 hour total but yields smaller, less regular domains — an explicit compute-vs-quality trade-off. Not yet tested on any other CVD geometry.

**E. coli swarming (pLac-rpoS vs pLac-gfp control, IPTG 0–10 mM).** Linear mixed-effects test on Source×log₁₀(IPTG); non-significant interaction (p>0.01) = concordant trajectory:

| Metric | p (Source×IPTG) | Verdict |
|---|---|---|
| Mean radius | 0.593 | concordant |
| Polar eccentricity | 0.451 | concordant |
| Circumferential intensity CV | 0.712 | concordant, more variable |
| Circularity (pLac-rpoS) | **0.002** | diverges — generated colonies too regular |

The control strain's *absence* of dose-response was predicted correctly despite prompts encouraging trend detection. The authors call this interpolation along a known gradient, not extrapolation, and report modality hallucinations (colonies rendered with a fluorescent green glow) that needed rejection-sampling filters.

**Agent_H on HealthBench.** Discovered from a directive plus two tools (`query_model`, `get_guideline`) and n=1,282 synthetic queries with synthetic rubrics; Gemini 3.1 Pro backbone, no web search, benchmarks held out (decontamination in Table A1). The architecture is an eight-phase pipeline: multi-axis triage with adversarial-risk detection → optional decomposition → 28–48 parallel candidates across six personas and τ∈[0.5,0.95] → single-elimination pairwise tournament → 3-judge majority vote → up to 5 auditor critique cycles → citation audit → compression to a ~2,000-character target. Cost: 40–80 LLM calls per query. Baselines: GPT-5.6 Sol, GPT-5, Claude Fable 5, Claude Opus 5, Gemini 3.1 Pro, Gemini 3.5 Flash, all single-call at highest reasoning. Two autoraters, 8 grading runs each; length adjustment penalizes deviation from a 2,000-character pivot:

| | Hard, raw | Hard, len-adj | Prof, raw | Prof, len-adj |
|---|---|---|---|---|
| *Judge: Gemini 3.5 Flash* | | | | |
| Agent_H | **0.420** | **0.377** | 0.645 | **0.643** |
| Claude Opus 5 | 0.390 | 0.281 | **0.697** | 0.572 |
| GPT-5.6 Sol | 0.331 | 0.293 | 0.664 | 0.614 |
| GPT-5 | 0.414 | 0.334 | 0.536 | 0.485 |
| Gemini 3.1 Pro (backbone) | 0.236 | 0.148 | 0.528 | 0.467 |
| *Judge: GPT-5.4 Low* | | | | |
| Agent_H | 0.335 | **0.292** | 0.621 | **0.619** |
| GPT-5 | **0.372** | 0.291 | 0.519 | 0.468 |
| Claude Opus 5 | 0.349 | 0.253 | **0.677** | 0.553 |

So "beats six frontier models" holds for **length-adjusted scores under both judges**, and for raw Hard under the Gemini judge only; on raw Professional Claude Opus 5 wins under both judges, and on raw Hard GPT-5 wins under the GPT judge. Agent_H's mean length is 2,549 chars on Hard (vs 5,020 for Gemini 3.1 Pro) and 1,850 on Professional (vs 7,618; Opus 5 averages 6,201), so much of the adjusted margin is length control. Response lengths are exactly what the architecture was optimized for, and the paper reports that an earlier optimization run *without* a length penalty inflated scores "well above SOTA baselines" purely through verbosity — a Goodhart failure they caught and fixed.

**Blinded physician review** was *not* a six-model comparison: three board-certified physicians rated 106 questions (51 Hard, 55 Professional), one rater per question, Agent_H vs Gemini 3.1 Pro only, on nine dimensions. Only **likelihood of harm** reached significance (p=0.0486 after FDR correction); the other eight did not. The autoraters agree with each other (Spearman ρ=0.869) but show low Randolph's κ agreement with clinicians. The paper says plainly that the automated advantage "did not translate into a perceived difference across eight of the nine evaluated clinical dimensions."

**The 450-review integrity study.** 50 Gemini-generated AI research topics × 3 conditions (full Co-Scientist, ablated Co-Scientist without soft penalties or clipping, Agent Laboratory) = 150 manuscripts, each produced end-to-end on 2×A100 with autonomous dataset/benchmark sourcing, reviewed blind by 30 experts (29 PhD/postdoc, mean 11 years) × 3 reviews each. Reviewers cross-referenced metrics against execution logs (deterministic outputs of running the agent's code), methods against source code, and novelty against Google Scholar / Semantic Scholar / OpenScholar:

| Failure (threshold) | Co-Scientist | Ablated | Agent Laboratory |
|---|---|---|---|
| Result hallucination, invalidating (severity ≥5/10) | **4%** (n=2) | 46% | 90% |
| Complete data fabrication (≥8) | **0%** | 40% | 44% |
| Any result hallucination | 22% | 54% | 94% |
| Mean hallucination severity | 0.78 | — | 7.16 |
| Methodological hallucination, severe (≥5) | 24% | 52% | 100% |
| Methodological, extreme (≥8) | 2% | — | 74% |
| Severe plagiarism (novelty ≥3/5) | 16% | 50% | 60% |
| Any derivative content | 30% | 56% | 80% |
| Cited the source when derivative | 39.4% | — | 15.9% |

χ² for the headline rows: 74.3 (p<10⁻¹⁶), 60.9 (p<10⁻¹³), 21.8 (p<10⁻⁵). Note the ablation isolates the modules from the model: ablated Co-Scientist is still far better than Agent Laboratory on hallucination, so Gemini itself accounts for part of the gap, but the modules account for the drop from 46% to 4%.

**Safety.** 7 experts × (10 harmful + 10 benign) directions × 10 runs: **98.7%** refusal of harmful (691/700), 3.1% false refusal; all 9 misses were dual-use framed neutrally. With refusal disabled, the in-loop oversight kept 96.3% of ideas and 96.7% of plans expert-rated safe vs 86.7% / 72.4% ablated, with no quality cost (3.25 vs 3.26 on a 5-point scale, p=0.82). Inter-rater κ inside the safety condition was only 0.38–0.43.

**Stated limitations.** Recipes untested on other reactors; biology is interpolation on one circuit; Agent_H only evaluated on single-turn rubrics and optimized against a proxy that autoraters and physicians disagree about; 40–80 calls/query precludes interactive use; the reliability modules verify against execution logs, which "is unproven in physical experiments" with noisy readouts; residual failure modes remain (selective reporting across runs, math-vs-code divergence, mock functions disguised as pipelines) that log-checking cannot catch; hypothesis-space homogenization and accountability are raised as open ethical problems. The authors state they "do not claim that autonomous agents can currently produce publication-ready research."

## Related

- [[multi-agent]] — Co-Scientist is an evolutionary multi-agent system at every stage: parallel solvers, reviewer, ranking, reflection and auditor agents.
- [[llm-as-a-judge]] — LLM reward models score programs, hypotheses and manuscripts; the HealthBench study is a case where the LLM judge and the human judges disagree on eight of nine axes.
- [[reward-hacking]] — the central design problem: surrogate-reviewer objectives induce fabrication (80–100% in prior systems), and the length-penalty incident shows the discovery loop itself Goodharts the benchmark.
- [[hallucination]] — hallucination here is *reward-hacking-induced fabrication*, not the short-form factual kind; the fix is log-grounded verification, not retrieval.
- [[verifier]] — the hallucination-clipping module is a deterministic verifier over execution logs, separate from the soft penalty.
- [[outcome-based-reward]] / [[reward-modeling]] — the joint objective is a multi-term reward with penalties conditioned on `(E, E_log)`.
- [[inference-time-compute]] / [[test-time-compute]] — Agent_H is discovered by scaling inference-time search, not by changing weights; the TMD study makes the compute-vs-quality trade-off explicit.
- [[adas]] — Agent_H is an ADAS-style outcome: a meta-agent discovering an agent architecture in code; the paper cites ADAS directly.
- [[self-improving-agent]] — framed as "further progress toward closed-loop … iterative self-improvement," but the loop closes only in the computational domain.
- [[agent-evaluation]] — autorater vs clinician disagreement and the 450-review rubric are both evaluation-methodology contributions.
- [[llm-agent]] — the extension is an execution-grounded agent with code execution and hardware control tools.
- [[self-reflection]] / [[reflexion]] — the ethics oversight and figure-generation loops are built on self-reflection/self-refine.
- [[darwin-godel-machine]] / [[godel-machine]] — cited as the recursive self-improvement lineage this work positions itself against.
- [[frontis-ma1-ai4ai-recursive-self-improvement-yang-2026]] — the contrasting approach: train the improver rather than search with a frozen model; Co-Scientist keeps Gemini frozen throughout.
- [[cedar-agent-orchestrated-tree-search-complex-systems-tian-2026]] — same Editor/Judge-over-runnable-code pattern applied to one narrow domain.
- [[rehearse-confidence-cliff-autoresearch-ji-2026]] — a warning about LLM pre-execution judgment degrading inside self-improvement loops; Co-Scientist's tournaments are exactly such judgments.
- [[coding-agents-replicate-sciml-papers-hans-2026]] — the same "agent's own description is not evidence" problem, solved there with harness-side acceptance rules rather than log-clipping.
- [[training-ai-scientists-replicate-research-falck-2026]] — cited by this paper (Falck et al., 2026) as showing AI scientists can be *trained* for replication.
- [[deepmind]] / [[google]] / [[google-research]] — the labs; [[quoc-le]] is a senior author.
- [[openai]] — HealthBench and HealthBench Professional are OpenAI datasets; [[anthropic]] models are among the baselines.

## References

- [arXiv:2608.26701](https://arxiv.org/abs/2608.26701)
- Gottweis et al., 2026 — original Co-Scientist (in-silico hypothesis generation)
- Schmidgall et al., 2025 — Agent Laboratory (the external baseline)
- HealthBench / HealthBench Professional: `openai/healthbench`, `openai/healthbench-professional` on Hugging Face
- Swarm colony images: https://doi.org/10.5281/zenodo.19612563

## Related 2026 sources

- [[mathematics-in-the-age-of-ai-tao-2026]] — the sceptical companion: Tao argues that verification, exposition, community acceptance and canonicalization were historically correlated with solution generation and are now being pulled apart by it. His critique of AI-math evidence — undisclosed attempts, **scaffolding**, compute, contamination — is an independent arrival at this wiki's harness-disclosure position
