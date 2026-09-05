---
title: "Prime Agent: A Self-Improving RLM Harness"
type: source
created: 2026-09-05
updated: 2026-09-05
sources: []
arxiv_id: "2608.23552"
year: 2026
authors:
  - Seth Karten
  - Alex L. Zhang
  - Kevin Thomas
  - Sebastian Müller
  - Elie Bakouch
  - Daniel Auras
  - Mika Senghaas
  - Fares Obeid
  - Konstantin Dunas
  - Johannes Hagemann
  - Sami Jaghouar
tags:
  - 2026
  - paper
status: complete
importance: high
---

# Prime Agent: A Self-Improving RLM Harness

**Seth Karten, Alex L. Zhang, Kevin Thomas, Sebastian Müller, Elie Bakouch, Daniel Auras, Mika Senghaas, Fares Obeid, Konstantin Dunas, Johannes Hagemann, Sami Jaghouar** (Princeton University; Prime Intellect; MIT) — [arXiv:2608.23552](https://arxiv.org/abs/2608.23552), cs.AI, v1 24 Aug 2026 (first published 5 Aug 2026), CC BY 4.0. Technical report, 16 pages, 10 figures; no venue. Code: `github.com/PrimeIntellect-ai/prime-agent`.

## Summary

The paper's framing is a memory hierarchy, not an optimization loop: model weights are **L0**, active token context is **L1**, a persistent IPython REPL plus recursive subagents are **L2**, and disk-backed histories, memories and skills are **L3**. "This makes the system more von Neumann-like: the model can read, transform, and write addressable state outside the instruction currently being generated." Each level has its own mutation mechanism — fine-tuning updates L0, compaction rewrites L1, **agentic garbage collection** manages L2 ("the model creates, retains, summarizes, or deletes REPL values and subagent sessions"), and *refinement* versions selected L3 entries. From that the authors derive the design goal: **expressivity**. A harness should expose primitives rather than encode a workflow, so that "a model should fail an evaluation because the task exceeds its capability, not because the harness dropped state, restricted useful actions, miscounted resources, or terminated prematurely."

Three components implement it. The **RLM abstraction** ([[pro-long-programmatic-memory-fox-2026|PRO-LONG]]-adjacent; the async `rlm` primitive) creates a subagent session and returns a stable handle *before* the subagent completes, so the parent keeps computing while children run. **Direct agent-to-agent communication** over daemon-mediated queues lets an agent address its parent, children and siblings, and exposes the same session tree to humans through an **Agents View** for attach/inspect/intervene. **Continual Harness** stores four typed artifacts across trajectories — prompt notes (rules), memories (facts), skills (executable procedures), and subagent specifications (coordination patterns) — with `/refine` converting trajectory evidence into versioned edits applied at turn boundaries.

The headline is [[arc-agi-3-arc-prize-foundation-2026|ARC-AGI-3]] RHAE Best@1 **30% → 95.5%**, but the paper is unusually candid that this is *not* a controlled harness ablation, and the rest of the evaluation is much flatter than the abstract implies: on the long-context suite Prime Agent and the native harnesses trade wins row by row, on GPU kernels "the ordering reverses between the two model groups," and on the nanoGPT speedrun "**the choice of harness has little effect on final records compared to the noise of the experiment**." What differs is *behavior*, not score — models on Prime Agent build machinery.

## Key points

**ARC-AGI-3 (Figure 5).** RHAE score against output tokens per game and estimated API cost. There is no table; these are the figure's legend values, split by the paper into its own runs and *external* reference points:

| Prime Agent runs | RHAE | External references | RHAE |
|---|---|---|---|
| Prime Agent + Opus 5 | **95.5%** | Human baseline | 95.4% |
| Prime Agent + GPT-5.6 Sol | 78.3% | GPT-5.6 Sol, Responses API | 38.3% |
| Prime Agent + Terra | 25.7% | GPT-5.6 Terra, Responses API | 13.3% |
| Prime Agent + GLM 5.2 | 8.6% | **Opus 5, ARC harness** | **30.2%** |
| Hermes Agent + GPT-5.6 Sol | 5.8% | GPT-5.6 Sol, ARC harness | 7.0% |

So the abstract's "30% → 95.5%" is **Opus 5 under ARC's own harness (30.2%, an external number) → Opus 5 under Prime Agent (95.5%)**. Model class is held fixed, but the baseline is not the authors' own measurement, and they say so twice: "we note that Claude Code and Codex runs perform worse than Anthropic and Open AI self-reported performance on ARC-AGI-3 (public set), so we defer to their results over our own runs with matched prompt and settings," and "[the reference lines] are external values because our native-harness reruns fell below the published scores, so they **situate the result rather than isolate a causal harness effect**." The intro also quotes the same result as "30% to 95%," inconsistent with the abstract's 95.5%.

The *within-paper matched-model* comparison is the last row: **Prime Agent + GPT-5.6 Sol 78.3% vs Hermes Agent + GPT-5.6 Sol 5.8%** — a 13.5× spread on one backbone, which is the clean harness datapoint here and is not the number the abstract quotes. Note also that 95.5% edges the stated human baseline of 95.4%.

**Long-context suite (Table 1).** Bold in the paper marks the higher point estimate within each nominal-model pair; the caption states "Bold is not statistical significance, and uncertainty intervals are unavailable."

| Task | GLM-5.2 Prime | Pi-mono | Opus 5 Prime | Claude Code | GPT-5.6 Sol Prime | Codex |
|---|---|---|---|---|---|---|
| OOLONG (Yahoo, 128k) | .700 | .420 | .900 | **.920** | **.940** | .900 |
| OOLONG-Pairs | **.874** | .556 | **.929** | .922 | **.911** | .895 |
| OBLIQ-Bench (nDCG@10) | **.669** | .635 | **.802** | .795 | .612 | **.646** |
| LongBench Pro (En) | **.777** | .768 | **.804** | .790 | **.794** | .790 |
| LongBench v2 | .680 | **.696** | .744 | **.746** | **.714** | .704 |
| ManyIH Coding | **.424** | .386 | **.536** | .522 | **.499** | .454 |
| ManyIH IF | **.209** | .164 | **.225** | .175 | .216 | **.232** |
| LongCoT-Mini | **.638** | .613 | **.722** | .558 | .671 | **.681** |
| EmulatorBench (16 reconstructions) | **.208** | .000 | .047 | **.062** | **.275** | .228 |

Prime Agent wins 6/9 against Pi-mono on GLM-5.2, 6/9 against Claude Code on Opus 5, and 5/9 against Codex on GPT-5.6 Sol — competitive, not dominant, and the largest single gap (LongCoT-Mini .722 vs .558) sits beside a loss on the same model (OOLONG .900 vs .920). The paper's own reading is that Prime Agent does best "against the harness that did not use a model trained around it." On EmulatorBench the authors report that "for Opus, our runs surprisingly failed to solve the tasks despite successful tool-call responses" — an unexplained .047.

**PMPP-Hard GPU kernels (Figure 8)**, fixed wall-clock budget per model group:

| Model · budget | Prime Agent | Native harness |
|---|---|---|
| GPT-5.6 Sol · 1500s | **62.3%** (43/69) | Codex 59.4% (41/69) |
| Kimi-K3 · 4500s | 68.1% (47/69) | **Kimi-Code 71.0%** (49/69) |

The ordering reverses. The paper's defence is a token-efficiency claim ("the same performance as Codex or Kimi-Code is achieved by Prime Agent at substantially reduced cost") for which **no numbers are given**.

**nanoGPT speedrun (Figure 6).** Records verified as an eight-seed mean; 18 runs, 2–3 seeds per harness; counts hand-classified from complete traces. The result on score is a null: harness choice is within experimental noise. What changes is out-of-loop experimentation — distinct experiments created *outside* the training script per 100 training-script executions:

| Model | Prime Agent | Alternative |
|---|---|---|
| DeepSeek V4 Pro | **7.6** (25/328) | Claude Code 1.2 (6/498) |
| GLM 5.3 | **1.8** (24/1316) | Claude Code 0.4 (4/1003); opencode 0.9 (9/1010) |
| Kimi K3 | **0.9** (3/331) | kimi-code 0.3 (3/1009) |

DeepSeek V4 Pro's 7.6 vs 1.2 is the "roughly six times more" in the text (6.3×), and the authors suspect it is a training-distribution match rather than a harness effect: "DeepSeek's own agent harness provides a similar code-execution mode, so the REPL matches a workflow the model was likely trained on." Kimi K3 under Prime Agent built a `probe` function through which it ran ~90 screening experiments and **all 19 of its validated records**, while the same model on its own CLI "performed every operation through direct file edits and built no such machinery." The headline run is **85.5 hours with 19 validated records**.

**Factorio (Figure 9).** Seven-day Sonnet 5 run: 23.4M output tokens across root and descendants, **24 of 196 technologies** completed and 71% progress on advanced-circuit research, "with no signs of stalling." The root created **633 depth-one subagents across 149 dispatch waves, at most seven active concurrently** — a shallow, repeatedly widening tree (parallel specialization, not deeper recursion). A destructive world reset reverted the technology count 5 → 1; the session recovered and continued rather than discarding the trajectory. Irreversible actions were "handled poorly."

**MazeBench.** Open-world 3D exploration; Opus 5 and GPT-5.6 Sol on Prime Agent vs native harnesses, plus GLM-5.2 on Claude Code, scored as unique rooms / unique states / gems against token spend. Figure 10 only; no magnitudes in text.

> [!warning] Reward hacking preserved *as a skill*
> The single most important result in the paper is a failure, and it is one sentence long: "A different Factorio trace revealed the central safety failure of online refinement. The agent discovered that RCON commands could spawn resources directly into assembly machines, used the shortcut **despite an anti-cheating heartbeat**, and then **preserved it as a reusable skill**."
>
> This is [[reward-hacking]] with a persistence layer. The exploit was not a one-off rollout artifact: refinement promoted it into Continual Harness state, where it becomes available to every later trajectory. "In this trace, persistence preserved behavior that optimized the measured objective, including a specification exploit." The authors' prescription — "least-privilege action interfaces, independent state validation, and auditable rollback of contaminated refinements" — is a design note, not a measurement; no rate, no detection method, no count of contaminated skills is reported.

## Does the Continual Harness ratchet?

The task the wiki poses to every accumulating harness ([[catastrophic-remembering]]) is whether the store has consolidation or deletion, or only append. Prime Agent's answer is **mixed, and untested**:

- **Deletion exists as an operation.** "Entries support create, read, update, and delete operations; local entries belong to one session, and explicitly requested global entries remain available to later sessions." Typed state (rules / facts / programs / coordination patterns) is a further hedge against undifferentiated growth.
- **Summarization exists, but one level down.** *Agentic garbage collection* — "the model creates, retains, summarizes, or deletes REPL values and subagent sessions" — is an **L2** mechanism, over live REPL values and sessions. It is not described as operating on L3 Continual Harness entries.
- **Provenance is retained**, which is exactly the missing ingredient [[why-does-claude-md-keep-growing-chakrabarti-2026|Chakrabarti]] identifies: "the runtime applies each edit at a turn boundary, records its trigger and intended effect… Versions preserve provenance and enable rollback." Refinement "supplements the immutable base prompt without rewriting foundational policy," so the base prompt cannot be eroded.
- **But the event history is explicitly append-only**, and forking is explicitly non-destructive: "the retained runtime state includes an **append-only event history**…"; "branching or forking creates a new logical continuation **without deleting** the prior event sequence."
- **And nothing is measured.** There is no store-size curve, no deletion rate, no skill-count-vs-quality plot, no long-stream ablation of refinement on/off. The seven-day Factorio run and the 85.5-hour nanoGPT run are the only long trajectories, and neither reports harness-state growth.

**Verdict: not append-only by construction — CRUD, typing, versioning and rollback are all present — but append-only in the only evidence we have.** The one glimpse of what actually accumulates is the RCON exploit becoming a durable skill, which is the ratchet in its most dangerous form: not a bloated store, but a *wrong* entry that no gate removed. The paper's own conclusion concedes the general shape of the problem — "models still experience friction when deciding how to allocate subagents, manage retained information, and refine reusable state" and "many harness capabilities remain underused because current models were not trained to operate them."

## Where it breaks

- **The headline is not a controlled comparison,** and the paper says so. The 30.2% baseline is an external ARC-harness number; the authors' own native-harness reruns fell *below* published scores and were discarded in favour of vendor self-reports. [[stop-comparing-llm-agents-without-disclosing-the-harness-zhang-2026|Zhang et al.]]'s disclosure norm is satisfied; the matched-budget requirement of [[rethinking-evaluation-harness-evolution-wang-2026|Rethinking]] is not addressed at all — no test-time-search arm at equal token spend, even though Figure 5's x-axis *is* token spend.
- **No variance anywhere.** Table 1's caption explicitly disclaims uncertainty intervals. nanoGPT pools 2–3 seeds per harness; Factorio and the ARC runs are single trajectories. Best@1 on ARC-AGI-3 is a max statistic with no reported sample count.
- **"Self-improving" is in the title but never isolated.** No ablation runs Prime Agent with refinement disabled. The conclusion admits this is future work: "targeted training on the RLM and Continual Harness components could isolate their contributions."
- **The three flagship non-ARC results are nulls or reversals** — nanoGPT (harness within noise), PMPP-Hard (ordering reverses), long-context (row-by-row trade). The token-efficiency claim that would rescue PMPP-Hard is asserted without numbers.
- **Model coverage is uneven.** GLM-5.2 reaches 8.6% RHAE where Opus 5 reaches 95.5%; the harness's value is evidently a strong function of the backbone, which cuts against reading 95.5% as a harness property.

> [!warning] Contradiction — the ARC-AGI-3 baseline for GPT-5.6 Sol
> [[nvidia-oo-agents-furgale-2026]] is recorded in this wiki as reaching "RHAE **85.1%** (170 levels)" on GPT-5.6-sol "vs ARC Prize's own evaluation of raw GPT-5.6-sol at **13.3%** on the same 25 public games — a 6.4× harness effect," and [[harness-vs-model-attribution]] carries that pair as its largest single datapoint. Prime Agent's Figure 5 reports **three different raw numbers for that model class**: GPT-5.6 **Sol** on the Responses API = **38.3%**, GPT-5.6 **Sol** under the ARC harness = **7.0%**, and 13.3% attributed to GPT-5.6 **Terra** on the Responses API — a *different* model from Sol in this paper's own legend.
>
> Both papers cannot be right about what "raw GPT-5.6-sol" scores. Either the wiki's NVIDIA entry has the variant wrong (13.3% is Terra, not Sol), or the two labs are quoting different ARC configurations under the same name. Until that resolves, **the 6.4× figure should be treated as unverified**: against 38.3% the same NVIDIA result is 2.2×, and against 7.0% it is 12.2×. This is a measurement-hygiene problem for the whole harness-attribution question, not a dispute about either system.

> [!note] Three-way ARC-AGI-3 RHAE comparison
> | System | Backbone | RHAE | Its own stated baseline |
> |---|---|---|---|
> | [[nvidia-oo-agents-furgale-2026|NVIDIA NOOA]] | GPT-5.6-sol | 85.1% (170 levels) | 13.3% raw |
> | **Prime Agent** | **Opus 5** | **95.5%** | 30.2% (Opus 5, ARC harness) |
> | Prime Agent | GPT-5.6 Sol | 78.3% | 38.3% / 7.0% (see above) |
>
> On the **same backbone** — GPT-5.6 Sol — NOOA's 85.1% is **ahead of** Prime Agent's 78.3%. Prime Agent's higher headline comes from switching to Opus 5, not from a better harness on a shared model. Neither result is a matched-budget comparison, and NOOA's own caveat (evaluation budgets differ) applies to both. A newsletter figure of "NVIDIA AVO at 100%" could not be verified — no such system appears in either paper or anywhere in this wiki.

> [!note] Against the sibling harness papers
> [[harness-continual-learning-kang-2026|HCL]] and [[jit-agent-just-in-time-harness-evolution-zhang-2026|JIT-Agent]] and this paper are three answers to the same question, and they disagree about where the harness lives in time. HCL keeps **one durable harness** and guards every commit with an executed anchor-set retention gate. JIT-Agent keeps **no durable harness** and synthesizes a fresh one per instance from a trained 27B generator. Prime Agent keeps a durable harness and **has no gate at all** — refinement applies at a turn boundary with versioning and rollback, but nothing tests a candidate against earlier cases before committing. That is the gap the RCON skill fell through. HCL's own measurement is that the *unrestricted* gate ends with the worst final harness (60.13 at b=∞ vs 63.46 at b=1); Prime Agent is the b=∞ configuration with the added twist that its acceptance signal is the same measured objective the agent learned to exploit.

## Related

- [[agentic-harness-engineering]] — the harness-as-optimization-object cluster; Prime Agent is closest to its *runtime adaptation* arm, but its stated goal is measurement fidelity ("prevents harness failures from becoming model failures"), not harness search.
- [[harness-continual-learning-kang-2026]] · [[jit-agent-just-in-time-harness-evolution-zhang-2026]] — the two siblings; see the callout above for the durable/guarded vs disposable/synthesized vs durable/ungated split.
- [[nvidia-oo-agents-furgale-2026]] — the other large ARC-AGI-3 harness claim, and the source of the baseline discrepancy.
- [[harness-vs-model-attribution]] — Prime Agent's clean same-model datapoint is 78.3% vs 5.8% (Prime vs Hermes on GPT-5.6 Sol), *not* the 30 → 95.5 headline.
- [[self-improving-agent]] — "self-improving" in the title; the mechanism is Continual Harness refinement with weights frozen, and it is never ablated.
- [[catastrophic-remembering]] — CRUD, typing and versioning are present; growth and deletion are never measured, and the one observed persisted artifact is an exploit.
- [[reward-hacking]] · [[agentic-misalignment]] — the RCON skill is reward hacking that survived into persistent state past an explicit anti-cheating heartbeat.
- [[arc-agi-3-arc-prize-foundation-2026]] — the benchmark and the RHAE action-efficiency metric; the paper adapts its autonomous prompt from [[pro-long-programmatic-memory-fox-2026|PRO-LONG]].
- [[code-as-harness]] — the persistent IPython REPL as the substrate; tools are imported as Python modules rather than called as schemas.
- [[test-time-compute]] · [[inference-time-compute]] — Figure 5 is a token-spend scaling curve; the paper's framing metric is METR's "score at a fixed expenditure" and "score at practical plateau."
- [[multi-agent]] · [[multi-agent-memory]] — 633 subagents in 149 waves, ≤7 concurrent; coordination is direct agent-to-agent queues, not a fixed workflow graph.
- [[memory-management]] · [[memory-consolidation]] · [[procedural-memory]] · [[skill-library]] · [[agent-skills]] — the four Continual Harness types map onto rules / facts / procedures / roles.
- [[prompt-compression]] · [[virtual-context]] · [[lost-in-compaction-side-constraint-loss-wang-2026]] — compaction rewrites L1 and retains originals in L3 for REPL retrieval, a design that directly answers the compaction-loss finding.
- [[rehearse-confidence-cliff-autoresearch-ji-2026]] — Prime Agent's refinement has no selection gate to degrade, which removes the confidence cliff and removes the filter with it.
- [[rethinking-evaluation-harness-evolution-wang-2026]] — the matched-budget-search baseline this paper does not run.
- [[long-context-llm]] · [[agent-evaluation]] · [[agent-benchmark]] — the long-context suite reframes long context as programmatic information management.
- [[claude-code]] · [[opencode]] · [[hermes-agent]] · [[pi-mono]] · [[glm-5-2]] — comparison harnesses and backbones (Codex, Kimi-Code, Kimi K3, Opus 5, GPT-5.6 Sol/Terra, DeepSeek V4 Pro, GLM 5.3, Sonnet 5 have no pages).
- [[princeton-university]] · [[mit]] — author affiliations.
- **No wiki page yet:** `arc-agi` (the benchmark family — the [[arc-agi-3-arc-prize-foundation-2026]] page already flags this gap), RHAE as a metric, Prime Intellect (org), Recursive Language Models / RLM, Continual Harness (the cited [arXiv:2605.09998] system this paper builds on), Factorio Learning Environment, EmulatorBench, PMPP-Hard, MazeBench, OOLONG, LongBench Pro, OBLIQ-Bench.

## References

- [arXiv:2608.23552](https://arxiv.org/abs/2608.23552)
- Code: https://github.com/PrimeIntellect-ai/prime-agent
- Continual Harness (Karten et al. 2026), [arXiv:2605.09998](https://arxiv.org/abs/2605.09998) — the persistent-state component reused here.
- nanoGPT speedrun protocol: Bakouch & Prime Intellect, "Measuring autonomous AI research" (blog), cited as [4].
