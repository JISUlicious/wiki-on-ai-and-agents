---
title: "The Last AI Built by Humans: Toward Genuine Recursive Self-Improvement"
type: source
created: 2026-09-15
updated: 2026-09-15
sources:
  - last-ai-built-by-humans-recursive-self-improvement-duan-2026.md
arxiv_id: "2609.11873"
authors:
  - Yi Duan
  - Ying Liu
  - Zirui Tang
  - Haodong Chen
  - Jun Zhou
  - Zhiyuan Liu
  - Xuanhe Zhou
  - Fan Wu
  - "(et al., 34 authors)"
year: 2026
status: complete
importance: high
tags:
  - 2026
  - paper
---

# The Last AI Built by Humans: Toward Genuine Recursive Self-Improvement

**Duan, Liu, Tang, Chen, Zhou (equal contribution), … , Xuanhe Zhou (corresponding), Fan Wu** (34 authors). Affiliations: Shanghai Jiao Tong University, Theseus Labs, [[tsinghua-university]], ByteDance, ModelBest, Xiaohongshu, Humanlaya, Agent-Native Research Lab, Shanghai AI Lab. [arXiv:2609.11873](https://arxiv.org/abs/2609.11873), cs.LG, v1 10 Sep 2026. It is a survey with no venue stated. Project page: `theseus-labs-rsi.github.io`.

## Summary

The survey defines recursive self-improvement (RSI) as turning experience into **persistent changes to the system itself** (weights, harness, or improvement policy). Those changes must also be able to affect how later improvements are *generated, evaluated, selected and consolidated*. Its unit of analysis is the **improvement loop**, not an algorithm. Every system is asked three questions: *where does the loop close*, *what is updated and inherited*, and *which decisions remain external*. The answers place the system on a scale that measures **how much responsibility for improvement has moved from humans to the AI**. The survey stresses that this is not a measure of how good the improvement is: "a higher autonomy level does not by itself imply a better improvement process." The rest of the survey applies the scale to science, embodied AI, software engineering and healthcare, then to six industry case studies. Four of those cases are organizations on the author list (Theseus, Humanlaya, ModelBest, Agent-Native Research Lab), and a fifth, Lark, is a product of ByteDance, which is also an author affiliation.

## The autonomy stages

| Level | What the AI newly controls | Example given |
|---|---|---|
| **B0** In-task improvement (*not RSI*) | Revises its current output; nothing persists past the task | Self-Refine, [[reflexion]], Tree of Thoughts |
| **L1** Improvement-execution autonomy | Executes an update whose *what*, *how* and success criterion are set by humans | FineWeb-Edu quality labelling |
| **L2** Improvement-strategy autonomy | Diagnoses weaknesses and chooses *how* to improve; objective and evaluator stay fixed | Self-Harness; Karpathy's autoresearch |
| **L3** Learning-signal / experience-acquisition autonomy | Chooses *what experience* to acquire next, conditioned on the learner | SIMA 2, AZR, R-Zero |
| **L4** Environment-adaptation autonomy | Revises persistent state from deployment interaction, under external acceptance rules | PANDO |
| **L5** Recursive-inheritance (meta-improvement) autonomy | Persistently revises the *mechanism* that governs later improvement: improver, verifier, or successor generation | A-Evolve-Training |

Each step up the scale adds one condition. B0 to L1 adds **persistence**. L1 to L2 adds **strategy selection**. L2 to L3 adds control of the **learning agenda**. L3 to L4 adds **deployment**. L4 to L5 adds **recursive inheritance**. L5 is split in two:

- **Structural L5**: the revised mechanism persists and actually controls a later round.
- **Effective L5**: the revised mechanism produces better successors under matched budgets and independent assessment.

The survey calls effective L5 "a substantive empirical gap". Where it places familiar systems:

- [[darwin-godel-machine]] is a "transition case", because archive management and parent selection stay outside self-modification. Its appendix table tags DGM "L5 cand.", meaning plausible but not fully demonstrated.
- [[hyperagents-zhang-2026|HyperAgents]] transfers to unseen tasks, but its 200-iteration run found no significant final advantage.
- Gödel Agent ended below its starting policy in 14 of 100 MGSM trials.
- Weco's AIDE2 had 7 accepted improvements in 100 steps, but showed no significant efficiency gain when used as the outer improver.
- [[red-queen-godel-machine-iacob-2026|RQGM]] (co-evolving evaluator, frozen within each epoch): 71.7% vs 69.9% on held-out Polyglot.
- A-Evolve-Training on a 30B Nemotron: 0.80 → 0.86 against 0.87 for the top human submission. The survey reads this as "policy-level L5 within a human-defined objective".
- AlphaEvolve is tagged **B0**.

## Headroom-Closed Index (HCI)

HCI puts heterogeneous benchmarks on a common scale. **H = 100 × (s̄ − F₀) / (100 − F₀)**, where s̄ is a source-weighted consensus score and F₀ is the **90th-percentile model score in the benchmark's first year** in the dataset. So 0 means "the entry-year frontier" and 100 means a perfect score. Domain trajectories are √n-weighted means of the per-benchmark 90th-percentile HCI frontiers. The data are 393 model–benchmark observations across ten domains, for models released between 2023 and September 2026. In 2026, bounded domains are near saturation: math 86.4, graduate science 85.8, broad knowledge 77.2. Interactive ones lag: software engineering 52.6, search/terminal 56.8, tool agents 39.9 (up from 8.2). The survey's argument is that remaining headroom sits in long, stateful workflows, which are exactly what RSI loops would target.

**What it does not show.** HCI measures how much benchmark headroom *models* have closed. It is not a measurement of RSI, and no system is scored by it. The "post-2026 RSI stage" in Figure 3 is an **assumed** formula, R = 100 − 0.22(100 − T): RSI is simply taken to close 78% of whatever headroom remains, giving software engineering 52.6 → 89.6. That is an illustration, not a forecast. The autonomy levels are classifications from public descriptions, and many are company-reported; the appendix itself flags entries whose "technical boundary still requires stronger verification". The authors' own firms are among the evidence.

## Using it to audit a self-improving-agent claim

The scale gives [[self-improving-agent]] claims a concrete checklist:

1. Does anything persist past the task? If not, it is B0.
2. Who picks the intervention (L2)?
3. Who picks the next experience (L3)?
4. Is the improver or evaluator itself revised and reused (structural L5)?
5. Does the revised mechanism beat the original from comparable starting points under matched total budget (effective L5)?

Under this reading, [[reflexion]] (listed as a precursor on the concept page) is not RSI at all, and DGM is short of L5.

## Related

- [[self-improving-agent]] · [[darwin-godel-machine]] · [[godel-machine]]: the concept pages. There is no `recursive-self-improvement` page; [[self-improving-agent]] carries the concept for now.
- [[meta-n-recursive-self-improvement-emergent-depth-kim-2026]]: its "frozen driver caps realized meta-depth" is the same observation as this survey's "external controls" column (Table 7), made at the level of mechanism.
- [[self-improvements-in-modern-agentic-systems-ren-2026]]: the companion survey. Ren classifies by *what is modified* (θ vs scaffold); Duan classifies by *who decides*. Both surveys are needed to place a system.
- [[ai4ai-bench-algorithmic-design-chi-2026]] · [[frontis-ma1-ai4ai-recursive-self-improvement-yang-2026]] · [[arex-recursive-self-improving-research-baai-2026]]: AI-for-AI and research agents, mostly L2 under this scheme.
- [[red-queen-godel-machine-iacob-2026]] · [[hyperagents-zhang-2026]] · [[on-the-fragility-of-self-improving-agents-ye-2026]] · [[rehearse-confidence-cliff-autoresearch-ji-2026]]: the L5 candidates and the evidence-quality concerns behind "reliable verification".
- [[continual-learning]] · [[meta-learning]]: neighbouring paradigms (the survey's Table 1). Continual learning "approaches RSI" only when the adaptation process itself is revised.
- [[reward-hacking]]: cited cases of seed cherry-picking and evaluator-label extraction in automated research. [[skill-library]]: "Library Drift" under L4 persistent state.

## References

- [arXiv:2609.11873](https://arxiv.org/abs/2609.11873) (v1, 10 Sep 2026).
- Level exemplars: Self-Refine, Reflexion, Tree of Thoughts (B0); FineWeb-Edu (L1); Self-Harness, ADAS, AFlow, GEPA (L2); SIMA 2, AZR, R-Zero (L3); PANDO, Voyager (L4); STOP, Gödel Agent, DGM, HyperAgents, RQGM, A-Evolve-Training (L5 mechanisms).
- Industry: OpenAI, Anthropic ("When AI builds itself", tagged an L5 *target*), Google DeepMind, Meta AIRA2, Weco AIDE2, Karpathy's autoresearch; 72 companies/teams in Appendix B (Table 12).
