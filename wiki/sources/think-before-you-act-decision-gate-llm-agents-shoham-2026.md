---
title: "Think Before You Act: Decision Gate for LLM Agents"
type: source
created: 2026-11-10
updated: 2026-11-10
sources:
  - think-before-you-act-decision-gate-llm-agents-shoham-2026.md
arxiv_id: "2610.38620"
authors:
  - Iftach Shoham
  - Yuval David
  - Offer Akrabi
  - Erez Bilgory
  - Ido Levy
  - Sami Marreed
  - Sergey Zeltyn
  - Segev Shlomov
year: 2026
venue: Preprint (arXiv cs.AI)
status: complete
importance: high
tags:
  - 2026
  - paper
---

# Think Before You Act: Decision Gate for LLM Agents (JevAct)

**Source:** Iftach Shoham, Yuval David (equal contribution), Offer Akrabi, Erez Bilgory, Ido Levy, Sami Marreed, Sergey Zeltyn, Segev Shlomov (IBM SIL) — [arXiv:2610.38620](https://arxiv.org/abs/2610.38620), cs.AI, v1 28 Oct 2026. Preprint.

## Summary

**JevAct** is a runtime gate added to a [[react|ReAct]] agent loop. It checks selected proposals before they execute: **write calls** (booking, cancelling, changing a setting) and **hand-offs** (ending the attempt). Offline, a compiler turns a domain policy into fixed-choice questions, Boolean rules, executable code checks over tool state, and a hand-off checklist. At runtime, code verifies facts found in tool records. [[kev|Kev-27B]], an open Jev-style [[system-one-model|decision model]], answers the questions that need the conversation, such as whether the user actually asked for the action. A flagged proposal goes back to the agent with targeted feedback and at most two revisions. The agent still chooses actions; the decision model only verifies them. The authors contrast this with JevLoop (ZJUNLP), which gives action selection to the decision model.

## Key points

- **Conservative threshold.** A question counts as "no" only when Kev is confident, P(yes) ≤ 0.05. The call is flagged if code finds a violation or the rule over answers fails.
- **Results across 11 agent LLMs** (GPT-4.1, Claude Opus 4.6, gpt-oss-20b/120b, Qwen3.8-27B, DeepSeek-V4-Pro, Haiku 4.5 and others) on [[tau-bench|τ²-bench]] Airline/Telecom and BFCL. On Telecom all 8 agents improve both at-least-once success and pass^3 (success in all three runs). On BFCL all 9 improve pass^3, with mixed at-least-once results. Airline is smaller and mixed.
- **Code does most of the work.** Single-trial Airline ablation with GPT-4.1: ReAct 50.0% → full gate 72.0%. Code checks alone give 68.0%, Kev questions alone 57.1%. Flagging every write once without checking gives no gain (50.0%).
- **Revision opportunity can explain the gain.** For gpt-oss-120b as agent, an unconditional re-check or random flags at the gate's rate raise pass^3 as much as the full gate. So whether the judge's verdicts matter depends on the agent.
- **Cheap judge.** Kev-27B is competitive with generative judges (70.0% vs 67.3–72.0% with GPT-4.1) at an estimated $0.0003–0.0017 judge cost per task. Open-weight LLM judges cost $0.0020–0.0040 and API judges $0.0127–0.0231. Check cost does not grow with question count, since questions share one prefill.
- **Size does not reliably help.** Kev-9B agrees with Kev-27B on 97.0% of 609 replayed hand-off states and Kev-0.8B on 90.6%. Smaller judges miss more invalid calls on long inputs beyond their validated context length.

## Caveats

The authors state the results do not establish uniform gains or isolate the judge's contribution. With a gpt-oss-120b user simulator the Telecom gain disappears (pass^3 −1.8, CI −7.9 to +4.4). Of 101 incorrect write calls in 260 labelled ones, 42 break no compiled question, because the questions encode the policy rather than the user's request.

## Concepts & entities

- [[system-one-model]] · [[kev]] · [[jev]] · [[typesafe-ai]] · [[logit-scoring]] — the prefill-only judge.
- [[guardrailing]] · [[verifier]] · [[llm-as-a-judge]] — policy enforcement before execution, with probabilities instead of rationales.
- [[react]] · [[reflexion]] · [[tool-use]] · [[function-calling]] · [[tau-bench]] — agent loop and benchmarks.
- [[calibration]] — the authors note decision-model probabilities are not guaranteed calibrated and must be tested in the loop.
- Related: [[jevagentharness-decision-models-share-control-zhu-2026]] · [[jev-as-a-judge-accept-when-confident-li-2026]].

## References

- [arXiv:2610.38620](https://arxiv.org/abs/2610.38620)
