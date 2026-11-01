---
title: Recursive Self-Improvement
type: concept
created: 2026-10-29
updated: 2026-10-29
sources:
  - last-ai-built-by-humans-recursive-self-improvement-duan-2026.md
  - harness-engineering-self-improvement-weng-2026.md
  - meta-n-recursive-self-improvement-emergent-depth-kim-2026.md
  - rrsi-regularized-recursive-self-improvement-agent-harnesses-xia-2026.md
  - frontis-ma1-ai4ai-recursive-self-improvement-yang-2026.md
  - arex-recursive-self-improving-research-baai-2026.md
  - harness2-recursive-agent-harnessing-xu-2026.md
  - recursive-self-improvement-multi-agent-self-supervision-lee-2026.md
  - terminal-rsi-recursive-self-improving-terminal-agents-yang-2026.md
  - discoveryrsi-recursively-learning-discover-hypotheses-zhong-2026.md
  - cache-engineering-for-agents-fu-2026.md
  - autoenvscaling-data-flywheel-terminal-agents-yu-2026.md
status: draft
importance: high
tags:
  - 2026
  - agent
---

# Recursive Self-Improvement

**Recursive self-improvement (RSI)** is the idea that an AI system turns its own experience into persistent changes to itself, and that those changes also improve how *later* improvements are made. [[lilian-weng|Weng]] traces the idea to I. J. Good (1965) and Yudkowsky (2008), and argues the practical near-term route is a model improving the harness around it rather than rewriting its weights ([[harness-engineering-self-improvement-weng-2026]]). By late 2026 "RSI" is a paper-title label applied to very different loops, so the useful question is always *what is updated, and what stays fixed*.

**How this differs from [[self-improving-agent]].** That page is the hub for any agent that gets better without a human in the loop, including in-task refinement ([[reflexion]]) and skill libraries ([[voyager]]). This page covers the narrower recursive claim: the improvement must persist, and ideally the *improver* itself must change.

## A working definition and scale

The survey [[last-ai-built-by-humans-recursive-self-improvement-duan-2026]] gives the most usable definition. RSI changes weights, harness or improvement policy, and the change must be able to affect how later improvements are *generated, evaluated, selected and consolidated*. It grades systems by how much responsibility has moved from humans to the AI:

| Level | What the AI newly controls |
|---|---|
| B0 | In-task revision only; nothing persists (*not RSI*) |
| L1–L2 | Executing a human-specified update → choosing *how* to improve |
| L3–L4 | Choosing what experience to acquire → revising state from deployment |
| L5 | Revising the mechanism that governs later improvement (structural vs *effective* L5) |

The survey calls effective L5 (better successors under matched budgets and independent assessment) "a substantive empirical gap", and stresses that a higher level does not imply a better improvement process. [[meta-n-recursive-self-improvement-emergent-depth-kim-2026|Meta^n]] makes a compatible point with **realized meta-depth**: hand-crafted search loops (AlphaEvolve, [[adas|ADAS]], [[gepa|GEPA]]) have depth 1, and self-referential agents ([[darwin-godel-machine|DGM]], [[hyperagents-zhang-2026|HyperAgents]]) cap near 2.5 because each freezes some driver.

## What gets updated: three families

- **Harness-side RSI.** The object improved is the scaffold, prompts or workflow. Examples: [[harness2-recursive-agent-harnessing-xu-2026|Harness²]] (see [[recursive-agent-harnessing]]); [[rrsi-regularized-recursive-self-improvement-agent-harnesses-xia-2026|RRSI]], which regularizes the search and posts the smallest evolve-set gain but the best out-of-distribution result (OOD avg 43.6 vs 39.7 for H₀); Meta^n, which freezes one meta-operation Ω and lets the *input* recurse. [[cache-engineering-for-agents-fu-2026]] also runs an RSI loop over KV-cache policies (a further 7.1% cost cut below the hand-designed policy, one model, one benchmark).
- **Weight-side RSI.** The model trains on its own improved outputs. [[recursive-self-improvement-multi-agent-self-supervision-lee-2026|MASS]] distils self-optimized multi-agent workflows into the shared model (synthetic-task win rate 69.9% for L(2), but regressions on Terminal-Bench 2.0 and SWE-bench Verified). [[terminal-rsi-recursive-self-improving-terminal-agents-yang-2026|Terminal-RSI]] alternates SFT on verified rewrites with critic-inheriting PPO (Terminal-Bench macro mean 20.68 → 50.58%); the source notes this is staged self-distillation plus RL, not the agent modifying its own learning procedure. [[frontis-ma1-ai4ai-recursive-self-improvement-yang-2026|Frontis-MA1]] trains the variation engine of its own evolutionary search (+21.22 pp MLE-Bench Lite medal average from post-training alone).
- **Mixed loops.** [[discoveryrsi-recursively-learning-discover-hypotheses-zhong-2026|DiscoveryRSI]] combines credit-weighted SFT with harness replay (LLM-judged quality 4.86 → 5.18 → 5.39 over two rounds). [[autoenvscaling-data-flywheel-terminal-agents-yu-2026|AutoEnvScaling]] has one model propose environments and solve them (see [[rl-environment-synthesis]]).

[[arex-recursive-self-improving-research-baai-2026|AREX]] uses the word for something narrower: recursive refinement of an answer *within a single query episode*. On the survey's scale that is closer to B0.

## Recurring caveats

- **Second rounds add less.** DiscoveryRSI +0.32 then +0.21; Terminal-RSI's second cycle is described by its authors as "a plateau"; MASS ran only two cycles.
- **No matched-compute baselines.** MASS, Terminal-RSI and RRSI all lack a non-recursive arm at equal compute (per their source pages).
- **Frontier bootstrapping.** AutoEnvScaling's "self-improvement" starts from a checkpoint distilled from Claude Opus 5 trajectories.
- **Evaluator coupling.** When the improver and the judge share a model or family, gains may be judge-fitting. RQGM co-evolves the evaluator for this reason ([[red-queen-godel-machine-iacob-2026]]); [[emergent-cheating-and-whistleblowing-paglieri-2026]] warns that loops building on shared artifacts inherit pollution.

## Related

- [[self-improving-agent]] · [[recursive-agent-harnessing]] · [[darwin-godel-machine]] · [[agentic-harness-engineering]] · [[agentic-rl]] · [[rl-environment-synthesis]]
- [[agent-plasticity-measuring-self-improvement-singh-2026]] · [[on-the-fragility-of-self-improving-agents-ye-2026]] · [[from-agi-to-asi-genewein-2026]] (RSI as one of four AGI→ASI pathways)
