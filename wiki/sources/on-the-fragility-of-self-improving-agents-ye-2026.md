---
title: "On the Fragility of Self-Improving Agents: Variance, Task Order, and Underspecification"
type: source
created: 2026-09-05
updated: 2026-09-29
sources: []
arxiv_id: "2608.18066"
year: 2026
authors:
  - Qinyuan Ye
  - Yu Li
  - Yada Pruksachatkun
  - Jiaxin Zhang
  - Chien-Sheng Wu
tags:
  - 2026
  - paper
status: complete
importance: high
---

# On the Fragility of Self-Improving Agents: Variance, Task Order, and Underspecification

**Qinyuan Ye, Yu Li, Yada Pruksachatkun, Jiaxin Zhang, Chien-Sheng Wu** (Salesforce AI Research) — [arXiv:2608.18066](https://arxiv.org/abs/2608.18066), cs.AI / cs.CL / cs.LG, 18 Aug 2026, v1. CC BY 4.0. A NeurIPS paper checklist is attached but no venue is stated. Code and data: `SalesforceAIResearch/self-improve-fragility`.

## Summary

This is a **re-evaluation paper**, not a method paper. It takes two established *memory-based* self-improving agents — **Agent Workflow Memory (AWM)** (Wang, Mao, Fried, Neubig, ICML 2025), which distills reusable step-by-step workflows from successful trajectories and puts all of them in context, and **ReasoningBank (RBank)** (Ouyang et al. 2025), which extracts free-form reasoning insights from both successes *and* failures and retrieves the most relevant ones per task — and asks whether their reported gains survive a stricter protocol. The setting is the standard one for this pattern: an online stream of N tasks, a textual memory bank, a memory-construction module that updates the bank after each task from the trajectory and (optionally) the reward. The paper's own name for the class is a useful one to hold onto: **memory-based self-improving agents**, i.e. the inference-only, no-weight-update end of the [[self-improving-agent]] spectrum.

Three things change relative to the original papers. The **baseline is upgraded** — GPT-5-mini as both the agent and the memory-construction model, on the WALT harness for WebArena and VisualWebArena and the SCUBA harness for SCUBA — so that the no-memory agent alone already scores **54.8% on WebArena**, above the *memory-enhanced* numbers the prior papers reported. The memory constructor is given the **ground-truth reward** instead of the LLM-judge proxy the originals used (the authors found the proxy noisy enough to confound the analysis). And the evaluation is broadened along two axes: **three identical runs per configuration** to measure variance, and **two randomly shuffled task orders** (Shuffle-1, Shuffle-2) in addition to the default ascending-task-ID order every prior paper used.

The verdict, in the authors' own words from the conclusion: *"agents do not self-improve, but instead degrade, in challenging settings where the baseline is already strong or when tasks arrive in a random order."* Two mechanisms are documented. First, self-improvement loops are stateful, so early sampling noise compounds into divergent memory states — variance across runs **increases in 17 of 24 (≈71%) cases**, and the best-worst gap of a single configuration reaches **10.4 points**. Second, the default task order is an **implicit easy-to-hard curriculum** — pass rate starts near 75% and falls below 40% past task ID 150 — and it turns out to be a hidden prerequisite: under the default order RBank gains **+1.5** on WebArena (p = 0.23), under Shuffle-1 both methods lose **~5 points** against the no-memory baseline. A manual read of the memory bank attributes much of this to **underspecification**: the constructor is never told the environment is browser-only, so it writes plausible-but-unexecutable advice ("use the API", "ask the user to confirm") that then gets retrieved and followed. Supplying rubrics, environment feedback, and an explicit prompt constraint closes **31%** of the shuffled-order degradation; **69% remains unexplained**.

## Key points

**Benchmarks and scale.** [[gui-agent|Web-browsing]] only: WebArena (812 tasks, six domains), VisualWebArena (910 tasks, three domains), and SCUBA, an enterprise CRM benchmark (267 tasks after manually removing tasks invalidated by site updates; the SCUBA headline metric is *Avg. Milestone* 49.6, whereas its strict *Avg. Pass* is 29.7). One no-memory WebArena run costs ≈$25 and one SCUBA run $29 on GPT-5-mini. Prior work reports single-run pass@1 almost without exception; the paper flags [17, 18] as the rare multi-run cases.

**Variance — the baseline is already noisy, and the methods make it worse** (Table 1, three runs each; relative change vs. baseline in parentheses):

| Domain (n) | Baseline std / gap | AWM std / gap | RBank std / gap |
|---|---|---|---|
| WA GitLab (180) | 1.98 / 4.44 | 1.20 (−39%) / 2.78 (−38%) | **3.34 (+69%) / 7.78 (+75%)** |
| WA Map (109) | 1.30 / 2.75 | 1.98 (+53%) / 4.59 (+67%) | **3.89 (+200%) / 8.26 (+200%)** |
| WA Multisite (48) | 0.98 / 2.08 | 0.98 (0%) / 2.08 (0%) | **4.28 (+336%) / 10.42 (+400%)** |
| WA Admin (182) | 1.19 / 2.75 | 2.99 (+152%) / 6.59 (+140%) | 2.12 (+79%) / 4.95 (+80%) |
| VWA Reddit (210) | 1.03 / 2.38 | 2.38 (+131%) / 5.71 (+140%) | 1.47 (+43%) / 3.33 (+40%) |
| SCUBA Admin (158) | 0.30 / 0.63 | 2.09 (+600%) / 4.43 (+600%) | 1.58 (+429%) / 3.80 (+500%) |
| SCUBA Service (45) | 2.77 / 6.67 | 2.77 (0%) / 6.67 (0%) | 3.78 (+36%) / 8.89 (+33%) |

Std and best-worst gap are in pass@1 points. Across all 24 method × domain cells, variance rises in 17 and by more than 50% in 11. The intro's headline instance: the GitLab best-worst gap for the *baseline* is 4.4 points — "which in some contexts would often be considered a meaningful improvement" — and RBank widens it to 7.8. The paper's diagnosis is structural rather than a bug: memory construction is conditioned on prior outcomes and on stochastic sampling, so "early randomness can compound over time, leading to substantially different memory states across runs."

**Reported gains vs. re-evaluated gains.** This is the direct answer to "how much do the gains shrink":

| Method | Original report (model, tasks) | Original Δ | This paper's baseline | This paper's Δ (default order, mean of 3 runs) |
|---|---|---|---|---|
| AWM | 32.7 → 36.3 (Claude-3.5-Sonnet, 812 WA tasks) | **+3.6** | 54.8 | **−0.7** on WA; −0.4 VWA; +0.5 SCUBA |
| RBank | 46.7 → 53.9 (Gemini-2.5-Pro, 684-task WA subset) | **+7.2** | 55.3 on the same 684 subset | **+1.5** on WA (p = 0.23); +0.7 VWA; +1.5 SCUBA |

The no-memory GPT-5-mini baseline (55.3% on the 684-task subset) already exceeds ReasoningBank's *memory-enhanced* 53.9%, and its 54.9% on VisualWebArena matches the leaderboard best (54.0% as of 2026-05-01). RBank's +1.5 on WebArena is an unpaired t-test over three runs at **p = 0.23**. The authors' framing is that the methods "are effective for less capable web agents" and that "more advanced approaches are required" once the initialization is strong — a diminishing-returns reading, not an accusation. Human WebArena performance is 78%, so headroom exists; the methods just do not use it.

**Task order — the implicit curriculum.** The default order is ascending task ID. Plotting the *no-memory* agent's moving-average pass rate (window 30) along that order shows it starting around **75%** and dropping **below 40% once task ID exceeds 150** on both WebArena and VisualWebArena (SCUBA oscillates). The authors suggest this is a benchmark-construction artifact — annotators write easy tasks first. Under two shuffles, performance "degrade[s] significantly in **6 out of 8 cases**." The numbers the text reports:

| Setting | Baseline | AWM | RBank |
|---|---|---|---|
| WebArena, default order | 54.8 | 54.1 (−0.7) | 56.3 (+1.5) |
| WebArena, Shuffle-1 | 54.8 | **49.1 (−5.7)** | **49.8 (−5.0)** |
| Abstract's summary (RBank) | — | — | +1.5 default → **−4.5** shuffled |

On VisualWebArena AWM is order-sensitive while RBank is comparatively robust. The key asymmetry: shuffling the order should at worst leave the agent at the no-memory level (the tasks are the same); instead it falls *below* it, meaning the memory is actively harmful when the curriculum is removed.

**Underspecification — what is actually in the memory bank.** Manual inspection of RBank memories (qualitative, on a subset) finds three failure modes:

- *Environment underspecification.* The RBank construction prompt is generic ("extract and summarize useful insights…"); it never says the environment is browser-only with no code execution. So the agent repeatedly writes and retrieves memories recommending **API-based solutions**, and **"user confirmation"** (26 memory items across 3 WebArena runs, 22 across 3 VisualWebArena runs) — which in this environment produces `wait` actions until timeout.
- *Task underspecification.* WebArena task 118 ("I have jaw bruxism problem, show me something that could alleviate the problem") is meant as navigate-to-a-mouth-guard; the agent answers as a medical QA, fails, and reflects into the memory "gather targeted patient/context details before giving medical guidance." Task 329 fails on an evaluator bug (`0.00` ≠ `0` under string match) and the agent writes a spurious memory about checking alternate accounts.
- *Memories are "contagious."* In the Map domain the OSM routing page sometimes fails to load, the agent falls back to the **Haversine formula** on URL-embedded coordinates, occasionally gets the answer right (task 38: "yes, under one hour"), and stores the trick. The earlier it enters memory in a run, the more it is retrieved later — a stochastic seed that the authors explicitly tie to the run-to-run variance. Appendix C names this as **[[reward-hacking]] occurring during the agent's learning process**, aligned with Baker et al.'s CoT-monitoring observations.

**Partial fix — better-specified memory construction.** Three additions at construction time only (the agent never sees them while acting): **+Rub** (the benchmark's function-based rubric and pass/fail per criterion), **+Env** (click/input action success or error), **+PMod** (a prompt edit: "STRICTLY AVOID: API or programmatic solutions, visiting external websites, requesting human confirmation…"; include procedural knowledge and site navigation).

| RBank, WebArena | Original | +All |
|---|---|---|
| Shuffle-1 | 49.8 | **52.7 (+2.9)** |
| Shuffle-2 | — | +1.1 |
| Default | 56.3 | maintained |

Each addition helps modestly on its own; combined they are "beneficial and non-disruptive." But **even with +All the agent still underperforms the no-memory baseline under both shuffles.** The paper's accounting is that this closes 31% of the shuffled-order degradation and leaves 69%; I could not reconstruct the 31% exactly from the numbers given in the text, so treat it as the authors' figure.

**Stated limitations.** Scope is memory-based methods in web browsing only — no scaffold-, code-, or weight-level self-improvement, and the two methods are the *minimalist* ones, chosen because they are "highly scalable and increasingly adopted"; newer memory-management techniques were not evaluated. Memory inspection was qualitative on a subset. Three runs is what the budget allowed, so the variance estimates are themselves noisy.

**Recommendations.** For evaluation: never a single run; report multi-run statistics and randomized task orders; pilot on well-specified benchmarks, then stress-test on underspecified ones for worst-case. For systems: "without proper validation mechanisms, agent memories are merely **unverified hypotheses** rather than true lessons learned" — build memory validation that filters bad items; and because underspecification cannot be foreseen and deployments are "a single, irreversible run," build human-oversight interfaces that can catch the wrong lesson early.

## How this lands on existing pages

This paper is the most direct empirical test in the wiki of the [[self-improving-agent]] page's core-mechanism bullets, and it refutes three of them for the memory-based end of the spectrum:

> [!success] Resolved contradiction — "compounding rather than overwriting … capability grows monotonically"
> [[self-improving-agent]] states that appended skills/prompts make "the agent's capability grow monotonically." Here the same class of system, under a stronger initialization and a randomized order, goes **below** the agent with no memory at all (WebArena 54.8 → 49.1 AWM / 49.8 RBank under Shuffle-1), and the shuffled-order gap survives every fix tried. This is the same conclusion [[continualskillbench-guan-2026]] and [[catastrophic-remembering]] reached from library size and instruction-file growth, now reached from run variance and task order.
>
> **Resolved 2026-09-29:** [[self-improving-agent]]'s mechanism section was rewritten to reflect this finding.

> [!success] Resolved contradiction — "every proposed modification is trialed … and accepted only if it passes"
> The page's "empirical validation loop" bullet describes DGM-style scaffold evolution, but memory-based methods have **no such gate**: AWM dedups workflows and RBank retrieves by relevance; neither validates a memory item before it is written. The paper's Haversine case shows an unvalidated item entering memory on a lucky pass and propagating. This is the write-side counterpart to [[rehearse-confidence-cliff-autoresearch-ji-2026]]'s finding that the *select* gate degrades along the trajectory: Rehearse shows the gate gets worse; this paper shows the common memory systems have no gate at all.
>
> **Resolved 2026-09-29:** [[self-improving-agent]]'s mechanism section was rewritten to reflect this finding.

> [!success] Resolved contradiction — "memory of past attempts … preserves stepping stones"
> The stepping-stone framing assumes memory contents are correct. §4.1 shows a materially different object: memories reflecting evaluator bugs, misread task intent, and environment-inapplicable strategies, which are "contagious" and seed run-to-run divergence. The [[memory-management]] lifecycle's *Write* step ("what should be recorded at all?") and the [[memory-construction]] framing both treat construction as an extraction problem; this paper shows that with a generic constructor prompt the answer to "what gets recorded" includes whatever the agent happened to try.
>
> **Resolved 2026-09-29:** [[self-improving-agent]]'s mechanism section was rewritten to reflect this finding.

Consistent with, and sharpening, three other pages: [[frontis-ma1-ai4ai-recursive-self-improvement-yang-2026]] reported **71.21% ± 8.57 pp** over three epochs on [[mle-bench]] and the wiki already reads the ranking through that variance — this paper generalizes the point (variance rises in 71% of cases *because* the loop is stateful) and makes the single-run reporting norm the target. [[rethinking-evaluation-harness-evolution-wang-2026]] demanded a matched-compute search baseline before attributing scaffold gains to the mechanism; this paper's analogue is a **matched-model, multi-run, shuffled-order** baseline before attributing memory gains. And [[agent-evaluation]]'s "benchmark scores are claims about the benchmark as much as the agent" gets a concrete instance: the `0.00` vs `0` evaluator bug not only miscounts a pass but *teaches the agent a wrong lesson*.

## Related

- [[self-improving-agent]] — the pattern under test; the three Contradiction callouts above attach to its core-mechanism bullets.
- [[memory-management]] / [[memory-construction]] — the write phase is where underspecification enters; the RBank constructor prompt is reproduced in Appendix E.
- [[memory-evolution]] — contrast: neither AWM nor RBank rewrites existing memories when a new one arrives, so a wrong early memory is never revised, only diluted.
- [[catastrophic-remembering]] — same family of result; here the accumulated artifact is memory items rather than instruction files, and the failure is measured as variance and order-sensitivity rather than growth.
- [[continualskillbench-guan-2026]] — the other 2026 result finding no aggregate advantage for an explicit library over plain in-context adaptation.
- [[rehearse-confidence-cliff-autoresearch-ji-2026]] — the selection-gate degradation; complementary to this paper's missing write-gate.
- [[rethinking-evaluation-harness-evolution-wang-2026]] — the matched-baseline argument for scaffold evolution; this is the same argument for memory methods.
- [[frontis-ma1-ai4ai-recursive-self-improvement-yang-2026]] · [[mle-bench]] — the wiki's existing ±8.57 pp variance example.
- [[reward-hacking]] — the Haversine fallback is reward hacking that happens *inside the learning loop* and is then memorized.
- [[agent-evaluation]] — single-run reporting, evaluator bugs, and the paper's evaluation recommendations.
- [[gui-agent]] · [[osworld]] — the web/computer-use agent family and benchmarks.
- [[skill-library]] / [[agent-skills]] / [[procedural-memory]] — AWM's workflows are the textual-skill form of a skill library; the paper's §5 treats "skills" and "workflow memories" as the same object.
- [[sage-skill-library-wang-2025]] — cites AWM as the prompt-induced skill baseline it improves on via RL; this paper measures the prompt-induced baseline's reliability.
- [[webxskill-skill-learning-web-agents-wang-2026]] · [[beyond-domains-reusing-web-skills-he-2026]] — other WebArena skill-learning results, all single-run and default-order as far as their pages record.
- [[pro-long-programmatic-memory-fox-2026]] — argues the write step should be "as dumb as possible"; this paper is evidence that a *smart-but-underspecified* write step is the problem.
- [[self-improvements-in-modern-agentic-systems-ren-2026]] — the survey's "performance trajectory under a budget" evaluation formalism is what this paper operationalizes with multi-run, multi-order curves.
- [[demystifying-evals-for-ai-agents-anthropic-2026]] — cited by the paper for evaluation science; Anthropic's "grade the outcome" advice is complicated by outcomes reached via unintended strategies.
- [[darwin-godel-machine]] · [[godel-machine]] — cited as the scaffold/code-level self-improvement the paper explicitly does *not* evaluate.
- [[graham-neubig]] — AWM co-author; WALT (the baseline harness) and WebArena share authorship lineage.
- [[a-mem]] · [[mem0]] · [[memgpt]] · [[memorybank]] · [[titans]] — memory systems cited in related work; not evaluated.
- No pages yet: ReasoningBank, Agent Workflow Memory, WebArena, VisualWebArena, SCUBA, Salesforce AI Research, and the constructs *implicit curriculum / task-order sensitivity* and *underspecification*.

## References

- [arXiv:2608.18066](https://arxiv.org/abs/2608.18066)
- Code and data: `github.com/SalesforceAIResearch/self-improve-fragility`
- Re-evaluated methods: Agent Workflow Memory (Wang, Mao, Fried, Neubig, ICML 2025); ReasoningBank (Ouyang et al., arXiv:2509.25140)
