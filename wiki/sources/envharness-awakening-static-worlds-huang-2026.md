---
title: "EnvHarness: Awakening Static Worlds for Agent Learning"
type: source
created: 2026-09-05
updated: 2026-09-05
sources: []
arxiv_id: "2608.19880"
year: 2026
authors:
  - Chengsong Huang
  - Zifeng Wang
  - Rujun Han
  - Jun Yan
  - Yanfei Chen
  - Zoey CuiZhu
  - Ke Jiang
  - Peng Xia
  - Han Yu
  - Yufan Zhuang
  - Yifei Ming
  - Jiaqi Pan
  - Bhavana Dalvi Mishra
  - Jiaxin Huang
  - Burak Gokturk
  - Tomas Pfister
  - Chen-Yu Lee
tags:
  - 2026
  - paper
status: complete
importance: high
---

# EnvHarness: Awakening Static Worlds for Agent Learning

**Chengsong Huang, Zifeng Wang, Rujun Han, Jun Yan, Yanfei Chen, Zoey CuiZhu, Ke Jiang, Peng Xia, Han Yu, Yufan Zhuang, Yifei Ming, Jiaqi Pan, Bhavana Dalvi Mishra, Jiaxin Huang, Burak Gokturk, Tomas Pfister, Chen-Yu Lee** (Google Cloud AI Research; University of North Carolina at Chapel Hill; Washington University in St. Louis) — [arXiv:2608.19880](https://arxiv.org/abs/2608.19880), cs.AI, 20 Aug 2026, CC BY 4.0. Preprint; no venue. Code: `github.com/google-research/envharness`; site: envharness.com.

## Summary

The paper takes the *harness* idea — a plug-in layer that turns a frozen LLM into a capable agent without touching weights — and applies it to the **other side of the loop**. An **Environment Harness (EnvHarness)** is a programmable layer that wraps a static, hand-built environment and reshapes its behaviour purely through the standard `reset`/`step` interface, leaving the simulator, its tasks, and above all its **human-built verifier** untouched. The paper's own Table 1 makes the analogy explicit: `Agent = Model + Harness`; `Customized Env = Static Env + EnvHarness`. The motivation is that hand-built environments are "blind to an agent's weaknesses, and quickly left behind as it improves," while LLM-generated environments are domain-specific and carry unreliable generated verifiers.

Formally an environment is `E = (S, A, O, T, R, s₀)` and a component is an environment-agnostic transformation `E' = w(E)`. Three component types ship: **Stage** (replay a sequence of actions after `reset()` to change `s₀` — e.g. hide the mug in a drawer so the agent must search), **Contract** (a triplet `(f_A, f_T, f_O)` of hooks that filter/block actions, rewrite transitions, or mask observations — e.g. block `clean mug` unless the mug is held, truncate the room description to two sentences), and **Chain** (compose a second environment with logic `g`; reward is the conjunction of both verifiers). Components stack non-commutatively.

**EnvRigger** automates the choice. It treats the policy as a black box, runs **Observe → Diagnose → Write → Validate**: K = 5 baseline rollouts, a textual root-cause diagnosis (repetitive action loops, unparsed long observations, misread tool constraints; or "too easy — make it harder" at 100% success), synthesis of a candidate component set as Python, then K = 5 fresh rollouts to accept, reject (unsolvable or trivial), or refine — at most 5 write-validate rounds. The designer and the policy are **the same model** on every benchmark (Gemini 3.1 Flash-Lite on ALFWorld/WebArena, Gemini 3.5 Flash elsewhere), so gains cannot come from distilling a stronger teacher.

Headline claims, all verified against Tables 2–4: skill-based learning from EnvHarness environments beats skills from the original environments on all five benchmarks, **up to +9.0 on held-out ALFWorld** (61.4 → 70.4 OOD) with **9.8% fewer steps** on SWE-bench Verified (55.01 → 49.61 vs. original-env skills); in online [[grpo|GRPO]] the reshaped environments give **up to +6.5** (ALFWorld in-distribution 81.4 → 87.9); and under a fixed environment budget on SWE-bench Verified the co-evolving loop climbs **47.67 → 54.79** while real (52.13) and SWE-smith-generated (50.37) environments flatten.

## Results

**Protocol.** Five benchmarks, four domains. Training and evaluation episodes disjoint (Table 7): ALFWorld 100 train tasks / all remaining held-out; WebArena 20 per sub-domain / rest; SWE-bench 100 tasks from SWE-bench Lite / **407 Verified issues not in Lite**; OfficeQA 50 / 172 official test; SpreadsheetBench 100 of 400 verified / 299 held-out (897 instances). Skills are extracted from trajectories in the customized environments following ReasoningBank and retrieved at test time. The **Chain component is excluded from the automated pipeline** ("difficult for EnvRigger to observe the internal states of joined environments") and studied separately. Baselines share seed instances, environment count, extraction pipeline, and policy model. Three runs, mean ± std.

**Table 2 — ALFWorld and WebArena** (success rate):

| Skill source | ALF In-Dist | ALF OOD | ALF Avg | WA Reddit | WA Shopping | WA Shop Admin | WA GitLab | WA Avg |
|---|---|---|---|---|---|---|---|---|
| No Skills | 62.6 ±1.7 | 60.7 ±5.2 | 61.7 ±3.4 | 39.6 ±2.3 | 35.2 ±3.3 | 44.1 ±2.3 | 35.8 ±8.4 | 38.7 ±2.3 |
| Original Envs | 63.3 ±2.8 | 61.4 ±4.3 | 62.4 ±3.4 | 38.7 ±9.7 | 35.2 ±1.3 | 44.6 ±3.0 | 35.4 ±4.0 | 38.5 ±3.1 |
| GenEnv | 63.3 ±1.2 | 61.9 ±2.7 | 62.6 ±1.9 | – | – | – | – | – |
| VeriEnv | – | – | – | 39.6 ±4.2 | 30.2 ±0.0 | 49.7 ±2.4 | 38.9 ±5.6 | 39.6 ±1.4 |
| **EnvHarness Envs** | **66.2 ±0.3** | **70.4 ±2.3** | **68.3 ±1.3** | **40.6 ±4.7** | **37.4 ±0.3** | **50.8 ±1.5** | 37.7 ±3.1 | **41.6 ±1.8** |
| Δ vs Original | +2.9 | **+9.0** | +5.9 | +1.9 | +2.2 | +6.2 | +2.3 | +3.1 |

**Table 3 — SWE-bench Verified, OfficeQA, SpreadsheetBench:**

| Skill source | SWE-V SR ↑ | SWE-V avg steps ↓ | OfficeQA EM | OfficeQA F1 | SSB Pass@1 | SSB Mean Score |
|---|---|---|---|---|---|---|
| No Skills | 47.67 ±0.93 | 53.58 ±2.93 | 54.23 ±2.84 | 55.77 ±2.98 | 46.44 ±0.15 | 61.32 ±0.37 |
| Original Envs | 49.88 ±2.59 | 55.01 ±1.69 | 54.40 ±1.84 | 55.77 ±1.59 | 45.88 ±1.19 | 61.47 ±0.59 |
| SWE-smith | 50.12 ±1.74 | 54.72 ±2.03 | – | – | – | – |
| **EnvHarness Envs** | **52.58 ±2.72** | **49.61 ±2.49** | **56.20 ±2.34** | **57.73 ±2.29** | **49.15 ±0.36** | **62.48 ±0.27** |
| Δ vs Original | +2.70 | +5.40 | +1.80 | +1.96 | +3.27 | +1.01 |

Two things the paper says and one it does not. It says static-environment skills can *hurt*: on SpreadsheetBench original-env skills fall below no-skills (45.88 vs 46.44), and on SWE-bench Verified they *lengthen* episodes (53.58 → 55.01). It says EnvHarness beats every domain-specific generator where one exists — GenEnv by 5.7 avg / 8.5 OOD on ALFWorld; SWE-smith by 2.46 SR and 5.11 fewer steps. What it does not say: **only ALFWorld OOD (+9.0, ~2σ) and SpreadsheetBench Pass@1 (+3.27, ~3σ) are clearly outside one standard deviation.** SWE-bench Verified (+2.70 with σ≈2.6–2.7), WebArena (+3.1, σ≈2–3), and OfficeQA (+1.8, σ≈2) sit inside it. The word "consistent" is doing work that the error bars do not fully support.

**Table 4 — online RL** (Qwen3-8B-base, GRPO, 8×H100, batch 16, 150 steps, temp 0.4, max 50 steps/episode, invalid-action penalty 0.1; same held-out instances):

| Training set | ALFWorld In-Dist | ALFWorld OOD | ALFWorld Avg | WebShop Score | WebShop SR |
|---|---|---|---|---|---|
| Original Envs | 81.4 | **89.6** | 85.5 | 75.6 | 66.0 |
| EnvHarness Envs | **87.9** | 88.8 | **88.4** | **79.2** | **67.4** |

Three of four metrics improve; ALFWorld OOD dips 0.8 (called "negligible"). Single run, no std.

**Table 5 — Chain on SWE-bench Verified** (two randomly paired base environments, success only if both halves succeed; evaluated on ordinary single tasks):

| Skill source | SR | Avg steps |
|---|---|---|
| No Skills | 47.67 | 53.58 |
| Original Envs | 49.88 | 55.01 |
| EnvHarness (Stage/Contract only) | 52.58 | 49.61 |
| EnvHarness (Chain only) | 49.63 | **41.96** |
| **Combined (Stage/Contract + Chain)** | **54.30** | 43.12 |

Chain-only skills buy efficiency (−11.6 steps) at slightly *lower* SR than original-env skills; the two skill sets are complementary. Extracted Chain skills are about budget management across a handoff ("re-orient environment after task handoff").

**Environment scaling (Figure 5).** Same policy, budget, and retrieval; each batch of 50 environments yields one skill bank (15 skills at 300 environments). Baselines draw batches independently of the learner; EnvHarness targets the policy *equipped with previously accumulated skills*, so environment and policy co-evolve. EnvHarness 47.67 → **54.79 (+7.12)** and still rising at 300; original 52.13; SWE-smith 50.37. Appendix F.2 shows what each round teaches — Round 1 forces `pytest -x` and per-file test targeting and blocks inline `python -c` editing; Round 2 breaks the pytest entrypoint so the policy learns `pytest.main`; Round 3 forces interpreter resolution through the conda `bin` path and disables `sed -i`. Gains shrink round over round, and by Round 3 **about a third of training tasks are solved on every baseline rollout, EnvRigger tries to make them harder, and every candidate is rejected** — those tasks contribute nothing.

**Cross-model (Table 9, SWE-bench Verified; designer = policy in each case):**

| Skill source | Gemini 3.1 Flash-Lite SR / steps | Qwen3.6 27B | Gemini 3.5 Flash | Claude Sonnet 4.6 |
|---|---|---|---|---|
| No Skills | 30.7 / 36.7 | 41.0 / 69.8 | 47.7 / 53.6 | 67.2 / 29.3 |
| Original Envs | 36.8 / 50.0 | 48.4 / 37.1 | 49.9 / 55.0 | 69.2 / 25.4 |
| **EnvHarness Envs** | **40.0** / 50.6 | **52.1** / 40.8 | **52.6** / 49.6 | **72.4** / 25.6 |

The EnvHarness-over-original gain is **2.7–3.7 points on every backbone** across a 30.7–67.2 skill-free range — the loop neither breaks on the weakest nor saturates on the strongest. Steps tell three stories: skills nearly halve Qwen's flailing (69.8 → 37.1), make Flash-Lite *persist* instead of quitting (36.7 → 50), and barely move Sonnet.

**Appendix G.** Leave-one-out on ALFWorld task types: +3.1 average, **+16.4 on `clean`, −8.7 on `heat`** — one real regression. Token cost: design tokens are 1.46M vs GenEnv's 38K, but rollouts dominate; against VeriEnv (also real execution) totals match (137.3M vs 137.8M); GenEnv's 3.5× lower total buys LLM-simulated rollouts. Objective targeting on 100 ALFWorld tasks: tasks with per-task SR in [0.4, 0.6] rise from **6.0% to 80.0%** (mean SR 0.74 → 0.48); average-steps-in-[25,35] from 18.0% to 53.0%. Nine "specified weakness → component → distilled skill" cases (e.g. *submits without running tests* → a Contract that fails submission with a fake `pre-commit hook 'verify-tests' failed` message → skill "Verification-Driven Development Loop").

## Where it breaks

- **The evaluation paradigm is mostly skill extraction, not RL.** Four of five benchmarks are evaluated via ReasoningBank-style skill retrieval on a frozen policy; the RL result is one Qwen3-8B run on two text games. The "substrate for RL" claim rests on Table 4 alone.
- **Error bars swallow three of five headline gains** (see above). The paper reports them honestly and then summarizes past them.
- **Chain is not automated**, and Chain-only skills lower SR. The one component that extends horizon is hand-paired.
- **The harder-direction loop fails at saturation** (Round 3): once the policy solves a task reliably, EnvRigger cannot find an accepted harder variant within five revisions.
- **Requires a resettable gym-style interface** (Appendix H) — Stage replays actions after `reset()`, so live services, real accounts, and physical robots are out of scope. Text-only; no GUI/visual.
- **Designer cost** is paid per environment (1.46M design tokens on ALFWorld) and scales with designer weakness.
- **Same-model designer is a control, not a ceiling** — the paper never tests whether a stronger designer helps, so it cannot say whether EnvRigger is designer-limited.
- **Baseline generators are not tuned for this protocol**: GenEnv, VeriEnv, and SWE-smith each get "the same number of environments," but they were built to be run at scale, not at 100–300 seeds.

## Relation to the harness cluster

EnvHarness is the environment-side mirror of [[agentic-harness-engineering]]: that cluster freezes the model and evolves the scaffold; this paper freezes the *environment* (and the policy) and evolves a scaffold around the environment. The closest structural twin is [[harness-r1-learning-to-edit-runtime-harnesses-shao-2026|Harness-R1]]'s lifecycle hooks — its **pre-action veto** and **post-feedback** hooks are exactly a Contract's `f_A` (`Blocked`) and `f_T`, but installed on the *agent* side to help it, where EnvHarness installs them on the *environment* side to make the task harder in a targeted way. [[adapting-the-interface-not-the-model-xu-2026|LIFE-HARNESS]] sits between them. Against the RL-through-harness line ([[agent-lightning-v1-harnessed-agentic-rl-he-2026|Agent Lightning v1.0]], [[clawgym-ii-black-box-rl-agent-harness-song-2026|ClawGym II]]), which keeps the deploy-time harness fixed and moves weights, EnvHarness supplies the *other* half of the RL substrate — what the fixed harness is pointed at — and its Table 4 shows GRPO through a wrapped environment gives a better policy than GRPO through the raw one. The matched-budget objection of [[rethinking-evaluation-harness-evolution-wang-2026]] applies here with less force: baselines are matched on environment count, policy, and extraction pipeline, and the token accounting is published (Table 11).

## Related

- [[agentic-harness-engineering]] · [[code-as-harness]] · [[agent-three-layer-model]] — the agent-harness concept the paper transplants; Table 1 is the explicit analogy.
- [[harness-r1-learning-to-edit-runtime-harnesses-shao-2026]] · [[adapting-the-interface-not-the-model-xu-2026]] — agent-side lifecycle hooks; Contract is the environment-side twin.
- [[meta-harness-lee-2026]] — cited as the "harness rewritten and tested around a frozen model" line that EnvHarness contrasts with.
- [[agent-harness-survey-meng-2026]] — the harness definition (execution loops, tool registries, context management) the paper adopts.
- [[agent-lightning-v1-harnessed-agentic-rl-he-2026]] · [[clawgym-ii-black-box-rl-agent-harness-song-2026]] — RL through a fixed harness; EnvHarness is the environment-side complement.
- [[agentic-rl]] · [[grpo]] · [[outcome-based-reward]] · [[verifier]] — the RL arm; the design constraint that the original verifier is never touched is the paper's answer to generated-verifier unreliability.
- [[self-improving-agent]] — filed under "self-evolving agent" related work; here the *world* evolves against a frozen policy, and the co-evolution loop in Figure 5 is a policy–environment arms race with a hard ceiling at saturation.
- [[agent-skills]] · [[skill-acquisition]] · [[skill-library]] · [[experiential-learning]] — the skill-based learning paradigm (ReasoningBank extraction) that carries four of five benchmarks.
- [[agent-world-real-world-environment-synthesis-dong-2026]] — Appendix B contrasts programmatic synthesis-from-scratch (Agent-World) with wrapping an existing trusted benchmark.
- [[qwen-agentworld-language-world-models-for-general-agents-qwen-2026]] · [[world-model]] — the "simulate whole families of environments" line in related work; EnvHarness deliberately does *not* simulate, it wraps real execution.
- [[metaclaw-just-talk-agent-meta-learns-xia-2026]] · [[reflexion]] · [[voyager]] — the self-evolving-agent lineage cited in §6.2.
- [[alfworld]] · [[webshop]] · [[swe-bench]] · [[swe-bench-jimenez-2023]] · [[swe-gym]] — environments used or contrasted. **No wiki pages** for WebArena, OfficeQA, SpreadsheetBench, SWE-smith, ReasoningBank, GenEnv, VeriEnv.
- [[swe-bench-live-zhang-2025]] · [[terminal-bench]] — cited for the cost of building environments by hand.
- [[reward-hacking]] — the Contract that blocks `git reset`/`checkout` and the "guesses URLs" case are shortcut-closing, the same instinct as reward-hack guards.
- [[rethinking-evaluation-harness-evolution-wang-2026]] — the matched-budget standard; largely met here.
- [[google]] · [[qwen]] · [[claude]] — Google Cloud AI Research authorship; Qwen3.6 27B, Qwen3-8B-base, and Claude Sonnet 4.6 as policies.

## References

- [arXiv:2608.19880](https://arxiv.org/abs/2608.19880)
- Code: https://github.com/google-research/envharness · https://www.envharness.com
- Baselines: GenEnv (arXiv:2512.19682), VeriEnv (arXiv:2603.10505), SWE-smith; skill extraction per ReasoningBank (arXiv:2509.25140)
