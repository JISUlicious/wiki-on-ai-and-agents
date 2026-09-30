---
title: "Training AI Scientists to Replicate Research"
type: source
created: 2026-09-05
updated: 2026-09-05
sources: []
arxiv_id: "2608.13331"
year: 2026
authors:
  - Damon Falck
  - Samer Sabri
  - Anja Surina
  - Thom Foster
  - Anya Sims
  - Sam Devlin
  - Dylan Rogers
  - Tantum Collins
  - Kaloyan Aleksiev
  - Louis Kirsch
  - Edward Hughes
tags:
  - 2026
  - paper
  - ai-for-science
status: complete
importance: high
---

# Training AI Scientists to Replicate Research

**Damon Falck\*, Samer Sabri\*, Anja Surina, Thom Foster, Anya Sims, Sam Devlin, Dylan Rogers, Tantum Collins, Kaloyan Aleksiev§, Louis Kirsch†, Edward Hughes†** (Inherent; \* equal first, § infrastructure lead, † equal last) — [arXiv:2608.13331](https://arxiv.org/abs/2608.13331), cs.LG / cs.AI, 13 Aug 2026 (v1, dated 14 August 2026 on the title page). arXiv perpetual non-exclusive license; no venue given. Contact: faraday@inherentlaboratories.com. No code, weights, or task-space release is announced.

## Summary

The thesis is that paper replication is the right *training* task for an [[llm-agent]] that is supposed to eventually do science, because replication has the shape of research — a paper "lossily compresses" the work that produced it, so filling in the missing details requires the same hypothesis-driven exploration as open-ended research — while still being gradeable. The three obstacles the authors name are that replication is underspecified by construction, that frontier agents are post-trained for closed-ended verifiable problems, and that hill-climbing harnesses (autoresearch, AlphaEvolve) do not apply because there is no test-time reward to climb. Their answer is to move the capability into weights rather than into scaffolding: **Faraday** is a 27B model post-trained with RL to *direct* a frontier coding agent, and the paper's slogan for this is **coding agent as a tool (CAT)**.

Three artifacts carry the work. **Replica** is an auto-generated task space: 310 figure-replication tasks (242 train / 68 test) mined from 100 well-known ML and AI-for-science papers spanning 1990–2026. **The rubric judge** is the reward: a per-task rubric auto-written by Claude Opus 4.7, applied by Codex GPT-5.5 running *inside the agent's own container* with permission to re-execute the agent's code. **The recipe** is a modified [[grpo]] with three stability devices — multi-sample judge aggregation, judge-emitted turn-level credit weights, and a staged curriculum — applied as [[lora]] adapters on Qwen3.6-27B.

The headline is that Faraday beats both Claude Opus 4.8 and Codex GPT-5.5 on **73% of in-distribution ML tasks and 60% of held-out AI-for-science tasks**, for a mean **+6% over Claude and +8% over Codex on the test split**. The qualitative claim is more interesting than the quantitative one: across hand-inspected high-margin rollouts, Faraday *implements the mechanism the figure is designed to test* where baselines hard-code the expected output — it runs the Darwin-Gödel evolutionary search instead of pasting in a "discovered" agent, runs a real skill-acquisition phase for [[voyager]] instead of pre-populating the [[skill-library|skill library]], and refuses its own coding agent's attempt to hand-craft an LSTM in place of training one.

> The comparison against the frontier is **not matched-harness**. See [Is the frontier comparison fair?](#is-the-frontier-comparison-fair) — this is the single most load-bearing caveat on the headline number.

## Replica: the task space

Each task hands the agent a paper PDF **with one results figure irreversibly redacted**, that figure's caption, a 60-minute wall clock, and a **single one-seventh MIG slice of an H200**. The agent works in a containerd container with research libraries pre-installed and internet access. It never sees the original plot; the judge does.

Construction is fully automatic in three vision-language stages powered by Gemini 2.5 Pro — scan for every main-text results plot and caption, localise a bounding box inside an LLM-verifier repair loop, then redact — yielding a triple of (caption, extracted "gold plot", redacted paper). Every task was then hand-inspected and filtered for insufficient redaction, non-results plots, and misattributed captions. Each paper contributes **1 to 13 tasks, median 2**.

| | Train split | Test split |
|---|---|---|
| Tasks | 242 | 68 |
| Domain | ML papers | AI-for-science papers |
| Years | 1990–2026 | 2012–2026 |

The scale-down requirement is deliberate, not a concession: where the original experiment cannot fit the budget, the prompt asks for "the most faithful scaled-down version of the underlying experiment." The authors argue this *adds* underspecification and therefore teaches the decision-making that open-ended research demands. Papers were chosen for fame so that human raters could judge them, and the authors are explicit that this makes pre-training contamination likely; their defence is that the **process** data that produced each figure was never recorded anywhere, so it cannot be memorised, and that redaction decontaminates the agent's context and makes cheating (e.g. downloading the original plot) easier for the judge to catch.

## How replication is scored

This is the paper's core mechanism, so it is worth stating precisely.

**Rubric generation.** From a short hand-written meta-prompt, **Claude Opus 4.7** writes a task-specific rubric. Two hiding decisions matter: the gold plot is hidden *from the rubric generator*, so the rubric encodes the paper's claims rather than axis ranges and formatting; and the rubric is hidden *from the agent during training*, so the policy cannot game criteria it never sees. The generator is instructed to reward faithful scale-downs.

**The five rubric dimensions** — each criterion scored continuously in [0,1], all averaged into one overall score:

| # | Dimension | What it asks |
|---|---|---|
| 1 | Visual fidelity | How closely the replicated figure matches the paper's |
| 2 | Claim reproduction | How well the replication supports the paper's scientific claim |
| 3 | Implementation fidelity | Whether the experiment actually implements and tests what the paper describes |
| 4 | Experimental depth | Whether the agent makes good use of its compute budget |
| 5 | Scientific integrity | Whether the agent followed instructions and did not cheat |

**Coding agent as a judge.** The grader is **Codex GPT-5.5**, given the *same workspace and compute as the agent*: the redacted paper, every tool the agent had, the replication codebase and git history, the full rollout trace, and — unlike the agent — the gold plot. It gets **10 minutes** to explore. The point of using a coding agent rather than a text judge is that it can **re-execute the agent's code** to check whether the reported result is robust and real. During training the judge is sampled **three times per rollout** and the scores averaged.

**Does the judge track human taste?** Partly, and the paper is honest that the evidence is weaker than the abstract implies. Against a baseline judge (same model, one constant prompt across tasks, mirroring the human instructions), on ten deliberately *maximally disputed* tasks, measured by Kendall τ:

| Pair | Kendall τ |
|---|---|
| Rubric judge vs. rubric judge (two draws) | **0.66** |
| Baseline judge vs. baseline judge | 0.46 |
| Human vs. human | **0.30** |
| Rubric judge vs. human | **0.19** |
| Baseline judge vs. human | 0.15 |

So the rubric judge is far *more self-consistent than humans are with each other*, and slightly more human-aligned than the baseline judge — but on disputed pairs humans side with the rubric judge on only **63%**, which is above chance and **not significant (p = 0.109)**, from 76 rankings by 19 participants. On noise, **eight baseline-judge samples are needed to reach the noise share obtained with three rubric-judge samples**, which is the actual justification for using it as a GRPO reward.

Raters were PhD students or PhDs from top universities, preferentially with an ICML/ICLR/NeurIPS main-track publication, paid £150 per task with £125 bonuses at the fifth and eighth task; **117 rankings from 20 participants** were collected across both studies, blind to model identity.

## The training recipe

Base model **Qwen3.6-27B**, trained with [[lora]] (rank 128, α = 128, adapters on all linear projections), 128K context, constant LR 6×10⁻⁶, Adam. Each optimiser step draws **10 tasks × 8 rollouts** from the 242-task train split, sampled so every batch spans the corpus year range evenly and each epoch visits every task exactly once — "no single era of science dominates any update."

Not vanilla [[grpo]]: leave-one-out group-relative baseline, DAPO token-level loss with asymmetric clipping (ε_ℓ = 0.15, ε_h = 0.35), IcePop token-level discrepancy masking (zeroing tokens whose sampler–trainer likelihood ratio falls outside [0.3, 4]), and a small KL to base with β = 3×10⁻³. Rollouts are generated asynchronously with staleness capped at 3–6 optimiser steps; bf16 throughout after fp8 proved less stable.

**Turn-level credit assignment** is the novel piece. The judge emits a weight distribution u_k over the rollout's turns, normalised so that Σ_k u_k n_k = Σ_k n_k (n_k = tokens in turn k) — i.e. credit is *redistributed within* a rollout without changing the update's overall magnitude — averaged turn-wise over the three judge draws, then used to scale the per-token GRPO advantage. Over 577,585 turns, credit concentrates in the **early-to-middle** stage of a rollout and lands disproportionately on **turns that delegate to the coding agent**.

**Five-stage lineage** to the final step-659 checkpoint:

| Stage | Steps | Delta |
|---|---|---|
| I | 1–100 | fp8 rollouts, GPT-5.4 mini as coding tool, 30-min horizon, LR 3×10⁻⁶, 4 steps off-policy, 1 judge sample, uniform credit |
| II | 101–382 | bf16 rollouts, LR 6×10⁻⁶, 3 steps off-policy |
| III | 383–489 | GPT-5.5 as coding tool, 60-min horizon |
| IV | 491–635 | 3 judge samples, turn-level credit assignment |
| V | 636–659 | fixes to task captions in 17% of tasks |

Infrastructure: Kubernetes (Hopper + Blackwell) with Kueue quotas, Ray jobs, a fork of NeMo-RL, Megatron-Core trainer with tensor + context parallelism over the full 128K context, [[vllm]] inference with policy weights refit in place so in-flight rollouts continue on newer weights, and a fork of NeMo-Gym for rollout collection.

## The CAT paradigm

Faraday's harness is deliberately thin — five function-calling tools (`apply_patch`, `read_file`, `list_dir`, `grep_files`, `shell`), a subset of the Codex CLI schema reimplemented in Python, a **linear append-only conversation with no compaction**, concurrent tool calls within a turn, and a 16K per-turn token limit whose overflow simply ends the rollout (the partial rollout is judged like any other). The stated design principle: *"Faraday's scientific capabilities are improved by changing its policy weights, not by complexifying its harness."*

The coding agent is reached through a `coding_agent.py` wrapper that runs the Codex CLI non-interactively, resuming the previous session by default (or `--fresh`, or several in parallel), with a Faraday-configurable per-request deadline that returns a partial transcript on expiry and a queryable output-token budget. The system prompt is blunt: *"You are a researcher, not a coder… You must NOT write code yourself — always delegate implementation to the coding agent."*

The striking part is the size inversion: a **27B** model directing one with **an estimated 5T parameters**, and beating that larger model alone. The authors frame this as an oversight result as well as a capability one — a weaker, open-weights, inspectable model successfully supervising a stronger closed one — while noting nothing in CAT requires the outer agent to be the smaller one. Two supporting results: the CAT ablation ("Faraday Coder", same recipe with no coding-agent tool) **collapsed after ~300 steps** and was consistently worse at equal step count *despite having double the time horizon* (60 vs 30 min); and a checkpoint trained only with GPT-5.4 mini scores **higher when its tool is swapped to GPT-5.5 at evaluation**, so the outer agent is not specialised to its train-time tool and should compound with frontier progress.

## Results

**Main comparison** (mean over 8 rollouts per task per agent; baselines are Claude Opus 4.8 in [[claude-code|Claude Code]], GPT-5.5 in Codex, GLM-5.2 in Claude Code, all at max/extra-high thinking effort):

| Claim | Value |
|---|---|
| Faraday beats both Claude and Codex, in-distribution (ML) | **73% of tasks** |
| Faraday beats both Claude and Codex, out-of-distribution (AI-for-science) | **60% of tasks** |
| Mean improvement over Claude Opus 4.8, test split | **+6%** |
| Mean improvement over Codex GPT-5.5, test split | **+8%** |
| Strongest baseline | Claude Opus 4.8 |
| Weakest agent overall | Qwen3.6-27B (Faraday's own base model, same harness) |

The paper reports these as distributional shifts — the fraction of tasks scoring at least σ, on a march-of-nines axis — not as single scalars, and the absolute mean rubric scores live only in figure legends. Faraday's curve lies above every baseline at every threshold in-distribution and across almost the whole range out-of-distribution, with a *thinner weak tail* as well as a higher mean.

**Where the advantage comes from** (rubric decomposed): Faraday is stronger on **experimental depth, claim reproduction, and visual fidelity**, and **merely matches Claude on implementation fidelity and scientific integrity**. The integrity result is worth flagging against the paper's own qualitative narrative about Faraday not cheating — quantitatively that dimension is a tie.

**Difficulty structure.** Score declines with publication year for *every* agent (the authors speculate: thinner pre-training coverage, plus higher original compute making the scale-down decision harder). NLP and LLM papers are hardest; classical ML and statistics easiest; AI-for-science harder than ML across the board. Faraday's untrained base degrades with recency fastest of all.

**Prompting control.** 24 generations of automated prompt optimisation on the Codex baseline — each generation sampling 10 train tasks × 8 rollouts, with Claude Opus 4.8 rewriting the prompt from all prior rollouts *including judge feedback* — produced **no meaningful improvement**, so the gap to Faraday survives. The optimised prompt correctly names the failure modes and still fails to fix them.

**Ablation: turn-level credit.** Removing it from a late checkpoint causes **collapse within ~50 steps**: carry-forward mean reward falls, token entropy spikes then collapses, and the Jensen–Shannon divergence between generation and training policy grows by **two orders of magnitude** — which the authors report as a common precursor to such collapses.

**Human agent-comparison study.** On 41 triplets where the rubric judge puts Faraday **≥0.2 above both baselines** on its 0–1 scale, blind expert rankings from 11 participants prefer Faraday over Claude in **80%**, over Codex in **88%**, and above both in **71%** (29/41), all p < 0.01. The paper is careful that this design supports only a *conditional* claim — it says humans agree when the judge sees a clear Faraday advantage, and says **nothing about Faraday's average standing against Claude or Codex in human eyes**.

**Generalisation beyond the training regime.**

| Test | Setup | Outcome |
|---|---|---|
| Full-scale replication | 8 held-out tasks (5 AI-for-science, 3 ML), up to **8 hours and 8 B300 GPUs** as estimated by Claude Opus 4.8; Faraday's context raised 128K→256K; **1 rollout each** | Faraday beats Claude on average and on **5 of 8** tasks. 3 of the 8 papers post-date every model's knowledge cutoff |
| Stronger tool | Faraday checkpoint trained only with GPT-5.4 mini, evaluated with GPT-5.4 mini vs GPT-5.5 | Scores improve with the stronger tool, no retraining |
| "Innovation" | 20 counterfactual variants of 10 papers (5 train, 5 test) generated by Claude Opus 4.8 — one swaps the dataset/environment, one changes the claim | Judge prefers Faraday to Codex on **19 of 20** |

**Within-rollout behaviour.** On nine randomly sampled test tasks, post-hoc scoring of the best-so-far plot over elapsed time shows the same accumulating "moments of insight" curve that evolutionary AI-Scientist harnesses produce — except Faraday has no hand-coded evolutionary harness, does not modify its harness at test time, and **has no access to the reward at test time**. The authors read this as the agent having learned to value insight intrinsically.

## Is the frontier comparison fair?

**Matched:** task materials, the 60-minute wall clock, the single-GPU (one-seventh MIG H200) budget, the rubric judge, 8 rollouts per task, and the thinking effort of the Codex component (pinned to extra-high "to ensure a fair comparison").

**Not matched: the harness.** Claude Opus 4.8 and GPT-5.5 run bare, "using the built-in system prompts for Claude Code and Codex respectively," with the one-line user prompt *"Read prompt.md to understand your task, then complete it."* Faraday runs in a bespoke five-tool harness **with a full Codex GPT-5.5 CLI agent nested inside it as a tool**. So the Faraday-vs-Codex row is not model-vs-model; it is *Codex GPT-5.5 plus a trained 27B orchestrator* versus *Codex GPT-5.5 alone*. Faraday's system strictly contains its own baseline.

That is a real harness advantage, and the paper never denies it — CAT *is* the contribution, and the authors present the delegation layer as the thing being trained. But it means the 6–8% figure cannot be read as evidence about the 27B model's standalone capability, and it sits squarely in the swing band this wiki has already measured: [[harness-vs-model-attribution]] records **11–15 point scaffold-only swings** (GPT-5, Kimi K2 on [[swe-bench-verified|SWE-bench Verified]]) with the model held fixed, and ~13–15 points between a bad and a good scaffold. Also unreported: any token or dollar accounting for the extra inner-agent inference Faraday spends, so the comparison is matched on *wall clock and GPUs* but not on *inference compute*.

Two comparisons in the paper **are** clean, and they are the ones that carry the evidential weight:

1. **Faraday vs. Qwen3.6-27B** is strictly matched-harness — same simple harness, same Codex GPT-5.5 tool, differing *only* in RL post-training — and it is a large uplift. This isolates the training, not the scaffold.
2. **Prompt-optimised Codex vs. Codex** rules out the cheapest alternative explanation (that a better prompt would close the gap), though a prompt control is not a scaffold control: nobody ran Claude or Codex *as the outer agent* in Faraday's harness with a coding agent tool.

## Limitations

Stated by the authors: the rubric judge agrees with humans **better than the baseline judge but not significantly so**, and disagrees with humans on several tasks; the human agent-comparison study is conditional by design and cannot establish average human preference; the rubric judge was **never validated on the "innovation" variants** or at full experimental scale, so both generalisation claims are provisional; full-scale results rest on a single rollout per task on 8 tasks; the corpus is deliberately famous papers, so pre-training contamination is present and scaling the space would surface papers that genuinely do not replicate, requiring judges that can recognise non-replicability. Some corpus papers were written by the authors or their acquaintances (declared; same automated pipeline applied throughout). Faraday **failed to replicate several papers the authors believe were rigorously conducted**, and they explicitly decline to read those failures as evidence against the original work.

Reality-check from four original authors (Rupp et al. 2012, Gómez-Bombarelli et al. 2018, Reed et al. 2022, Lu et al. 2024) who reviewed Faraday's replication of their own figure: praise for correctness and inventiveness ("the reflexion implementation looks correct", "nice and clever toy task design", "more closely follows equation (1)"), alongside "the problem selected is probably too easy", "paragraph on pre-training dataset design is particularly bad", and "calculations contain unnecessarily convoluted code."

## Related

- [[accelerating-scientific-research-gemini-real-world-schmidgall-2026]] — the closest comparison and the mirror image. Co-Scientist puts the intelligence in an elaborate hand-built evolutionary harness (tournaments, LLM reward models, hallucination-clipping) and validates it in *physical* labs; Faraday puts the intelligence in the weights and deliberately keeps the harness minimal, in silico only. That paper cites this one as evidence AI scientists can be *trained* for replication.
- [[frontis-ma1-ai4ai-recursive-self-improvement-yang-2026]] — same lineage: post-train an "AI4AI" model to do ML engineering, evaluated on [[mle-bench]] Lite. Faraday's target is replication rather than leaderboard hill-climbing, and its reward is a rubric rather than a metric.
- [[coding-agents-replicate-sciml-papers-hans-2026]] — the same replication task attacked from the opposite end: harness-side evidence bundles and acceptance rules to stop the agent treating its own description of progress as evidence. Faraday moves that policing into a judge that re-runs the code.
- [[rehearse-confidence-cliff-autoresearch-ji-2026]] — the standing warning about LLM judgment degrading inside self-improvement loops; here the same LLM judgment *is* the RL reward across 659 steps, which makes multi-sample aggregation and judge self-consistency load-bearing rather than cosmetic.
- [[cedar-agent-orchestrated-tree-search-complex-systems-tian-2026]] — Sakana's LLM Editor + LLM Judge over runnable Python. Structurally the same variation/fitness split; Faraday replaces the hand-coded tree search with a trained policy, and cites the absence of a search harness as the point.
- [[naturebench-coding-agents-nature-family-papers-wang-2026]] / [[frontier-lms-struggle-to-copy-wen-2026]] — neighbouring evidence on how far frontier agents get on scientific reproduction unaided.
- [[harness-vs-model-attribution]] — the scaffold-swing evidence that bounds how much of the 6–8% gap could in principle be harness rather than weights.
- [[gamut-two-level-meta-rubrics-chen-2026]] — the meta-rubric-generates-rubric pattern used here to auto-write per-task graders.
- [[llm-as-a-judge]] / [[agent-as-a-judge]] — the judge is a *coding agent* with execution rights, which is the agent-as-a-judge variant rather than the text-only one.
- [[verifier]] / [[outcome-based-reward]] / [[reward-modeling]] — the paper's central bet is that a non-verifiable rubric reward is *safer* against [[reward-hacking]] than a verifiable one, because a hindsight judgement of a whole rollout is a moving target while a specified evaluation procedure is a fixed one. Asserted, not demonstrated.
- [[grpo]] / [[agentic-rl]] / [[reinforcement-learning]] — long-horizon, non-verifiable GRPO with turn-level credit; the ablation showing collapse without it is the concrete contribution.
- [[lora]] / [[fine-tuning]] — rank-128 adapters on all linear projections are the entire trained surface.
- [[self-improving-agent]] / [[darwin-godel-machine]] — the contrast the paper draws with self-modifying and evolutionary AI-Scientist systems; DGM's Figure 4 is itself one of the Replica tasks.
- [[voyager]] / [[voyager-wang-2023]] — another Replica task, and the clearest example of the hard-coding failure mode: the baseline pre-populates the skill library the figure exists to test.
- [[mle-bench]] / [[mlagentbench-huang-2023]] — the verifiable-benchmark tradition this paper argues has a ceiling, since its innovations are "adaptations but not exaptations."
- [[llm-agent]] / [[agentic-harness-engineering]] — CAT as an alternative to harness construction: "encoding capabilities in the weights of a neural network, rather than expressing them in code, is more flexible and generalisable in the long run."
- [[anthropic]] / [[openai]] / [[claude-opus-4-8]] / [[qwen]] / [[glm-5-2]] — models used as baselines, judge, rubric writer, and base policy respectively.

**Concept gaps noted, not created:** `ai-for-science` (used as a tag here and on the Schmidgall page but has no concept page), `paper-replication` / `replication-crisis`, `coding-agent-as-a-tool` (the CAT paradigm itself), `rubric-based-reward`, `turn-level-credit-assignment`, and entity pages for `inherent` (the lab), `damon-falck`, `edward-hughes`, `louis-kirsch`.

## References

- [arXiv:2608.13331](https://arxiv.org/abs/2608.13331) — Falck, Sabri, Surina, Foster, Sims, Devlin, Rogers, Collins, Aleksiev, Kirsch, Hughes. *Training AI Scientists to Replicate Research.* Inherent, 13 Aug 2026.
