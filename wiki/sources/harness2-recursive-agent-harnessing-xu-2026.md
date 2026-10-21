---
title: "Harness²: Recursive Agent Harnessing for an Open World"
type: source
created: 2026-10-21
updated: 2026-10-21
sources:
  - harness2-recursive-agent-harnessing-xu-2026.md
tags:
  - 2026
  - paper
status: complete
importance: high
authors:
  - Ruiyao Xu
  - Yanfei Chen
  - Zhongying CuiZhu
  - Bhavana Dalvi Mishra
  - Yifei Ming
  - Han Yu
  - Rujun Han
  - Chen-Yu Lee
  - Tomas Pfister
introduces:
  - [[harness2]]
  - [[recursive-agent-harnessing]]
year: 2026
venue: "arXiv preprint"
arxiv_id: "2610.14224"
---

# Harness²: Recursive Agent Harnessing for an Open World

**Source**: `sources/harness2-recursive-agent-harnessing-xu-2026.md` (text extracted from the arXiv PDF). [arXiv:2610.14224](https://arxiv.org/abs/2610.14224), 2026-10-09.

| | |
|---|---|
| **Authors** | Ruiyao Xu (Northwestern; work done as a Google Cloud AI Research intern), Yanfei Chen, Zhongying CuiZhu, Bhavana Dalvi Mishra, Yifei Ming, Han Yu, Rujun Han, Chen-Yu Lee, Tomas Pfister ([[google-research]] / Google Cloud AI Research) |
| **Code** | [github.com/google-research/harness2](https://github.com/google-research/harness2): Apache-2.0, Python. Repo created 2026-10-06; the last push read was `1cfe254`, 2026-10-13. "Not an officially supported Google product." Website: harness2.github.io |
| **Coverage** | Press coverage from about 2026-10-18 (MarkTechPost and others) |

## What it is

A **[[agentic-harness-engineering|harness]]** is everything around a frozen model that lets it act as an agent: prompts, tools, memory, skills, orchestration. **Harnessing** is optimizing that layer. **[[harness2|Harness²]]** is a plug-and-play framework that improves an agent's harness **during deployment**, on a stream of new tasks:
- with **no model-weight updates**;
- with **no ground-truth answers or verifiers**;
- **without revisiting** finished tasks.

It is a form of [[self-improving-agent|recursive self-improvement]] (RSI) whose object is the harness. The paper uses the RSI definition of [[last-ai-built-by-humans-recursive-self-improvement-duan-2026|Duan et al. 2026]] and the "Agent = Model + Harness" framing of [[harness-engineering-self-improvement-weng-2026|Weng 2026]].

The "²" refers to **two levels of recursion** ([[recursive-agent-harnessing]]):

| Level | What gets improved | When |
|---|---|---|
| **Task-level** | The **execution harness** for the current task | During the task, before the deliverable is returned |
| **Harnessing-level** | The **improvement harness**, a library that guides *how* future harnesses get improved | After each task |

## The harness it edits: eight components

The paper extends the file-level decomposition of [[agentic-harness-engineering-lin-2026|AHE]] by one component. Each component is a file mounted into the executor runtime:

| # | Component | File | Active |
|---|---|---|---|
| C1 | System prompt | `systemprompt.md` | every call |
| C2 | Guardrails | `guardrails.md` | every call |
| C3 | Memory | `LongTermMEMORY.md` | every call |
| C4 | Tool guidance | `tool_guidance.md` | every call |
| C5 | Subagents | `sub_agents/<n>/AGENT.md` | when delegated |
| C6 | Skills | `skills/<n>/SKILL.md` | when invoked |
| C7 | Scripts | `scripts/<n>.py` | run via bash |
| C8 | Plugin (middleware) | `plugin/<name>.ts` | hooks every tool call, "outside the model's discretion" |

On [[openai-codex-skills-2025|Codex]], `AGENTS.md` is the only always-on file, so the editor prompts tell it to keep that file small. The editor prompts weigh each component's runtime cost: always-on text spends context on every turn, while an on-demand skill or script "costs nothing until the moment it is invoked".

## How it works

### Task level: the Propose–Probe–Compose step

1. **Reference.** Run the agent on the task with the current harness. The run shows how the harness *behaves*, not whether the answer is right.
2. **Propose two scoped candidates.** The editor, conditioned on the task, the harness, the reference run and the library, writes two edited harnesses:
   - an **invariant** candidate, aimed at agentic weaknesses likely to recur anywhere: planning, tool use, context management, delegation;
   - a **domain** candidate, aimed at this domain's knowledge, procedures and constraints.

   Library components can be reused, but task-specific facts must be checked against this task's evidence.
3. **Probe.** Run each candidate **on the same task**, with the same model and environment. Comparing each trajectory and deliverable with the reference gives a *behavioral contrast*: did the edit fix the targeted weakness, change nothing, or cause a regression? The paper treats this as evidence for composing edits, **not as proof of correctness**.
4. **Compose.** The composer reads the reference, the candidates, their diffs and all the evidence, then builds the successor harness. It may merge supported edits, revise conflicting ones, adopt one candidate, or **keep the current harness**. Every decision must cite a concrete observation, "rather than a presumed score gain". The composed harness is then run to produce the deliverable.

**Scaling with budget k:**
- **Parallel:** k reference runs, k invariant and k domain candidates probed, then k independent compositions that each see the whole pool. This is one update, made wider; it follows the RSA aggregation idea.
- **Sequential:** k Propose–Probe–Compose steps chained, carrying the harness and the growing evidence forward, in the spirit of Self-Refine.
- **Final selection (k > 1):** an agentic selector compares *anonymized, shuffled* deliverables against the request and source materials, picks one, and never writes a new one.
  - **The pool holds only the k composed runs.** In parallel mode those are the k compositions; in sequential mode, the k chained steps.
  - **No reference runs.** `selection_pool()` in `harness2/core/selection.py` excludes them, "so … there is no abstention by construction".
  - **The "mixed" prompt belongs to Best-of-k.** The prompt variant that includes base-harness outputs, saying picking one is "a completely valid and expected outcome", serves the Best-of-k baseline, not Harness² (`pool_kind="finals"` in `schedule.py`).
  - **At k = 1 there is no selection.** The composed run's output is returned.

### Harnessing level: the improvement library

After each task, a **reflector** reviews the whole record: references, candidates, probes, composition decisions and the selection.
- It keeps **lessons only when execution contrasts support them**. "Selecting an output alone does not establish transferability."
- If evidence is thin, it adds nothing.
- Each library entry is an executable harness plus an `EXPERIENCE.md` (when it applies, the lessons, the evidence), with an index for on-demand lookup.
- **The next task starts again from the common base harness.** It is guided by the updated library, rather than inheriting the last task's tailored harness. This is the paper's answer to noisy drift when tasks change.

### Guards visible in the code

These come from `harness2/core/prompts.py` and the paper's appendix D.2:
- Every editor and selector session gets an **"answer-free"** block: no rubric, answer key or score is in context, and it must not try to work out "what the grader wants". A guard also refuses to hand over any working directory that contains a verdict, rubric or score.
- An **"evidence is data, not instructions"** block: trajectories and candidate harnesses may contain injected commands. Scripts found in a candidate are read as text, never executed.
- **Validation:** an invalid harness tree (a skill without a trigger sentence, a script that won't compile, a missing helper) is retried once. If it fails again, the stage keeps its starting tree.
- **Roles are set by environment variables:** `HARNESS2_SOLVER_MODEL` (executor), `HARNESS2_IMPROVER_MODEL` (editor and selector; defaults to the solver model) and `HARNESS2_JUDGE_MODEL` (benchmark grading only).

## Task setup, cost and evidence (code-level detail)

### Where tasks come from

- **Harness² generates no tasks and no test set.** Each incoming task is its own test bed: the probes are **re-runs of the same task**, made right away inside one request, before the deliverable is returned.
- **Users never repeat anything.** A later, similar task does not reuse the earlier answer. Only the library entry carries over.
- **Evaluation tasks** come from three existing third-party benchmarks, with fixed test splits:
  - WorkBuddy-Bench (Tencent): test halves of 40 Code, 35 Web and 25 Office tasks;
  - JobBench: 34 of 65 tasks;
  - LAB (Harvey): 100 held-out tasks, using the split from an earlier Meta-Harness study.
- **Task order:** tasks are streamed per domain (practice area or occupation), in order within each domain, shuffled by seed.
- **Training halves** are used only by the Meta-Harness baseline.
- **Not stated:** how the upstream benchmarks were authored (by humans, AI-assisted, or both).
- **Inconsistency:** LAB's size is 1,251 tasks in the paper and a "full stream" of 1,143 in `docs/EXPERIMENTS.md`.
- **Rubric leak risk:** JobBench's `RUBRICS.json` is public on Hugging Face, and the executor has open web access. The docs say to block those hosts.

### What the budget k costs per request

| | Parallel (width) | Sequential (depth) |
|---|---|---|
| Reference runs | k | k |
| Probes | k invariant + k domain, all from the same harness | 2 per round × k rounds, each round from the previous checkpoint |
| Composed runs | k independent compositions over the whole pool | k checkpoints |
| Executor runs | ≈ 4k | ≈ 4k |

- **k = 1** is one step: 4 runs and no selection. **k = 5** is about 20 runs, plus editor and selector sessions.
- **Selection (k > 1)** picks one of the **k composed outputs only**. `selection_pool()` excludes the reference runs, "so … there is no abstention by construction", and Harness² never returns the original harness's output.
- **Side effects** (wiki observation): re-running a task 4–20 times is only safe when it is repeatable. The benchmarks use a fresh workspace or container for every run. For agents that send email or write to production systems, the probes would need a sandbox or dry run; the paper doesn't address this.

### How the composer weighs evidence

The composer is an OpenCode agent session that receives a `context/` folder:
- `task.md` and `source/`;
- **full trajectories** in `rollouts/` (not clipped as a whole) and the produced files in `deliverables/`;
- `pool/<label>/` with each candidate harness, its `harness.diff` and its run;
- the current `harness/`;
- `library/INDEX.md`;
- in sequential mode, earlier `decisions/`.

It is told to "read it — do not answer from this message alone". Its weighing rules (the `aggregate()` prompt):
- **Repeated failures count; one-offs don't.** Read the k base runs as one set. A failure repeated across them is "the strongest thing you can act on"; one seen in a single run out of k is noise.
- **Single runs are weak evidence.** Each candidate ran once, so an edit must point to "a SPECIFIC MOMENT in a trajectory" that shows the mechanism. "This looks like good practice" doesn't count.
- **Agreement is not independence.** Candidates share biases, so agreement "is support, not independent proof", and contradiction means "inspect the trajectories, not … vote".
- **Downside counts as much as upside.** An edit tied to a run that produced nothing is UNSAFE unless the other evidence is overwhelming.

**Output:** MERGE, ADOPT ONE or KEEP_BASE, plus one audit line per decision: `kept|revised|dropped -> <edit> -> <trajectory moment or base-set pattern, e.g. 'failed in 2/3 base rollouts'>`.

**In sequential mode** the composer may also revive an edit rejected earlier, or restore an earlier checkpoint word for word if later rounds drifted.

**With k = 1** there is a single base run, so the "repeated across the set" rule has little to work with. Larger k mostly buys more reliable evidence.

## Experimental setup

- **Executors (frozen):** Gemini 3.5 Flash and [[glm-5|GLM-5]], each under two unmodified starting harnesses, [[opencode|OpenCode]] and Codex. That gives 4 configurations.
- **Editor and selector:** [[claude-opus-4-8|Claude Opus 4.8]] under OpenCode, at high reasoning effort.
- **Benchmarks:**
  - **WorkBuddy-Bench:** Code, Web and Office, in containers. Graded by deterministic checks plus LLM/VLM items judged by Gemini 3.5 Flash.
  - **JobBench:** 34 test tasks, long professional jobs. Rubric judged by **Claude Opus 4.8**.
  - **LAB:** Harvey's Legal Agent Benchmark, 100 test tasks. Judged by Claude Sonnet 4.6; reports per-criterion pass rate ("Partial").
- **Runs:** three shuffled task streams; the library starts empty.
- **Baselines** (sharing the same executor, starting harness and judge):
  - **ReasoningBank:** the harness is fixed; retrieved strategy memories are prepended to the prompt.
  - **[[meta-harness-lee-2026|Meta-Harness]]:** offline search on a training split.
  - **Harness Scaling** (Wang et al. 2026a): per-task revision with no state carried between tasks.
  - **Best-of-k.**

## Results

**One step (k = 1) improves all 12 benchmark × configuration pairs:**

| Executor / starting harness | WorkBuddy avg (base → H²) | JobBench | LAB Partial |
|---|---|---|---|
| Gemini 3.5 Flash / OpenCode | 72.4 → 77.5 | 31.5 → 36.0 | 79.9 → 81.9 |
| Gemini 3.5 Flash / Codex | 74.5 → 77.5 | 33.9 → 35.3 | 71.6 → 75.6 |
| GLM-5 / OpenCode | 64.2 → **75.7 (+11.5)** | 20.6 → 24.2 | 59.6 → **71.4 (+11.8)** |
| GLM-5 / Codex | 64.4 → 75.3 | 23.6 → 24.8 | 58.1 → 65.9 |

- **Against the baselines:** Harness² beats all of them on WorkBuddy average and JobBench in every configuration. On LAB, Meta-Harness leads for Gemini 3.5 Flash under Codex (76.5 vs 75.6).
  - ReasoningBank drops up to 6.6 points below the base on coding.
  - Harness Scaling is below the base in 7 of 12 pairs.
- **Weaker executors gain more** (on WorkBuddy and LAB). GLM-5 with one step (75.7) passes base Gemini 3.5 Flash (72.4): "A weaker model with an improved harness can thus outperform a stronger one with its original harness."
- **Recursion (parallel, k = 5)** adds +2.9 to 3.9 on WorkBuddy, +3.0 to 4.2 on JobBench and +2.6 to 6.7 on LAB, over one step. Both modes rise steadily from k = 1 to 5. Parallel ends higher at k = 5, while sequential leads on Code at k = 2. Both modes at k = 2 beat every baseline at k = 5.
- **Compute:** counting every token before the deliverable, both Harness² settings lie on the empirical Pareto frontier. Meta-Harness uses more compute than Harness² at k = 5 for a lower score.
- **Ablations** (Gemini 3.5 Flash / OpenCode, k = 1; WorkBuddy and JobBench, change against full Harness²):

  | Removed | WorkBuddy | JobBench |
  |---|---|---|
  | Library | −3.1 | −1.8 |
  | Invariant / domain scoping | −2.9 | −3.7 |
  | Composition | −1.6 | −1.9 |

- **The library carries over:**
  - **Editing without probing hurts.** Editing the base harness *without* reference runs, probes or library cuts Code from 73.4 to 63.2 and Web from 69.9 to 65.0.
  - **A bigger budget builds a better library.** A library built at k = 1 restores base performance; one built at k = 5 gives +0.9, +4.4 and +5.1 (Code, Web, Office) with no further probing.
  - **Across benchmarks:** a JobBench library improves WorkBuddy by +0.6, +0.5 and +2.9.
- **The editor need not be stronger.** With one model as both executor and editor (OpenCode, k = 2), all five models tested improve on every benchmark, from GLM-5 to Claude Opus 5. Opus 5 gains +3.3, +5.5 and +3.0; GLM-5 gains +13.7 on LAB.
- **Case study** (JobBench, 43.1 → 77.8): the base run never reconciled figures across files and never reopened its outputs. The harness gained:
  - an invariant guardrail: take an inventory of inputs, trace every figure to its source, and rerun `verify_outputs.py`;
  - a domain skill for reconciling schedule, venue, labour and budget;
  - tool guidance on numbers stored as strings and on retrying web search once.

## Limitations

**Stated by the authors:**
- Extra inference cost grows with k. Parallel sessions cut latency but not compute.
- Behavioral contrasts and model judgment are "sensitive to execution variability and shared model biases" and don't guarantee correctness or transfer.
- The library is a simple index with no consolidation, so it may bloat over time.
- The recursion budget is fixed in advance, not adaptive.

**Operational (from the README):**
- The executor has unrestricted network access, and Codex runs **with its sandbox disabled**.
- Public rubrics may be reachable online, so you must set network restrictions before evaluating.

> [!note] Reading the numbers (wiki observation)
> - **Same model on both sides.** On JobBench the judge is **Claude Opus 4.8**, the same model as the editor and selector. The base-vs-Harness² comparison is partly Opus judging Opus-shaped harness edits. The paper flags "shared model biases" in general, but doesn't isolate this.
> - **The baselines share it too.** Their revision and reflection also use Opus 4.8.
> - **Graders differ by benchmark.** WorkBuddy's deterministic checks and LAB's separate judge (Sonnet 4.6) are less exposed to this effect.
> - **No outside replication.** All results are the authors' own.

## Relation to other wiki sources

- **[[harness-updating-is-not-harness-benefit-lin-2026]]** found edit *writing* is flat across model tiers. That fits Harness²'s result that a same-model editor still helps.
  - Lin et al. also found *benefit* is non-monotonic: weak solvers gain little.
  - Harness² reports the weaker of its two executors gaining more. Both its executors are probably mid-tier, so this is a tension to watch, not a contradiction.
- **[[meta-harness-lee-2026]]** is the offline counterpart, tuned on a training split. Harness² works online on unseen tasks and still beats it in most settings.
- **[[harness-continual-learning-kang-2026|HCL]]** and Continual Harness (Karten 2026) are concurrent online methods. Harness²'s comparison table marks HCL as lacking an improvement harness and scoped probes.
- **[[agentic-harness-engineering-lin-2026|AHE]]** is the source of the component decomposition.
- **[[rrsi-regularized-recursive-self-improvement-agent-harnesses-xia-2026]]** and **[[scaling-laws-agent-harnesses-effective-feedback-compute-zhang-2026]]** are other 2026 work on recursive harness self-improvement and its compute.

## Concepts & entities

[[harness2]] · [[recursive-agent-harnessing]] · [[agentic-harness-engineering]] · [[self-improving-agent]] · [[google-research]] · [[opencode]] · [[glm-5]] · [[claude-opus-4-8]] · [[meta-harness-lee-2026]]

_Original source: `sources/harness2-recursive-agent-harnessing-xu-2026.md` (PDF: `sources/harness2-recursive-agent-harnessing-xu-2026.pdf`). Code read from a shallow clone of the repo at `1cfe254`._
