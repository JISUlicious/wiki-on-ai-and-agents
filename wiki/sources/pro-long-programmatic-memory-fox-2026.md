---
title: "PRO-LONG: Programmatic Memory Enables Long-Horizon Reasoning"
type: source
created: 2026-08-04
updated: 2026-08-04
sources:
  - pro-long-programmatic-memory-fox-2026.md
arxiv_id: "2607.20064"
year: 2026
authors:
  - Alexis Fox
  - Junlin Wang
  - Paul Rosu
  - Bhuwan Dhingra
tags:
  - 2026
  - paper
status: complete
importance: high
---

# PRO-LONG: Programmatic Memory Enables Long-Horizon Reasoning

**Alexis Fox, Junlin Wang, Paul Rosu, Bhuwan Dhingra** (Duke University) — [arXiv:2607.20064](https://arxiv.org/abs/2607.20064), v2, 23 Jul 2026 (cs.AI). No venue in the arXiv comments. Code and logs: `github.com/alexisfox7/PRO-LONG`.

## Summary

PRO-LONG frames agent memory as a `write`/`read` pair and argues both should be as dumb as possible. Existing systems — scratchpads, summarized observations, embedded memory entries, distilled skills (MemGPT, Mem0, ReasoningBank, SkillRL) — put intelligence in the `write`: heuristics deciding what to persist, what to compress, what to discard. That creates a **fidelity–tractability tradeoff**: reducing what you save makes retrieval easier but throws away detail whose relevance is only evident in hindsight, and over a long horizon a lossy summary can matter many turns later.

**Programmatic memory** inverts this. The `write` is the harness appending *every* observation, action, and outcome into a structured log — no learned or heuristic decisions, nothing compressed or summarized, so the log is a faithful ground-truth record of environment state. The `read` is **programmatic search over that log, primarily through code**: grep, regex, and Python, which are native to coding agents and stay tractable over logs of 100k+ lines. The design is deliberately parasitic on coding-agent capability rather than orthogonal to it: this is the same move as recursive language models querying context slices through a REPL, applied to an agent's own trajectory. The distinction the paper leans on is **accessed** context (what's in the active window at a model call, ~100k–1M tokens) versus **accessible** context (tool-reachable state, 10M+ tokens).

The implementation is almost nothing: each action appends a structured entry to `logs.txt` — a header (action number, level, attempt, score), the agent's summarized plan, the selected action, and the resulting board — and the prompt is **around 30 lines with no subagents**. Compare the baselines it matches: WorldModeler has a ~600-line prompt instructing the agent to write and continually update a Python simulator, plus helper scripts and two subagents; Schema replaces Claude Code's default tools with twelve custom MCP tools, maintains a `step(grid, action)` function, and searches plans by BFS; Arcgentica runs a main orchestrator over four subagents sharing a hypothesis database. On the 25-game ARC-AGI-3 public set, PRO-LONG **improves over the same coding agents without the log by 18.0 percentage points on average** and reaches or beats those specialized harnesses at **4.2–5.8× fewer billed tokens**. Notably, world-modeling behavior emerges anyway: on `m0r0` the PRO-LONG agent writes its own transition function and runs breadth-first search over joint block positions, without being told to.

## Key points

**Setup.** ARC-AGI-3, 25 public games, six to ten levels each, rules undisclosed so dynamics must be inferred by exploration. Scoring is the benchmark-standard **RHAE**: `S_{l,e} = min((h_{l,e}/a_{l,e})², 1.15)` against an upper-median human action count, so beating the human baseline earns up to 115% per level; later levels weighted more heavily. Default config: high reasoning effort, **500 actions per game**, ≤20 actions per turn, boards as 64×64 ASCII text grids (no vision, except WorldModeler). GPT models run under Codex, Claude models under [[claude-code]], standard tool space in a sandbox with `python3` and only the Python standard library. Prior agents' released runs were **rescored** under the same 500-action budget and scoring version.

**The `no-log` baseline is deliberately strong** — a full coding agent with ~250k accessed context, file system, and shell, free to write its own notes and saved boards. That is far above the standard for the benchmark, where ARC-AGI-3 reference agents see the last 3–5 boards through a 10-message window and BALROG agents see the last 16 observations.

**Main results (Figure 2).**

| Model · harness | PRO-LONG | no-log baseline | Best prior harness | Cost |
|---|---|---|---|---|
| GPT-5.5 (Codex) | **41.2 pass@1**, 60.1 best@5 | 24.0 (+17.2 pp) | WorldModeler 45.1 pass@1 | **5.8× fewer tokens** at pass@1; 1.2× cheaper at best@5 |
| Opus 4.6 | **42.4 pass@1** (best public result for that model) | 21.4 (+21.0 pp) | Arcgentica 39.0 pass@1 | +3 pp *and* cheaper |
| Fable 5 (Claude Code), 500 actions | **76.1 pass@1**, 82.1 best@2 | 60.4 (+15.7 pp) | Schema 84.4 best@2 | **4.2× fewer tokens** (~400M tokens saved across the game set) |
| Fable 5, 2,000 actions | **94.6 pass@1**, **97.4 best@2** | — | Schema 99.0 best@2 at $6,447 | $1,500 / **$1,750** — more than 3× cheaper |

- Across GPT-5.5, Opus 4.6, and Fable 5, PRO-LONG beats the no-log baseline by **15.7–21.0 pp** (18.0 pp average). Under matched scoring it surpasses specialized harnesses on Opus 4.6 (+3 pp) and lands within 2–4 pp on GPT-5.5 and Fable 5 — while being model-agnostic in implementation, where prior harnesses are tuned per model.
- The 97.4 best@2 figure is a **lower bound**: several games were run only once at the 2,000-action budget.
- **Variance across independent runs is large**, so best@k matters: GPT-5.5 goes 41.2 pass@1 → **60.1 best@5 (+18.9 pp)**. Cost detail: a few games dominate — `bp35` alone costs $298.

**Memory fullness drives it (Figure 3, GPT-5.5 best@k).** Three conditions: full log, log truncated to the last 25 actions, and no log. Ordering is full > last-25 > none at every k (best@5: **60.1 / 51.9 / 34.2**), and the gap *widens* with k — fuller memory expands the set of games solvable at all, rather than just making already-solvable games more reliable.

**Tool-ladder ablation (Table 1, GPT-5.5 pass@1).** Confirms the read side is where the value is:

| Tools enabled | Score |
|---|---|
| Read only | 23.1% |
| + Grep / regex search | 27.2% (+4.1) |
| + Python | 38.3% (**+11.1**) |
| + Write, Edit | 41.2% (+2.9) |

Programmatic access accounts for nearly all of the gain; adding the write/edit tools — the agent's primary means of maintaining *other* forms of memory like notes — adds only 2.9 points.

**Workspace-persistence ablation (Table 2).** Wiping the workspace after every model call costs `no-log` about four points (24.0 → 19.9) but costs PRO-LONG **0.5 points** (41.2 → 40.7). The log, not self-authored notes, is doing the work.

**Tool-call distribution (Table 3).** PRO-LONG spends **60.6%** of calls on programmatic analysis (Python) vs. 40.8% for no-log, with 20.3% on log parsing (9.6% regex search across turns, 10.7% tail/head/cat for recent state) — a category no-log uses **0.0%** of the time. Conversely no-log spends 59.2% on workspace management vs. 19.1%, and workspace *writing* falls from 30.3% (no-log) to 4.6% (PRO-LONG).

**Where it matters (Figure 4).** Games whose current board largely specifies the dynamics (`ft09`, `cd82`) show little gap — full history adds nothing when the present state is sufficient. The gap is largest where long-horizon reasoning is required: `g50t`, whose "rewinding" mechanic spawns ghosts replaying earlier paths, produced **run logs exceeding 320,000 lines** and PRO-LONG still scored up to 56.3% with GPT-5.5; on `m0r0` PRO-LONG scored **100% on all five runs vs. 34.2% for no-log**. One agent wrote a `regress.py` that replays every action in the log to verify its coded game model against the recorded boards.

## Concepts & entities

- [[memory-management]] — the central contribution: `write` = append-all, `read` = code search, deliberately putting zero intelligence in the write path to avoid the fidelity–tractability tradeoff.
- [[prompt-compression]] — the explicit foil. PRO-LONG's argument is that summarization discards detail whose relevance surfaces later; the ablations (last-25 truncation, workspace wiping) are direct measurements of that cost.
- [[long-context-llm]] — motivated by context rot: nominal million-token windows degrade in practice, hence the accessed vs. accessible split with 10M+ tokens kept accessible.
- [[code-as-harness]] — the strongest instance in this batch: the harness contribution is a log file plus a ~30-line prompt, and all retrieval intelligence is the coding agent executing code against its own trajectory.
- [[claude-code]] — one of the two base coding agents (Claude models); Schema, the strongest prior Claude harness, replaces its default tools with twelve custom MCP tools.
- [[agent-evaluation]] — a careful protocol worth noting: RHAE scoring, rescoring prior agents' released runs under a matched 500-action budget, bootstrap CIs, and reporting best@k because run-to-run variance is large.
- [[agent-three-layer-model]] — sharpens the model/scaffold boundary: nearly all of PRO-LONG's gain comes from the tool layer (Python, grep) acting on a passive artifact, not from orchestration logic.
- [[dynamic-workflows]] — a negative result for elaborate scaffolding: subagent orchestration, custom tool suites, and 600-line prompts are matched by an append-only log plus a coding agent.
- [[recontext-recursive-evidence-replay-zhao-2026]] — same family of idea (keep the evidence, re-reach it on demand) applied to a static long context rather than an agent's own trajectory.

## References

- [arXiv:2607.20064](https://arxiv.org/abs/2607.20064)
</content>
