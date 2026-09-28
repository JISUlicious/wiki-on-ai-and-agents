---
title: "ClawGym II: Exploring Black-Box RL on Agent Harness"
type: source
created: 2026-09-05
updated: 2026-09-05
sources: []
arxiv_id: "2608.16798"
year: 2026
authors:
  - Huatong Song
  - Fei Bai
  - Ming Yang
  - Renyuan Li
  - Jia Deng
  - Jujie He
  - Zhange Zhang
  - Daixuan Cheng
  - Yan Xing
  - Qi Yun
  - Xuxing Chen
  - Danyang Li
  - Feng Chang
  - Chuan Hao
  - Ran Tao
  - Jian Yang
  - Bryan Dai
  - Wayne Xin Zhao
  - Mingjie Tang
  - Ji-Rong Wen
tags:
  - 2026
  - paper
status: complete
importance: high
---

# ClawGym II: Exploring Black-Box RL on Agent Harness

**Huatong Song, Fei Bai, Ming Yang, Renyuan Li, Jia Deng, Jujie He, Zhange Zhang, Daixuan Cheng, Yan Xing, Qi Yun, Xuxing Chen, Danyang Li, Feng Chang, Chuan Hao, Ran Tao, Jian Yang, Bryan Dai, Wayne Xin Zhao, Mingjie Tang, Ji-Rong Wen** (IQuest Research; Gaoling School of Artificial Intelligence, Renmin University of China — corresponding: Wayne Xin Zhao et al.) — [arXiv:2608.16798](https://arxiv.org/abs/2608.16798), cs.CL, 17 Aug 2026, arXiv perpetual non-exclusive license. Preprint; no venue. Sequel to ClawGym (Bai et al., arXiv:2604.26904), whose SFT models (ClawGym-8B / -30A3B), synthetic training data (ClawGym-SynData), and benchmark (ClawGym-Bench) are reused here.

## Summary

The research question is stated in one line: *"How can black-box RL be performed through a harness to stably optimize general agents?"* The harness — [[openclaw|OpenClaw]] or [[claude-code|Claude Code]], literally the production binaries — is treated as an **unmodified, opaque rollout engine**. A serving proxy sits at the model boundary as the harness's model endpoint, serves the current policy, and records exact input tokens, generated tokens, rollout log-probs, and task metadata for every call. Each rollout runs inside a temporary sandbox provisioned on demand and destroyed on completion; extra capabilities (web search, task-specific tools) are exposed through [[mcp|MCP]] servers launched *inside* the sandbox so the native harness workflow is untouched. A verifier scores the final workspace state and produces one rollout-level reward.

The technical center is what happens *after* capture. Captured calls are "fragmented, forked, and often redundant" — mature harnesses fork on context compaction and subagent delegation, and retry on malformed tool calls — so ClawGym II organizes each rollout's calls into a **rollout-level prefix tree** (citing Kat-Coder-V2 for the construction), filters its leaves, and adapts both [[ppo|PPO]] and [[grpo|GRPO]] to optimize over the recovered tree with shared-prefix tokens counted once per rollout. Training–inference consistency is kept by a **token-in-token-out** discipline plus **token-level importance-sampling correction**. On top of that, because every harness meets the same boundary and the same trajectory representation, the paper introduces **mix-harness training**: one policy jointly optimized by rollouts from OpenClaw *and* Claude Code, with GRPO groups defined per task–harness pair.

Headline, all verified against Table 1: with Qwen3-30A3B as backbone, black-box RL raises **ClawGym-Bench Pass@1 by 9.98 points through OpenClaw and 14.81 through Claude Code**, and **PinchBench by 11.71 and 17.28**, while staying stable over **200–400 optimization steps**. The 30A3B models beat the 8× larger Qwen3-235A23B under both harnesses. The framework also transfers to JobBench-style (20.46 → 27.20) and OfficeQA-style (8.53 → 21.54) tasks with Claude [[code-as-harness|Code as harness]].

> [!note] Read the 9.98 carefully
> The OpenClaw gain is measured against **ClawII-Cold-30A3B (52.64)**, a lightly SFT-warmed initialization, not the base model. Against base Qwen3-30A3B under OpenClaw (45.11) the total lift is **+17.51**. The Claude Code arm has no cold start, so its 14.81 *is* against base (37.06 → 51.87). The two headline numbers are therefore not measured from the same kind of starting point.

## Method

**Prefix tree construction (§3.2.1).** For calls `{(xᵢ, yᵢ)}`, build a tree rooted at the initial task prompt; attach each call to the node whose accumulated history is the longest prefix of `xᵢ`. Comparing a child's input against the parent's history plus recorded response recovers the harness-inserted content between calls (tool outputs, environment feedback). Each root-to-leaf path is a candidate multi-turn trajectory; compaction and subagents create additional leaves. Shared prefixes are stored once.

**Filtering (§3.2.2)** — three rules:

| Rule | Trigger | Action |
|---|---|---|
| Remove dead leaves | Inference-server retries or harness regeneration after a malformed tool call (OpenClaw does this) leave superseded attempts as leaves with no continuation | Within each compaction-delimited segment, keep the leaf with the longest valid continuation; drop shorter retry siblings |
| Discard over-branching rollouts | Leaf count exceeds a preset threshold — signature of repeated/failed generation | Drop the whole rollout from the update |
| Exclude auxiliary trajectories | Subagent and compaction trajectories | **Not trained on at all** — broadcasting the terminal reward to them "would introduce ambiguous credit assignment"; only main-agent trajectories are optimized |

**Optimization over the tree (§3.2.3).** All retained trajectories of a rollout inherit its terminal reward `Rᵢ`. **GRPO:** group = the *n* rollouts of task *q*; advantage `Âᵢ = (Rᵢ − μ_q)/(σ_q + ε)` computed **once per rollout** and assigned to every trainable token on the retained trajectories, with shared-prefix tokens contributing once. **PPO:** a simplified variant — trajectories within a rollout treated independently, γ = λ = 1, so GAE collapses to `Â_t = Rᵢ − V_φ(s_t)`; no advantage propagates across branch points. The authors flag that this forces the critic to estimate long-horizon returns from intermediate states and may raise variance; "a more principled treatment of forked trajectories is left for future work."

**Training–inference consistency (§3.2.4).** Two decoupled views per call: the inference engine's generated tokens are *grafted directly onto the tree* and are the sole training data; the text the harness receives is decoded from those tokens and "never encoded back into the trained trajectory." Tool responses are encoded exactly as presented in the next request and appended. This defends against harness-side transformations (tool-call normalization, assistant-message re-serialization). A second mismatch remains — `π_rollout(a|s) ≠ π_old(a|s)` between inference and training engines under different precision/kernels/parallelism — corrected per token by `w_t = min(exp(log π_old − log π_rollout), c̄)`, choosing token-level (biased, low-variance) over sequence-level (unbiased, high-variance) ratios.

**Mix-harness training (§3.3).** Each `(task, environment, harness_k)` is a separate training instance; instances from different harnesses are randomly mixed in a batch, but **advantage normalization happens within task–harness groups**, never across harnesses, so harness-dependent reward distributions do not distort baselines.

**Safeguards (§3.4).** Wall-clock timeouts; sandbox HTTP/initialization failures excluded from both advantage estimation and the update. **Buffered pseudo-streaming**: output tokens are buffered while synthetic SSE keep-alives hold the streaming connection open, then parsed once non-streaming — because harnesses' incremental tool-call parsers occasionally truncate a *correct* call mid-stream. **Settling**: wait until the record count is unchanged across several checks (with a timeout) before assembling trajectories, since harnesses retry asynchronously in the background.

## Results

**Setup.** ClawGym-Bench scored by code verifier alone, or 0.7·code + 0.3·rubric where rubrics exist; PinchBench = 30 non-multimodal tasks from the 10 Apr 2026 release, native verifiers. Rubric judge: GPT-5.4 (prompt in Appendix A). Training data: ClawGym-SynData. 64K context; longer trajectories left to the harness's own context management. ClawII-OC initialized from ClawII-Cold (lightweight SFT on ClawGym-SynData, "due to data availability"); ClawII-CC trained directly from base.

**Table 1 — Pass@1** (six ClawGym-Bench categories: Productivity & Collaboration, Systems & Automation, Analysis & Reasoning, Content & Domain, Planning & Knowledge, Software Development):

| Harness | Model | PinchBench | Prod. | Sys. | Anal. | Cont. | Plan. | SW Dev | **Avg** |
|---|---|---|---|---|---|---|---|---|---|
| OpenClaw | Qwen3-8B | 54.50 | 37.46 | 29.06 | 30.40 | 41.12 | 44.47 | 30.69 | 35.02 |
| OpenClaw | Qwen3-32B | 49.40 | 40.68 | 36.21 | 37.84 | 47.16 | 49.29 | 33.11 | 40.32 |
| OpenClaw | Qwen3-30A3B | 55.60 | 42.47 | 42.09 | 45.04 | 51.95 | 45.98 | 46.24 | 45.11 |
| OpenClaw | Qwen3-235A23B | 60.60 | 53.66 | 52.27 | 47.18 | 61.39 | 67.33 | 49.23 | 54.48 |
| OpenClaw | ClawGym-8B (SFT) | 75.70 | 49.47 | 46.83 | 46.35 | 55.37 | 52.29 | 54.78 | 50.24 |
| OpenClaw | ClawGym-30A3B (SFT) | 86.00 | 52.98 | 50.97 | 64.64 | 61.46 | 57.90 | 56.13 | 56.82 |
| OpenClaw | ClawII-Cold-8B | 71.29 | 48.80 | 42.49 | 45.06 | 51.39 | 54.74 | 42.47 | 47.06 |
| OpenClaw | ClawII-Cold-30A3B | 75.61 | 54.34 | 47.61 | 52.58 | 53.42 | 60.64 | 48.88 | 52.64 |
| OpenClaw | ClawII-OC-8B | 77.44 | 53.17 | 55.99 | 51.28 | 59.60 | 61.68 | 50.83 | 54.98 |
| OpenClaw | **ClawII-OC-30A3B** | **87.32** | 62.75 | 59.81 | 64.09 | 62.18 | 63.12 | 66.71 | **62.62** |
| Claude Code | Qwen3-8B | 27.40 | 16.19 | 13.94 | 20.26 | 25.37 | 23.62 | 14.65 | 18.54 |
| Claude Code | Qwen3-32B | 53.27 | 33.34 | 25.63 | 27.68 | 33.03 | 27.85 | 28.30 | 29.37 |
| Claude Code | Qwen3-30A3B | 54.14 | 38.75 | 33.06 | 32.99 | 45.43 | 41.62 | 32.26 | 37.06 |
| Claude Code | Qwen3-235A23B | 62.47 | 41.74 | 40.53 | 43.70 | 50.35 | 49.03 | 54.90 | 45.59 |
| Claude Code | ClawII-CC-8B | 61.21 | 41.27 | 38.19 | 41.65 | 49.33 | 44.61 | 39.37 | 42.05 |
| Claude Code | **ClawII-CC-30A3B** | **71.42** | 47.84 | 45.59 | 48.88 | 57.05 | 56.93 | 62.93 | **51.87** |

Derived deltas (all reproduce the paper's text): 8B +7.92 (OC, vs Cold) / +23.51 (CC, vs base); 30A3B +9.98 / +14.81; ClawII-OC-30A3B beats the ClawGym-30A3B SFT baseline by **+5.80** and Qwen3-235A23B by **+8.14** (OC) / **+6.28** (CC). Note that *every* Qwen3 base scores far lower under Claude Code than under OpenClaw (30A3B: 45.11 vs 37.06; 8B: 35.02 vs 18.54) — the RL gain under Claude Code is partly recovering a harness the base model handles badly.

**Training dynamics (§4.2).** Qwen3-30A3B, both harnesses, PPO and GRPO. Matched rollout budget: GRPO 32 tasks × 8 rollouts; PPO 256 tasks × 1 rollout (PPO's critic gets a dedicated value-pretraining stage). Both stable over ~200–400 steps with comparable final results; PPO has smoother entropy, GRPO shows a late-stage entropy decline under OpenClaw; both run hotter under Claude Code, which the authors attribute to the missing cold start rather than the harness. No numeric table — figures only.

**Mix-harness (§4.3).** From base Qwen3-30A3B, GRPO, 32 task–harness instances × 8 rollouts. The mixed model "matches or slightly outperforms" each single-harness model under its own harness; under OpenClaw it closes an initial gap late in training, under Claude Code it is on par or higher throughout. No degradation, no instability. Figure-only; no table of final numbers.

**Cold start (§4.5).** OpenClaw + GRPO: the cold-started policy starts from higher training reward, climbs more smoothly, keeps entropy stable, and finishes stronger; direct-from-base still works but shows "a pronounced late-stage decline in entropy."

**White-box vs black-box (§4.6, Table 2).** A hand-built white-box agent loop (bash, search, fetch, code interpreter, file ops — from SWE-World) exposes the full trace. Same data, same evaluation:

| Eval harness | Qwen3-32B | Qwen3-30A3B | ClawII-OC-30A3B (black-box trained) | WhiteBox-30A3B (white-box trained) |
|---|---|---|---|---|
| White-box AgentLoop | 26.43 | 41.69 | 51.37 | **59.90** |
| OpenClaw | 40.32 | 45.11 | **62.62** | 50.33 |

In-loop, white-box RL wins big (+18.21 over base, +8.53 over the black-box model). But it transfers to OpenClaw at only **50.33 (+5.22 over base)** versus **62.62** for the model trained *through* OpenClaw. The authors' reading: harnesses share transferable agentic capability, but "training under a simple white-box agentloop may still be insufficient to fully capture the harness-specific interaction patterns required by an unseen black-box system." Read diagonally, the same weights swing **11.25 points** (ClawII-OC) or **9.57 points** (WhiteBox) depending on which harness runs them — a clean same-weights scaffold effect.

## Where it breaks

- **No error bars anywhere.** Single runs; Pass@1 on a 30-task PinchBench subset means one task ≈ 3.3 points, so PinchBench deltas under ~7 are one or two tasks.
- **Asymmetric initialization** (cold start for OpenClaw, none for Claude Code) is imposed by data availability and confounds every cross-harness comparison, including the entropy differences the paper itself flags.
- **Auxiliary trajectories are thrown away.** Subagent and compaction branches — the very forks the prefix tree exists to represent — receive no gradient. The tree is used to *find and prune* forks, not to train on them; using them is named future work.
- **PPO on the tree is admitted to be a simplification** (γ = λ = 1, independent branches).
- **Mix-harness results are figure-only** with the claim "matches or slightly outperforms"; no table.
- **The judge is GPT-5.4 for rubric tasks**; a 0.3 rubric weight is part of the ClawGym-Bench score, so some of the gain is judge-satisfaction ([[llm-as-a-judge]]).
- **The token-in-token-out discipline is exactly the "buffered token replacement" that [[agent-lightning-v1-harnessed-agentic-rl-he-2026|Agent Lightning v1.0]] argues is off-policy stitching** — see the contradiction below. ClawGym II's importance correction targets engine mismatch, not prompt mismatch.

> [!warning] Contradiction — is splicing sampled tokens into the next prompt correct?
> **This paper:** the generated tokens are grafted onto the prefix tree as the sole training record, and the harness's re-serialized text is never encoded back — so the trained sequence "is by construction identical to the one the policy model sampled."
> **[[agent-lightning-v1-harnessed-agentic-rl-he-2026|Agent Lightning v1.0]] (Microsoft, one day later):** replacing the retokenized response segment in `pᵢ₊₁` with the originally sampled `aᵢ` produces a stitched prompt `p̃ᵢ₊₁ ≠ pᵢ₊₁`; since `aᵢ₊₁` was sampled under `pᵢ₊₁`, training under `p̃ᵢ₊₁` "introduces an off-policy discrepancy." Agent Lightning therefore keeps **no** buffer and merges only when observed token IDs already match. Both cannot be fully right: ClawGym II's construction guarantees the *response* tokens are exact but can make the *prompt* for the next turn differ from what the policy actually consumed; Agent Lightning's guarantees the prompt is exact but fragments the rollout (64% of its coding rollouts split). Neither paper measures the size of the other's error. Unresolved.

## Agent Lightning v1.0 vs. ClawGym II — same boundary, different answers

| | [[agent-lightning-v1-harnessed-agentic-rl-he-2026\|Agent Lightning v1.0]] | ClawGym II |
|---|---|---|
| Name for the paradigm | "harnessed agentic RL" | "black-box RL on agent harness" |
| What the harness is | deploy-time harness, open and instrumentable (mini-SWE-agent, LLM-in-Sandbox) | production binary treated as opaque (OpenClaw, Claude Code), with pseudo-streaming and settling hacks to keep it happy |
| Sample construction | **best-effort linear merging** — extend a sequence only on exact token-prefix match, else close it; 36% of rollouts stay one sample, mean 2.41 | **rollout-level prefix tree** — attach by longest prefix, shared prefix stored once, forks kept as branches |
| Retokenization | named as a first-class challenge with three mechanisms; buffered replacement rejected as off-policy stitching | not named; handled by token-in-token-out grafting (i.e. buffered replacement) + token-level IS correction for engine mismatch |
| Advantage | rollout-level (agree) | rollout-level, once per rollout (agree) |
| Loss normalization | rollout-level token-mean, to stop multi-sample rollouts dominating | shared-prefix tokens counted once per rollout — the same goal by tree structure |
| Subagent / compaction branches | kept as extra samples inheriting the reward; credit assignment deferred | **discarded** as auxiliary; credit assignment deferred |
| Algorithms | GRPO, RLOO | GRPO and PPO (simplified on the tree) |
| Multi-harness | no | mix-harness training with per-(task, harness) groups |
| Infrastructure emphasis | ~3,500 LOC, Kubernetes Jobs, collocated async, idempotent gateway | temporary sandboxes + in-sandbox MCP servers, fault handling |
| Evidence | ablation of advantage/normalization choices; SWE-bench Verified 41.8 → 56.4 | Table 1 across 2 harnesses × 2 sizes; white-box vs black-box transfer |

**Does the prefix tree solve what Agent Lightning names?** Partly. (1) *Sample merging under forks* — yes: the tree is exactly the "prefix-shared / tree-structured training" option Agent Lightning lists as principled but backend-intensive, and ClawGym II ships it (the paper does not say whether it uses branch-aware attention masks or flattens root-to-leaf paths with a once-per-prefix loss mask). (2) *Advantage calculation* — both land on rollout-level; the tree does not change that. (3) *Loss normalization* — the once-per-rollout prefix counting is a structural version of Agent Lightning's rollout-level token-mean. (4) *Retokenization* — **no**: the tree's grafting is the buffered-replacement strategy Agent Lightning explicitly rejects, and ClawGym II's only correction term addresses inference/training engine drift, not prompt stitching. Where the two do agree, they agree against slime and AReaL (sample-level advantage).

## Related

- [[agent-lightning-v1-harnessed-agentic-rl-he-2026]] — the closest paper; contrast above.
- [[harnessed-agentic-rl]] — the paradigm both papers instantiate (**no wiki page yet**).
- [[molt-pytorch-native-agentic-rl-framework-hu-2026]] — NVIDIA's token-in/token-out loopback server; ClawGym II's TITO discipline and Molt's "never trains on a token it did not generate" are the same invariant, and Molt also seals segments at compaction.
- [[openclaw]], [[claude-code]] — the two harnesses trained through, unmodified. The base-model gap between them (Qwen3-30A3B 45.11 vs 37.06) is itself a harness-attribution datapoint.
- [[harness-vs-model-attribution]] — Table 2 is a same-weights, two-harness factorial: 11.25-point swing for ClawII-OC-30A3B, inside the 11–15 scaffold-only band that page records.
- [[agentic-rl]] — the "traditional" white-box agent loop of §4.6 is that paradigm; the paper shows it transfers only partially to a real harness.
- [[grpo]], [[ppo]] — both adapted to the tree; PPO with γ = λ = 1 and independent branches.
- [[outcome-based-reward]] — rollout-level reward from the final workspace state, rule-based or rubric-based.
- [[llm-as-a-judge]] — GPT-5.4 rubric judge for the 0.3-weighted rubric component.
- [[mcp]] — in-sandbox MCP servers are how capabilities are added without touching the harness.
- [[tool-call-parser]] — the pseudo-streaming safeguard exists because incremental parsers truncate correct calls.
- [[harness-handbook-wang-2026]] — cited as the "harness design, optimization, and self-evolution" line this paper is complementary to (train the model to exploit the harness, rather than evolve the harness).
- [[agentic-harness-engineering]] — the inverse lever; ClawGym II freezes the harness entirely and moves only the weights.
- [[evo-bench-can-language-models-improve-agent-harness-huang-2026]] — from the same Renmin group; uses Claw-Eval, whereas this paper evaluates on ClawGym-Bench / PinchBench.
- [[openclaw-skill-collective-skill-tree-search-lin-2026]] — the other route to better OpenClaw agents (skills, not RL); also reports on PinchBench.
- [[openclaw-rl-train-agent-by-talking-wang-2026]] — an online server–client RL loop for OpenClaw; ClawGym II is the offline, sandboxed, benchmark-verified counterpart.
- [[terminal-bench]], [[swe-bench]] — cited as the coding-side task families; JobBench and OfficeQA are the office/analysis ones (**no wiki pages** for ClawGym-Bench, PinchBench, JobBench, OfficeQA, or ClawGym I).
- [[qwen]] — Qwen3-8B / -30A3B backbones; Qwen3-235A23B as the scale baseline they beat.
- [[react]] — the "handcrafted agent loop" the harness abstraction is contrasted with.

## References

- [arXiv:2608.16798](https://arxiv.org/abs/2608.16798)
- ClawGym (predecessor): Bai et al., arXiv:2604.26904
- Prefix-tree construction credited to Kat-Coder-V2 (arXiv:2603.27703); token-in-token-out to Gallouédec & Rasul (HF blog); IS correction to Yao et al. 2025
- Compared / cited frameworks: Polar (arXiv:2605.24220), Dressage, OpenForgeRL (arXiv:2607.21557)
