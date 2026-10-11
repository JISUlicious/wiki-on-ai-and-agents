---
title: "Meta^n: Recursive Self-Improvement through Emergent Depth"
type: source
created: 2026-09-05
updated: 2026-10-08
sources: []
arxiv_id: "2608.24735"
year: 2026
authors:
  - Zae Myung Kim
  - Young-Jun Lee
  - Seungyeon Jwa
  - Dongyeop Kang
tags:
  - 2026
  - paper
status: complete
importance: high
---

# Meta^n: Recursive Self-Improvement through Emergent Depth

**Zae Myung Kim, Young-Jun Lee, Seungyeon Jwa, Dongyeop Kang** (Minnesota NLP, University of Minnesota) — [arXiv:2608.24735](https://arxiv.org/abs/2608.24735), cs.AI, v1 25 Aug 2026. Preprint; no venue. Code: `github.com/minnesotanlp/meta-n`.

## Summary

The paper's contribution is first a *definition*, and the definition is what makes the empirical claim legible. **Realized meta-depth** is "the highest level whose behavior actually changes over the course of a run, as opposed to the depth its architecture nominally permits," and a **driver** is "a component that controls such modification but itself never modified during a run." Under those terms the existing literature splits in two, and both halves are shallow:

- **Hand-crafted meta-systems** (FunSearch, AlphaEvolve, [[adas|ADAS]], [[meta-harness-lee-2026|Meta-Harness]], PromptBreeder, [[gepa|GEPA]]) put a frozen search loop outside the solver. "The solver improves; the improver does not. **Realized meta-depth is 1.**"
- **Self-referential agents** (Gödel Agent, [[darwin-godel-machine|DGM]], STOP, [[hyperagents-zhang-2026|HyperAgents]]) let the agent edit its own source, "in principle this permits unbounded depth; in practice **every such system holds some driver fixed** to keep from corrupting itself: Gödel Agent's action API, DGM's archive maintenance and parent selection, HyperAgents' outer evaluation loop." Realized depth is capped at **~2.5** — level 1 for editing the solver, level 2 because the edit logic lives inside the edited source, and half-credit for the frozen driver sitting inside every higher level.

The dilemma: "recursing the improver buys depth only at the price of stability, and every extant system resolves the tension by freezing a driver, which caps the very depth the recursion was meant to deliver." **Meta^n dissolves it by inverting what recurses.** One LLM-prompted meta-operation **Ω** is frozen forever; the *input* recurses. Ω reads the traces of the whole solver stack below **together with the code that produced them**, then writes the next layer as (a) a short Python pre-process injecting strategic context before each task and (b) a small library of callable helper functions. Because Ω never mutates it cannot destabilize the system; because its input strictly grows, "each application operates on inputs of strictly higher order than the last." Depth is not fixed — it grows until Ω stops finding improvements — and an evolutionary archive searches over *chains* of layers rather than greedily extending one.

The result the paper actually defends is narrower than the abstract's, and it is the more interesting one: **depth beyond two produces structurally distinct layers, not redundant ones**, and the distinction is measurable — rollback is *exactly zero* at depth 2 in both substrates under both raters, and appears at depth 3.

## Key points

**Table 1 — [[gemma|Gemma]] 4 31B-IT backbone**, mean ± stdev over seeds 42/43/44. Held-out test for CO-Bench / S2D / LawBench; benchmark score for AE Math, AlgoTune (speedup ×) and SR (4-domain mean).

| Method | CO-Bench | S2D | LawBench | AE Math | AlgoTune | SR |
|---|---|---|---|---|---|---|
| Meta^n archive-best | **0.851 ± 0.014** | 0.733 ± 0.015 | **0.815 ± 0.013** | **0.869 ± 0.045** | **×15.10 ± 2.4** | **5.22 ± 0.38** |
| Meta^n best chain | 0.782 ± 0.016 | **0.743 ± 0.034** | 0.796 ± 0.046 | n/r | n/r | 4.68 ± 0.45 |
| OpenEvolve | 0.814 ± 0.022 | 0.718 ± 0.022 | 0.745 ± 0.034 | 0.802 ± 0.052 | ×10.45 ± 1.8 | n/r |
| Gödel Agent | 0.451 ± 0.023 | 0.710 ± 0.034 | 0.775 ± 0.023 | 0.581 ± 0.061 | ×13.22 ± 2.7 | n/r |

**Table 2 — GPT-5.2 backbone**, same seed policy. ARC-AGI-2 column is the **dev** score.

| Method | CO-Bench | S2D | AE Math | SR | ARC-AGI-2 |
|---|---|---|---|---|---|
| Meta^n archive-best | **0.870 ± 0.011** | 0.725 ± 0.014 | **0.917 ± 0.016** | **5.03 ± 0.20** | **0.331 ± 0.010** |
| Meta^n best chain | 0.806 ± 0.017 | **0.734 ± 0.020** | 0.820 ± 0.024 | n/r | 0.123 (1 seed) |
| OpenEvolve | 0.702 ± 0.025 | 0.721 ± 0.007 | 0.726 ± 0.046 | 3.70 ± 0.13 | 0.003 ± 0.001 |
| Gödel Agent | 0.527 ± 0.033 | 0.708 ± 0.037 | 0.674 ± 0.068 | 2.45 ± 1.63 | 0.054 ± 0.006 |

**Variance and seeds: yes, and unusually well documented.** "Unless noted otherwise, every number is a mean ± stdev over three seeds (42, 43, 44)." Two deviations are declared up front: TB2 stdevs are computed **across the 13 task categories rather than across seeds**, and **TB2 under GPT-5.2 is single-seed (s42)**; benchmarks without a held-out split (AE Math, AlgoTune, SR) report the benchmark score directly. The one ARC-AGI-2 cell that is single-seed is labelled as such in the table itself (`0.123 (1 seed)`). The consolidation study reports a 95% CI. The paper also refuses to over-claim on overlapping ranges: on S2D "the per-seed ranges overlap and **we do not claim significance**"; on CO-Bench GPT-5.2 it notes the +0.168 margin has "**disjoint per-seed ranges**." This is materially better hygiene than either sibling harness paper in this wiki ([[harness-continual-learning-kang-2026]] and [[jit-agent-just-in-time-harness-evolution-zhang-2026]] both report no variance anywhere).

**Emergent layer roles (Figure 6).** 588 of 596 Ω-emissions at depths 2–5, labelled by two independently prompted raters (GPT-5.2 and Kimi-K2.6) over seven categories; only emissions **both** raters agree on are counted, "making every bar a lower bound." Mean inter-rater Cohen's κ = 0.59 — near-perfect on task routing and prompt engineering, weakest on the two most abstract roles (0.37 and 0.30). Multi-label rubric, so bars within a depth exceed 100%.

| Depth | Code substrates: tactical / specialized-lib / rollback | Prompt substrates: rollback / prompt-eng |
|---|---|---|
| 2 | 28% / 49% / **0%** | **0%** / 90% |
| 3 | **45%** / 68% / 55% | 33% / 79% |
| 4 | 26% / 74% / 50% | **47%** / 79% |
| 5 | 17% / 87% / 48% | 33% / 84% |

"Rollback is exactly zero at depth 2 in both substrates, under both raters, and appears at depth 3 — so **the corrective role is the one role that does not exist until a layer has something to correct**." Tactical primitives peak at depth 3 (45%) and decay to 17% by depth 5 as specialized libraries take over. On prompt substrates the mirror image: prompt engineering saturates at every depth, tactical primitives never exceed 9%.

**The depth-3 regression is the paper's own most honest number.** Depth 2 always helps: "every chain improves d1 → d2 (mean lift **+0.113**); depth 2 alone fixes a mean of 3.7 previously-failing tasks per chain." Depth 3 does not: "the aggregate d2 → d3 mean lift is only **−0.006**, but per-task variation is large, with **41% of (chain, task) pairs strictly regressing at depth 3** and 18% dropping substantially (d3 < 0.8·d2)." The worked case: on `multi_demand_multidim_knapsack`, depth 2's greedy + local_search lifted 0.0 → 0.759 while depth 3's simulated_annealing regressed it to 0.230. And mean score **per depth peaks at depth 3 (0.726) and falls to 0.602 by depth 6** — depth pays only through the archive, because "depth-≥4 candidates win 31% of CO-Bench tasks, 80% of SR-matsci, and 70% of SR-phys-osc," and `constrained_guillotine_cutting` first opens at depth 5 with all ancestors at 0.

**Ablations (Table 3).** Each row removes one mechanism:

| Condition | Removal | Score | Δ |
|---|---|---|---|
| Meta^n full (Gemma, CO-Bench) | — | 0.845 | — |
| − code library | L(d) injection | 0.825 | −0.020 |
| − outer context | inter-layer context string | 0.751 | **−0.094** |
| − recursion | meta-layer stacking (depth-1) | 0.714 | −0.131 |
| Meta^n full (GPT-5.2, CO-Bench) | — | 0.886 | — |
| − recursion | depth-1 | 0.806 | −0.080 |
| Meta^n full (GPT-5.2, AE Math) | — | 0.917 | — |
| − recursion | depth-1 | 0.759 | −0.158 |

So **~72% of recursion's gain is a plain string of context passed between layers**, ~15% is the callable code library, ~13% the recursion machinery (beam, retry, inspiration). The paper reports this as a result that undercut its own expectation ("locates the gain more precisely than expected"): the expensive channel is not the one that matters.

**Compute parity.** On CO-Bench Meta^n and OpenEvolve already run at near-identical token budget. On the prompt benchmarks, where full-budget Meta^n spends 10–20× more, the authors **hard-cap Meta^n to OE's budget** (mean 485K tokens, Gemma) and it still leads: S2D 0.732 ± 0.023 vs 0.718 ± 0.022, LawBench 0.784 ± 0.013 vs 0.745 ± 0.034 — "so the full-budget gap contains a compute component but does not reduce to one." On CO-Bench it beats OE using **~13× fewer candidate evaluations (29 vs 378)**. Gödel Agent is run in a *corrected* per-task configuration (its published single-solver interface "collapses to ~0.000" on CO-Bench structurally), and raising its budget 5× then 10× on GPT-5.2 s42 lifts held-out test only 0.502 → 0.615 → 0.628, against Meta^n's 0.870.

**Where it does not work, stated plainly.** AlgoTune is the over-constraint case — the agentic variant *underperforms its own single-shot sibling* (×14.11 vs ×18.47 on s42), traced to two FFT kernels rather than spread across the benchmark. And on **SWE-Bench** "the seed is already strong enough that the archive's best is the generation-0 candidate and **Ω never activates**." Ω's value scales with headroom above the seed times the diversity of failure modes.

## Claims checked against the paper

> [!warning] "Outperforms prior self-improving agents on all eight benchmark families across two backbones" — does not hold as stated
> The abstract says this; the body says something weaker three times: "Meta^n agentic **leads every benchmark family in Tables 1 and 2 on at least one estimator**," and the conclusion repeats "on at least one estimator." Three specific gaps:
>
> 1. **TerminalBench 2.0 has no baseline at all.** "Running either baseline on the Docker substrate would require substantial changes to its code, so TB2 measures Meta^n against its own seeds instead." One of the eight families is a self-comparison (single-shot +0.21 from a 0.067 seed, agentic +0.23 from a 0.258 seed on Gemma; 0.634 on GPT-5.2).
> 2. **No family is contested under both backbones except three.** SR baselines "were run under GPT-5.2 only"; LawBench and AlgoTune "were not run under GPT-5.2 for budget reasons"; ARC-AGI-2 is GPT-5.2 only. Only CO-Bench, S2D and AE Math have baselines on both.
> 3. **The estimator matters.** On Gemma CO-Bench, Meta^n's *best single chain* scores **0.782, below OpenEvolve's 0.814**. The win requires archive-best (0.851), which is a per-task max over the archive — a different quantity from a deployed system's score. The paper is explicit that the archive-over-chain margin (0.06–0.07) "is what per-task selection buys over committing to a single chain," but the abstract's headline elides which one is being compared.
>
> None of this is hidden — the tables carry every n/r — but the abstract's sentence is not supported by them.

> [!warning] "The only method scoring above zero on ARC-AGI-2" — not on the numbers the paper prints
> The abstract says "it alone scores above zero." **Table 2's ARC-AGI-2 column shows OpenEvolve at 0.003 ± 0.001 and Gödel Agent at 0.054 ± 0.006 — both above zero.** The intro states the claim more carefully and restricts it to a split: "On ARC-AGI-2's **held-out split** … Meta^n is the only system to solve any task at all, where both OpenEvolve and Gödel Agent solve none." But Table 2's caption promises "the held-out test result is reported in the text," and **it never is** — no held-out ARC-AGI-2 number appears anywhere in the paper, for Meta^n or for either baseline. The only quantities given are dev: Meta^n archive-best **0.331 ± 0.010**, best chain 0.123 (1 seed), against 0.054 and 0.003. The categorical claim ("6× and 100× the nearest baseline on dev") is strong on its own; the "alone above zero" framing is not what the reported table shows, and the held-out evidence for it is missing.

> [!note] "Realized meta-depth caps at roughly two" — verified, with a sharper number in the body
> The abstract says "roughly two"; §2.1 says **~2.5**, with the half-point being an explicit accounting convention: "the agent modifies the solver (level 1), and because the modification logic lives inside the source being edited, self-edits also change how future self-edits proceed (level 2). But as the frozen driver sits inside every higher level, nothing above level 2 ever changes in full; thus the half-credit (i.e., 0.5) records this partial modifiability." Hand-crafted meta-systems, including [[adas|ADAS]], are assigned realized depth **1**. This is a taxonomy the authors propose, not an independent measurement of other systems — the per-system justifications are in Appendix B.

## Where it breaks

- **Depth is not monotone and the paper knows it.** Mean score per depth peaks at 3 and declines to depth 6; 41% of (chain, task) pairs regress at depth 3. The architecture's value is the *archive* (a max over chains), not depth as such — which makes the framing "emergent depth" partly a claim about search breadth. The consolidation mode fixes this by construction (per-task-best 0.502 → 0.71 ± 0.02, +0.10 over a compute-matched best-of-4 control, 95% CI [+0.04, +0.16], zero regressions on 8/8 tasks) but is an appendix study on an 8-task band.
- **Archive-best is a favourable estimator.** It is the mean of per-task bests across every chain in a monotonically growing archive. Reporting it as the headline against baselines' single scores flatters the comparison; the paper does report best-chain alongside it everywhere, which is the right call, but the abstract quotes neither.
- **Same model at every level.** "We run the same model at the base solver and at every Ω invocation. That is the right control for separating depth gains from capability gains, but it leaves the practical deployment case untested."
- **Depth stops at 3–6 for an unexplained reason.** "Runs stop between depth 3 and depth 6 because Ω stops finding improvements, **not** because the base model runs out of reasoning capacity at higher meta-levels or the context fills with accumulated layer code." That is asserted, not demonstrated — no probe distinguishes "no improvement exists" from "Ω cannot find it."
- **Role labels are LLM-judged.** Figure 6 rests on two LLM raters at mean κ = 0.59, and the two most abstract categories sit at κ = 0.37 and 0.30. The conservative both-raters-agree rule makes the bars lower bounds, but the *categories* are still a rubric applied by models to models.
- **No cost table in the main text.** Agentic mode is "roughly 4 to 10× higher" token cost than single-shot (8 calls per task); the full accounting is in Appendix C.

> [!note] Where the archive sits against catastrophic remembering
> Meta^n's archive is explicitly **"a monotonically growing archive"** with no deletion or consolidation operator — the same shape [[catastrophic-remembering]] predicts should ratchet. Two things stop it from failing in the predicted way here, and both are worth naming because they are the general fix. First, the archive is **read selectively, not dumped**: a chain is sampled by weight `w(c) ∝ S̄(c) + α/(1+children(c))`, and only the best rival *trace* is appended to Ω's prompt on tasks where a parent underperforms. That is the focused-retrieval mitigation [[rehearse-confidence-cliff-autoresearch-ji-2026|Rehearse]] measured (83.5% vs 70.8% for full-history dump). Second, **every entry carries an executed score** — accumulation is filtered by a verifier, not by a judge, so assumption 2 of the ratchet (censored outcomes) does not hold. The residue that *is* uncensored is layer interference: depth 3's 41% regression rate is the store hurting itself, and the system's answer is a *later layer that rolls back*, not deletion. Meta^n therefore keeps the archive and pays for it with an emergent corrective role — an unusual and testable alternative to consolidation.

> [!note] Against the wiki's standing critique of self-improving scaffolds
> [[rethinking-evaluation-harness-evolution-wang-2026|Rethinking]]'s objection is that scaffold evolution is a search procedure and must be compared to plain search at matched budget. Meta^n is the first paper in this wiki's harness cluster to actually run that control: OpenEvolve on CO-Bench at near-identical token budget, and Meta^n **hard-capped to OE's budget** on the prompt benchmarks, where it still leads (S2D 0.732 vs 0.718; LawBench 0.784 vs 0.745). The compute-matched best-of-4 control in the consolidation study is a second one. It is not a *test-time-scaling* baseline in Rethinking's sense (parallel sampling of the base solver), and Rethinking's own finding was that instance-level harness adaptation is the arm that survives — which is [[jit-agent-just-in-time-harness-evolution-zhang-2026|JIT-Agent]]'s territory, not this one. But the budget bar is cleared here in a way it is not in the sibling papers.
>
> On [[self-improving-agent]]'s "compounding rather than overwriting" bullet, Meta^n is a partial defence and a partial confirmation: the *archive* compounds monotonically, while individual *chains* overwrite each other at depth 3 41% of the time. The paper's word for the latter is "layer interference," which is [[harness-continual-learning-kang-2026|HCL]]'s harness-level forgetting arriving by a different route — a committed edit at depth d breaking behaviour that depth d−1 had working. HCL guards it with an executed anchor set before committing; Meta^n does not guard it at all and instead grows a layer whose job is to undo it.

## Related

- [[self-improving-agent]] — the concept this paper re-axiomatizes; its realized-meta-depth definition is the sharpest tool the wiki has for comparing systems on this page.
- [[godel-machine]] · [[darwin-godel-machine]] · [[adas]] — the three ancestors placed in the taxonomy: Schmidhuber's proof-carrying limit case, DGM (paradigm 2, frozen archive maintenance and parent selection), ADAS (paradigm 1, realized depth 1).
- [[harness-continual-learning-kang-2026]] · [[jit-agent-just-in-time-harness-evolution-zhang-2026]] · [[prime-agent-self-improving-rlm-harness-karten-2026]] — the 2026 siblings. All four freeze the model and edit something outside it; only Meta^n freezes the *editor* and recurses on its input, and only Meta^n reports seeds.
- [[catastrophic-remembering]] — the monotonically growing archive with weighted selective retrieval and executed scores; see the callout for why it does not ratchet the predicted way.
- [[rehearse-confidence-cliff-autoresearch-ji-2026]] — focused retrieval over full-history dump; Meta^n's cross-candidate inspiration is the same move. Also the contrast on gating: Rehearse's judge degrades with depth, Meta^n's selection is an executed score.
- [[rethinking-evaluation-harness-evolution-wang-2026]] — the matched-budget-search objection, partly answered here (see callout).
- [[evo-bench-can-language-models-improve-agent-harness-huang-2026]] · [[on-the-fragility-of-self-improving-agents-ye-2026]] · [[self-improvements-in-modern-agentic-systems-ren-2026]] — the wiki's other evidence on whether self-improvement loops actually improve.
- [[red-queen-godel-machine-iacob-2026]] · [[cedar-agent-orchestrated-tree-search-complex-systems-tian-2026]] · [[frontis-ma1-ai4ai-recursive-self-improvement-yang-2026]] · [[arex-recursive-self-improving-research-baai-2026]] — the 2026 recursive-self-improvement cluster.
- [[meta-harness-lee-2026]] — cited by name as a paradigm-1 meta-scaffolding system (realized depth 1).
- [[hyperagents-zhang-2026]] — cited as a paradigm-2 system whose outer evaluation loop is the frozen driver.
- [[meta-learning]] · [[metacognition]] — the meta-level framing; the paper cites Ackerman & Thompson's meta-reasoning and Hofstadter.
- [[library-learning]] · [[skill-library]] · [[dreamcoder]] · [[lilo-grand-2023]] · [[voyager]] — Ω's callable helper functions are learned library primitives; `simulated_annealing` propagates from one depth-2 candidate to 15 of 36 CO-Bench winners.
- [[code-as-harness]] · [[code-generation]] — layers are Python pre-processes plus helper libraries; on TB2 Ω emits bash functions too.
- [[verifier]] · [[outcome-based-reward]] — every archive entry is scored by an executed evaluator, which is what keeps the growing store honest.
- [[llm-as-a-judge]] — used only for the emergent-role annotation (two raters, κ = 0.59), not for selection.
- [[terminal-bench]] · [[swe-bench]] · [[agent-benchmark]] · [[agent-evaluation]] — TB2 (89 tasks, 13 categories) is the no-baseline family; SWE-Bench is where Ω never activates.
- [[test-time-compute]] · [[inference-time-compute]] — agentic mode is 4–10× single-shot token cost for a richer trace signal; AlgoTune is where that backfires.
- [[arc-agi-3-arc-prize-foundation-2026]] — the successor benchmark in the same series; this paper uses ARC-AGI-2 (120 puzzles, pass@2, GPT-5.2 only).
- [[harness-vs-model-attribution]] — the same-model-at-every-level control is exactly the attribution design that page asks for, and the limitation section names the untested asymmetric case.
- **No wiki page yet:** `arc-agi` (benchmark family), realized meta-depth (this paper's concept), OpenEvolve, Gödel Agent (Yin et al. 2024), FunSearch, AlphaEvolve, PromptBreeder, GEPA, CO-Bench, AlgoTune, Symptom2Disease, LawBench, symbolic regression as a benchmark, University of Minnesota.

## References

- [arXiv:2608.24735](https://arxiv.org/abs/2608.24735)
- Code: https://github.com/minnesotanlp/meta-n
- ARC-AGI-2 ([[francois-chollet|Chollet]], Knoop, Kamradt, Landers, Pinkard), [arXiv:2505.11831](https://arxiv.org/abs/2505.11831)
