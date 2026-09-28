---
title: "Recursive Experiential–Working Memory Evolution for Long-Horizon Agent Harnesses"
type: source
created: 2026-09-05
updated: 2026-09-05
sources: []
arxiv_id: "2608.24876"
authors:
  - Zhaochen Yu
  - Yingcheng Wu
  - Zhenfei Yin
  - Kaiyuan Chen
  - Zhe Zhao
  - Mengdi Wang
  - Shuicheng Yan
  - Ling Yang
year: 2026
status: complete
importance: high
tags:
  - 2026
  - paper
  - harness
---

# Recursive Experiential–Working Memory Evolution for Long-Horizon Agent Harnesses

**Zhaochen Yu, Yingcheng Wu, Zhenfei Yin, Kaiyuan Chen, Zhe Zhao, Mengdi Wang, Shuicheng Yan, Ling Yang** (NUS; [[princeton-university|Princeton]]; [[stanford-university|Stanford]]; University of Oxford) — [arXiv:2608.24876](https://arxiv.org/abs/2608.24876), cs.AI, 25 Aug 2026, v1. The system is called **Recuris**. No venue stated.

## Summary

The problem is stated crisply: existing experiential-memory harnesses retrieve skills either **once from the initial instruction** — which stops describing the task almost immediately — or **from the full interaction history**, which by mid-episode "mixes outdated information, completed steps, unresolved requirements, and execution noise." Neither is a usable retrieval key. The paper's claim is that the missing piece is not more experience but **a compact, verified task state** against which experience can be selected.

Hence the split. **Experiential Memory (EM)** holds reusable skills in the [[skill-md-format|agent-skill format]]. **Working Memory (WM)** holds a structured per-goal state: content, status ∈ {pending, done, blocked}, supporting evidence, optional blocker. WM decides *what is needed now*; EM supplies *the corresponding experience*; the environment's response then *verifies* whether the state may advance. The loop is `w_t →ρ E_t →(π,T) (a_t,o_t) →(U,C,K) w_{t+1}`.

Two design details do the real work.

**State-grounded invocation happens at execution events, not per task.** The paper's `call-time` deliverer fires the moment the agent *drafts* a state-changing tool call; the draft is **not executed** — the harness returns a synthetic "not-executed" result — so the matched skill reaches the model *before* any irreversible action, and the action finally issued is drafted with that skill in context. A second `boundary` deliverer fires on a turn-boundary predicate (Terminal-Bench uses `first_turn`).

**A completion predicate, not the model's own claim, closes a goal.** Each checker `C_{k,g}` evaluates the tool receipt or environment result. "Invoking a skill or attempting a tool call is not completion evidence." A goal moves `pending → done` only on a matching receipt.

The Skill Memory is therefore **four components, not two**: `M_k = (E_k, W_k, ρ_k, C_k)` — skills, working-memory *specification*, invocation policy, checker set. That quadruple is exactly the patch space of the evolution loop. Because coupling links state → skill → action → observation → verified update, a failed run yields a **structured trace** `Γ_k` rather than a transcript, and a fixed Meta-Agent can attribute each diagnosed failure to one of the four components and patch only that one. A fixed validation gate admits the candidate only if it repairs the source task without regressing a held-out dev set of anchor tasks. **The base LLM, the Meta-Agent, the localization and patching procedures, and the gate never change** — recursion is confined to the externalized memory-control layer.

## Key points

**Main result** (Table 1; avg@4 task success %, all with the base model frozen at temperature 0, k=4, 200-step cap; the Skill Memory for each benchmark was evolved **once on doubao-seed-2-0-pro** and shipped unchanged to every other model). Recuris improves **35 of the 37 completed model–benchmark pairs**; the only two declines are Granite-4.1-3B on SkillFlow (−0.3) and Gemini 3.7 Flash on τ²-Airline (−1.5).

| Model | τ²-Retail | τ²-Airline | SkillFlow | Terminal-Bench 2.1 |
|---|---|---|---|---|
| Granite-4.1-3B | 9.7 → **23.0** (+13.4†) | 34.3 → 39.8 (+5.5) | 0.3 → 0.0 (**−0.3**) | 0.6 → 3.1 (+2.5) |
| Qwen3.5-4B | 68.0 → 68.3 (+0.3) | 75.3 → 79.0 (+3.8) | 6.0 → 7.1 (+1.1) | 10.1 → 13.0 (+2.9) |
| Qwen3.5-9B | 77.6 → 79.6 (+2.0) | 75.5 → 78.4 (+2.9) | 15.1 → 18.4 (+3.4) | 17.4 → 20.5 (+3.1) |
| GPT-OSS-20B | 50.6 → 60.8 (+10.2†) | 54.8 → 59.3 (+4.5†) | 7.8 → 10.4 (+2.6†) | 3.9 → 6.7 (+2.8) |
| Qwen3.6-27B | 62.8 → 71.2 (+8.3†) | 79.0 → 80.0 (+1.0) | 42.2 → **58.7** (+16.6†) | 38.8 → 42.1 (+3.3) |
| Qwen3.6-35B | 78.2 → 78.5 (+0.3) | 80.3 → 81.5 (+1.3) | 35.3 → **48.8** (+13.5†) | 33.1 → 36.4 (+3.3) |
| Gemini 3.7 Flash | 73.5 → 78.3 (+4.8) | 86.5 → 85.0 (**−1.5**) | n/a | 79.8 → 82.4 (+2.6) |
| **GPT-5.6 Sol** | 58.3 → **76.1** (**+17.8†**) | 79.0 → 86.0 (+7.0†) | n/a | 83.2 → 86.4 (+3.2) |
| **Claude Opus 5** | 72.4 → **87.9** (**+15.6†**) | 89.5 → 90.5 (+1.0) | n/a | 84.6 → 88.4 (+3.8) |
| Doubao-2.0-Pro (deployment) | 58.1 → **81.4** (+23.3†) | 75.5 → 80.5 (+5.0) | 34.6 → 51.4 (+16.8†) | 46.1 → 48.9 (+2.9) |

† = paired task-clustered bootstrap 95% CI excluding zero. All three headline numbers verify exactly (Table 7 gives +17.76 and +15.57 to two decimals, and 87.94 for Opus 5).

> [!note] "τ-bench" here means **τ²-Bench**, and the +17.8 is the Retail domain
> The paper evaluates the two domains of **τ²-Bench** (Barres et al. 2025) — τ²-Retail (114 tasks) and τ²-Airline (50 tasks) — not the original τ-bench. The +17.8 / +15.6 / 87.9% figures are all **τ²-Retail**. On τ²-Airline the same package is worth +7.0 to GPT-5.6 Sol and only **+1.0** to Opus 5, whose baseline there is already 89.5. This matters for reading the result: the wiki's [[compaction-cliff-long-running-agent-memory-zerhoudi-2026|Compaction Cliff]] page uses the *original* τ-bench retail/airline, so the two are not directly comparable.

**The ablation is the paper's most important table**, because it says the split is asymmetric (Table 2; matched on model, user simulator, skill library, tools, tasks, seeds, budget):

| Variant | τ²-Retail success | Δ vs base [95% CI] | τ²-Airline | Δ |
|---|---|---|---|---|
| Base (no EM, no WM) | 58.1 (265/456) | — | 75.5 (151/200) | — |
| **EM only** | 60.1 | **+2.0** [−4.0, +7.9] | 77.0 | +1.5 [−4.5, +7.5] |
| **WM only** | 82.0 | **+23.9†** [+17.5, +30.3] | 79.5 | +4.0 [−5.0, +13.5] |
| Model-controlled invocation | 65.6 | +7.5† [+1.5, +13.4] | n/a | n/a |
| **EM + WM (Recuris)** | **83.6** (381/456) | **+25.4†** [+18.4, +32.5] | **84.0** | +8.5 [−1.0, +18.5] |

**The working state carries nearly the whole effect.** Experiential memory alone is worth +2.0 with an interval spanning zero; added *on top of* a working state it contributes a further +1.5 [−2.4, +5.7]. The authors' own conclusion is blunt: "the skills do not carry capability on their own either." Skills are **invocation-conditional** — "what it is worth depends less on what it contains than on whether anything knows when it is needed."

**Long-horizon failure is an execution problem, not a retrieval problem.** Stratifying τ²-Retail into quartiles by *task-intrinsic* completion length (not the length a variant realises — that would be circular, and the choice moves 44 of 114 tasks between quartiles), Recuris leads in **all four quartiles by +17.0 to +44.7 points with no monotone decline**. Read-action recall stays at **88.0–97.9% for every variant in every quartile** — nobody fails to find out what the task needs. The entire separation is on the write path: **+26.7 points of required-write recall**. The base agent ends **42%** of write-requiring episodes having executed none of them, against **16%** for Recuris — while the median turn of the first correct write is **18 in all four variants**. Length costs completion, not comprehension.

**Invocation control beats skill content and beats more context** (Table 3; Model-controlled and Recuris carry the same ten skills, **byte-identical**):

| | Base | WM only | Model-controlled | Recuris |
|---|---|---|---|---|
| Task success (%) | 58.1 | 82.0 | 65.6 | **83.6** |
| Required-write recall (%) | 55.7 | 80.9 | 61.1 | **82.4** |
| Omitted required writes / episode | 0.596 | 0.145 | 0.417 | **0.121** |
| Agent tokens per success (k) | 116 | 102 | 147 | **101** |

Putting the whole library in context and letting the model decide scores **below the identical configuration carrying no skills at all** (65.6 vs 82.0) and costs 46% more per success. Appendix D: the always-resident regime carries **3,111 more tokens at the first call**, scores **18 points lower**, and costs 46% more per success. Errors also compound — once a required write mismatches, the probability a later one mismatches exceeds **60%** in every configuration — and Recuris enters that regime least often.

**The critical mechanism is a property of the domain, not the architecture** — and this is what justifies the whole evolution loop. WM is maintained by four mechanisms (write review, truth guard, status board, gate termination). Removing each in turn produces a **double dissociation**: on τ²-Airline removing *write review* costs **13.5 points** while the status board costs nothing measurable; on τ²-Retail the *status board* costs **17.3 points** while write review costs 0.7 with an interval containing zero. The **truth guard is the only mechanism that checks after the fact, and the only one that never matters — despite rejecting 172 unsupported completion claims**, because "a write that has executed has already moved the environment, and a later audit can record the error but not undo it." If the critical component cannot be read off the architecture, it cannot be chosen at design time — so read it off the failed trace instead.

**Structured traces make component faults observable** (Table 4; a known fault injected into one component, a fixed judge names it; balanced pool, so a constant answer scores 33.3%):

| Evidence given to judge | E | W | ρ | Macro | Macro-F1 |
|---|---|---|---|---|---|
| Outcome only | 0.0 | 38.9 | 0.0 | **13.0** | 10.4 |
| Raw trajectory | 61.1 | 50.0 | 0.0 | 37.0 | 31.2 |
| **Structured trace Γ** | 72.2 | 83.3 | 38.9 | **64.8** | 63.4 |

The outcome condition sits **below the constant-answer floor**; the raw trajectory clears it by under four points. The gain is *observability*, not reasoning: an invocation fault is a **non-event** — nothing appears in the transcript where a skill should have been injected — so neither non-Γ condition names it even once (0.0 → 38.9). Macro precision moves further than recall (27.6% → 64.4%) because Γ stops the judge charging the skills for another component's fault. Checkers (`C`) are excluded from the ground truth because removing them changes τ²-Retail pass rate by zero.

**Evolution compounds, and the Meta-Agent is replaceable.** From **16 training tasks** (12 for the gate, **86 frozen and never read**), every evolution run's package clears `M_0` with an interval excluding zero, by **+9.01 to +17.44 points**. Run C's second round adds **+6.98** on top of the first. Two runs of the byte-identical `M_0` differ by +0.00 [−6.98, +7.27], which sets the instrument's resolution. Swapping the Meta-Agent from [[claude-code|Claude Code]] to DeepSeek Harness moves the result by **−1.45 [−7.85, +4.65], p=0.72** — less than the instrument's own noise — and the two independent implementations converge *mechanistically* too, distilling the same sixteen failures into the same repair family (a service-request-authorization state field, an execution-gate check, anti-escalation skills). "What the loop learns is determined by the evidence it diagnoses from, not by the machinery that carries the diagnosis."

**Gating is cautious rather than protective.** Accepted patches broke 4 of the 42 dev tasks the base already solved (9.5%) against **25.9% for re-running a byte-identical package** on the same tasks (p=0.013) — i.e. regression is dominated by decode noise. All 18 rejected candidates had a dev interval containing zero. A fourth run that admitted everything and merely *recorded* the dev verdict landed all three packages at **+14.5 to +18.0** over `M_0` with every interval excluding zero, even though the gate had condemned two of them at −10.4 and −6.2. **The gate discards noise, not demonstrated gains** — but on this evidence it is also not yet demonstrated to be necessary.

**Memory only grows, and no single skill carries the gain.** Eight accepted patches added **51 skills, revised 2, deprecated none**, with 17 near-duplicate pairs surviving into admitted versions. Yet stripping the five procedure skills, then the remaining two, then the largest skill alone moves the held-out result by **−2.3 to +1.7** with every interval containing zero. Redundant rather than fragile — which is the paper's argument that a pruning operator is a natural addition rather than a risk.

**Transfer is bounded by what the receiving model gets wrong.** The single package lifts every model it is given to, and the *strongest* target ends highest (Opus 5 at 87.9 vs 81.4 on the model the package was evolved from) — so it is not a crutch for weak models. But two nearly identical starting levels get very different value: at 72.4 and 73.5 the same package is worth **+15.6 and +4.8**. "A SkillFlow package carries **procedure**, and any model able to run the procedure can use it. A τ² package carries **discipline** — what to verify and when a goal is still open — and discipline is worth only what the target model's own failures make it worth." Where held-out tasks retain no repairable failures, transfer is zero: two τ²-Airline lineages on 25 and 29 unseen tasks separate from zero on neither, both starting at 73.0% and 81.0%.

**Test-time adaptation (Terminal-Bench 2.1) is the paper's most honestly-reported negative** (Table 8, 87 tasks, budget of 4, stop at first success):

| Configuration | Budget | Solved | Δ | p | Δ isolates |
|---|---|---|---|---|---|
| Terminus-2 (baseline) | 1 | 30/87 (34.5) | — | — | — |
| + seed memory | 1 | 28/87 (32.2) | −2.3 | 0.824 | the layer alone |
| + seed memory, retry | 4 | 51/87 (58.6) | **+26.4** | <10⁻⁴ | **the attempt budget** |
| + test-time adaptation | 4 | 53/87 (60.9) | +2.3 | 0.774 | learning, matched budget |

The 60.9% headline is **almost entirely the retry budget**: retrying the *seed* memory is worth the same +26.4 on its own (23 tasks flip to solved, none flip back), and at a matched budget adaptation adds +2.3 (7 won, 5 lost, p=0.774). On the per-attempt metric a retry budget cannot buy, avg@4 on the 56 tasks with a learned skill moves 17.4 → 21.9 (+4.5, CI [−0.9, +9.8]); all four cuts move the same direction (+2.3 to +4.5) but **every interval contains zero**. The authors report a direction, not an effect. Cross-task evolution on Terminal-Bench **admitted no patch in thirteen runs** of the loop, because the benchmark has no shared task structure to carry a repair across.

## Does state-grounded invocation answer the routing-precision collapse?

The question is whether grounding [[skill-selection|skill selection]] in explicit task state answers what [[demystifying-agent-skills-jiang-2026]] measures: **actual-use precision falling from 29.6% at pool size 5 to 3.3% at 100**, while recall stays at 54.3–73.6% and downstream success barely moves (36.4% → 39.3%).

**Yes in mechanism, and Recuris hits the exact failure Jiang isolates.** Jiang's diagnosis is that the collapse is *over-browsing*, not missing — the right skill is found, the agent just also touches many wrong ones and does not restrict use to it — and that "semantic confusability, not pool size, is the dominant offline stressor." Both of those are failures of *when-to-invoke*, not of what-is-stored. Recuris removes the decision from the model entirely: `ρ` fires on an execution event (a drafted state-changing call) with the *named tool* as the retrieval key, so there is no browsing step to over-browse in. The measured contrast is the closest thing in the literature to a controlled version of Jiang's Arm 3: **the same ten skills, byte-identical bodies, all resident vs state-selected — 65.6 vs 83.6, an 18.0-point gap [+11.6, +24.6]**, with the all-resident variant scoring *below* carrying no skills at all. Availability is not the mechanism. And uniquely, `ρ` is itself an evolvable component: both the trigger predicate and the retrieval key are patch targets, so a "missed, late, or irrelevant skill invocation" is a diagnosable, repairable fault class rather than a property of the model.

**But it is not a demonstrated answer at Jiang's scale, and the two papers disagree about whether bad routing even costs anything.**

- **Scale.** Recuris' τ²-Retail library holds **ten skills** (eight tool skills in the model-controlled arm). Jiang's collapse is measured from 5 to **100**. Recuris never tests a 100-skill pool, and the mechanism that saves it — a retrieval key equal to the tool the drafted call names — is a *tool-indexed lookup*, whose behaviour under 100 semantically confusable skills mapping to the same tool is exactly the untested case.
- **Different regime.** Jiang's Arm 3 is progressive disclosure over a filesystem directory; Recuris' model-controlled arm injects the whole library into **every turn**. The all-resident regime is a harsher baseline than Jiang's, so the 18-point gap partly measures context bloat (3,111 extra tokens at the first call), not routing alone.
- **A real tension on consequences.** Jiang finds success **flat** under collapsing precision — bad routing costs nothing measurable. Recuris finds bad routing costs 18 points and pushes below the no-skills baseline. Most likely both are right about their own settings: Jiang's task success does not depend on using the exact annotated skill ("exact ground-truth invocation is neither sufficient nor necessary"), whereas τ²-Retail success is gated on issuing specific database writes, where a mis-timed skill directly causes an omitted write.
- **Recuris does not measure routing precision.** Its instruments are required-write recall, omitted writes per episode, and task success. There is no precision-against-ground-truth-annotation number anywhere in the paper, so the claim "state grounding fixes routing precision" is inferred, not measured.

**The strongest joint reading.** Jiang says the bottleneck is agent discipline and confusability rather than pool size; Recuris supplies an external discipline signal — a verified state that says which goals are still open — and shows it is worth ~24 points where the library alone is worth ~2. That is a direct, if small-scale, corroboration of Jiang's implied prescription. The open question both leave standing is whether a *state* key stays discriminative as the library grows, which neither paper tests.

## Where it breaks

- **The "memory" result is mostly a working-state result.** EM alone: +2.0 [−4.0, +7.9] on Retail, +1.5 [−4.5, +7.5] on Airline. EM on top of WM: +1.5 [−2.4, +5.7]. Only the *coupling with invocation control* separates from a no-skills configuration. A reader taking "experiential memory works" from this paper would be taking the wrong claim.
- **τ²-Airline resolves nothing.** With 50 tasks, **every interval in the Airline column of Table 2 includes zero**, including the headline +8.5. The authors say so explicitly and read it as a direction.
- **One deployment model, one memory per benchmark.** Every package is evolved on doubao-seed-2-0-pro alone; the authors defend this as deliberate (a per-model memory would absorb that model's idiosyncrasies) but it means all cross-model transfer numbers come from a single source memory with n=1.
- **Terminal-Bench's headline is the retry budget.** +26.4 of the +26.4 is attempts; the learning term is +2.3 (p=0.774), and every per-attempt interval contains zero.
- **The validation gate is unfalsified rather than validated.** The one run that ignored the gate did as well or better (+14.5 to +18.0). The gate's own dev split (10–14 tasks) has a two-run spread of [−12.1, +11.1] — "finer-grained than it can resolve."
- **Not every round lands, and one failure mode is invisible without the trace.** Run A plateaus at round 3; Run C's round 4 gives back most of its gain; Run B's round-4 package is *never invoked on any of the 86 held-out tasks* — a broken binding masquerading as a diminishing return, detectable only because the kernel counts invocations.
- **Monotone growth.** 51 skills added, 2 revised, **0 deprecated**, 17 near-duplicate pairs admitted. The paper argues this is safe because no single skill carries the gain, but an evolution loop that only ever adds is the shape [[catastrophic-remembering]] warns about; a pruning operator is proposed, not implemented.
- **Component ground truth covers 3 of 4.** Checkers are excluded from the localization benchmark because removing them changes τ²-Retail pass rate by zero — so the one component the loop can attribute to but never had to detect is untested.

## Related

- [[demystifying-agent-skills-jiang-2026]] — the routing-precision collapse (29.6% → 3.3% at pool 5 → 100) this paper's invocation policy is a candidate answer to; see the section above for where they agree and where they conflict on whether bad routing costs anything.
- [[skill-library]] · [[agent-skills]] · [[skill-md-format]] — EM stores skills in the Anthropic agent-skill format; the paper's finding is that the library is worth ~2 points without an invocation signal.
- [[at-skills-attention-is-all-you-have-yin-2026]] · [[is-progressive-disclosure-all-you-need-he-2026]] — the trigger-scarcity and disclosure-mechanism side of the same question; Recuris' answer is to move the trigger out of the model into the harness.
- [[skill-rag-failure-state-aware-retrieval-wei-2026]] · [[compositional-skill-routing-gao-2026]] — the nearest retrieval/routing designs; "state-aware" is the same instinct, and Recuris supplies the state.
- [[memory-management]] · [[memory-evolution]] · [[memory-construction]] — the four-component patch space `(E, W, ρ, C)` is a proposal for what "memory architecture" should mean: not just stored content but the specification, the invocation policy, and the verifier.
- [[procedural-memory]] · [[experiential-learning]] — EM is procedural knowledge; the transfer analysis distinguishes packages that carry *procedure* (SkillFlow, transfers with scale) from those that carry *discipline* (τ², transfers only where the target model's own failures make it worth something).
- [[memharness-memory-is-reconstructed-not-replayed-wu-2026]] — adjacent thesis about what memory is *for* in a harness; both argue the harness's memory layer, not the stored text, is the object of study.
- [[context-as-an-environment-scroll-lin-2026]] — the complementary move: Scroll makes the *history* addressable at query time; Recuris makes the *task state* addressable at invocation time. Both reject "decide from the growing transcript."
- [[agentic-context-engineering]] · [[context-assembly]] — `ρ` is a context-assembly policy that is itself learned; the appendix-D finding (3,111 extra tokens, 18 points worse, 46% more per success) is a direct measurement of assembly discipline.
- [[compaction-cliff-long-running-agent-memory-zerhoudi-2026]] · [[lost-in-compaction-side-constraint-loss-wang-2026]] — the other half of long-horizon memory: those papers ask what survives compaction, this one asks what should be *retrieved* at each step. Recuris sidesteps compaction by keeping WM compact by construction rather than compressing history.
- [[self-improving-agent]] · [[darwin-godel-machine]] · [[godel-machine]] — RSI framing, but deliberately bounded: the base LLM, Meta-Agent, localization, patching and gate are all fixed, and recursion happens only inside `M_k`.
- [[self-harness-harnesses-that-improve-themselves-zhang-2026]] · [[harness-r1-learning-to-edit-runtime-harnesses-shao-2026]] · [[jit-agent-just-in-time-harness-evolution-zhang-2026]] — neighbouring self-modifying-harness designs; Recuris' distinguishing constraint is the component-scoped patch space.
- [[on-the-fragility-of-self-improving-agents-ye-2026]] — the counterweight; Recuris' gate result (rejected candidates regress no more than decode noise; the ungated run did fine) is evidence the fragility is milder here than feared, over the horizon tested.
- [[agentic-harness-engineering]] · [[same-task-different-work-weinberger-2026]] — the cost side: 147k vs 101k agent tokens per success between two configurations of the *same* harness carrying byte-identical skills.
- [[verifier]] · [[reflection-mechanism]] · [[reflexion]] — the checker set is a verifier used as a *state-transition guard* rather than an outcome grader; the truth-guard result (172 rejections, zero measurable effect) is a sharp negative for post-hoc reflection on irreversible actions.
- [[continual-learning]] · [[catastrophic-remembering]] — 51 skills added and 0 deprecated across eight patches is exactly the accumulation pattern; the redundancy ablation is the defence.
- [[test-time-compute]] · [[test-time-memory]] — §3.5's adaptation mode, where the honest finding is that the attempt budget, not the learning, carries the headline.
- [[terminal-bench]] · [[terminal-bench-benchmarking-agents-cli-merrill-2026]] — Terminal-Bench 2.1 (87 tasks, Terminus-2 agent) is the isolated-task regime where cross-task evolution admitted no patch in thirteen runs.
- [[agent-benchmark]] · [[agent-evaluation]] — the statistical protocol is unusually careful: paired task-clustered bootstrap, 10,000 resamples, an A/A byte-identical re-run to establish instrument resolution.
- [[claude-code]] · [[deepseek]] — the two independent Meta-Agent stacks; swapping them moves the result by less than run-to-run noise.
- [[llm-agent]] · [[tool-use]] · [[function-calling]] — the deployment setting; τ² tasks succeed only on full environment reward, so verbal agreement with the user without the database write is a failure.

> [!note] Concept gaps this page could not link
> The wiki has no **working memory** page (the paper's central construct; nearest existing pages are [[long-term-memory]], [[episodic-memory-llm]] and [[memory-stream]]), no **experiential memory** page distinct from [[experiential-learning]], and no **[[skill-routing|skill routing]] / skill invocation** page (already flagged as needed on the [[demystifying-agent-skills-jiang-2026]] page — this paper strengthens the case, since it names invocation a first-class evolvable component). No pages for **τ²-Bench** or **SkillFlow**.

## References

- [arXiv:2608.24876](https://arxiv.org/abs/2608.24876)
- Barres et al. 2025, *τ²-Bench* — retail (114 tasks) and airline (50 tasks) dual-control tool-use domains.
- Zhang et al. 2026c, *SkillFlow* — 166 tasks in 20 families sharing an execution flow; programmatically verified.
- Shaw et al. 2026, *Terminal-Bench 2.1* — 87 terminal tasks, Terminus-2 agent, official Docker environments.
- Anthropic 2025 — the agent-skill format that Experiential Memory stores skills in.
- Yang et al. 2025 (*Qwen3*); IBM Granite Team 2026 (*Granite*); OpenAI 2025 (*gpt-oss*) — the open-weight target families.
