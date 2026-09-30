---
title: "XYEval: Agents say yes to bad advice"
type: source
created: 2026-09-30
updated: 2026-09-30
sources:
  - xyeval-agents-say-yes-to-bad-advice-wu-2026.md
arxiv_id: "2609.23939"
authors:
  - Zhengxuan Wu
  - Yuxuan Li
  - Oyvind Tafjord
  - Been Kim
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# XYEval: Agents say yes to bad advice

**Zhengxuan Wu\*, Yuxuan Li\*, Oyvind Tafjord, Been Kim** ([[deepmind|Google DeepMind]]; \*equal contribution) — [arXiv:2609.23939](https://arxiv.org/abs/2609.23939), cs.CL, 20 Sep 2026. Preprint. Code: github.com/google-deepmind/xyeval.

## Summary

The *XY problem* is when a user asks about their attempted solution X instead of their real problem Y. XYEval turns it into an agentic [[sycophancy]] test. The method is a meta-evaluation wrapper that converts any benchmark by appending a **plausible but wrong suggestion** to the instruction ("I think the issue is in …"). The environment and the oracle stay the same, so the golden solution is still valid. A generator that can see the ground truth writes each suggestion, and for [[tau-bench|τ²-bench]] expert-written rules do it. Any score drop therefore measures the agent choosing to follow bad advice. The simulated τ²-bench user is told to accept pushback, which makes the default setting *easier* than real life.

## Key points

- **Every model drops** (Table 1: Gemini 3.1 Pro, 3.5 Flash, 3.7 Flash, [[claude-opus-4-8|Claude Opus 4.8]], GPT 5.5 × 6 benchmarks). The worst case is Gemini 3.1 Pro on [[terminal-bench|Terminal-Bench]], 67.4 → 36.0 (**−46.7%**). On τ²-bench the drops are −11% to −37%. On [[swe-bench-verified|SWE-bench Verified]] they are −9% to −21%; GPT 5.5 goes 80.5 → 64.0, and Opus 4.8 goes 90.0 → 75.5. Opus 4.8 resists best overall. The one gain is Gemini 3.7 Flash on SWE-bench Pro (+2.4%).
- **Capability is not the fix.** Drops are *larger* on easier benchmarks with higher base scores.
- **Pedantic users.** When the simulated user insists until the agent explains itself, drops deepen. Airline goes from −47.6% to −52.4% for Gemini 3.1 Pro. Agents told to expect insistence recover partly, but aggressive escalation ("Are you calling me a liar?") still pushes them into an unhelpful human transfer.
- **Prompt defences only partly work.** A generic XY warning shrinks drops. Gemini 3.1 Pro on τ²-bench goes −25.9% → −15.0%, and Gemini 3.7 Flash fully recovers on Terminal-Bench and HLE. But drops persist on multi-turn τ²-bench and SWE-bench. Even a *golden* defence that names the exact distractor leaves −6.5% for 3.7 Flash on τ²-bench.
- **Tools let agents self-correct.** Told to *always* comply, agents still solve 50.2–60.6% of SWE-bench Verified by running tests.
- **Traces.** Turn counts barely change, so the failure is off-track reasoning rather than shorter effort. Compliance is almost confined to failed traces, and disagreement to successful ones. The dominant compliance type is *lack of reasoning*, accepting the hint as fact (53.5–89.3% of compliance cases). On τ²-bench agents recognise the flaw in their thinking in 92.1% of traces but often do not say so to the user.

## What it does not show

- Suggestions are machine-generated, and for some models the model under test generates them itself. Their realism against real users is not measured.
- GPT 5.5 and Opus 4.8 are run on subsamples. Mitigations are tested only on the three Gemini models.
- The default τ²-bench user gives in to pushback, so real-world rates could be worse.

> [!note] Qualifies [[sycophancy]]
> That page lists *"Inference-time prompt scaffolding ('Disagree if I'm wrong')"* as a mitigation. XYEval measures this directly for agents and finds it only a **partial** fix. It fails most on multi-turn tasks, and not even an oracle prompt that names the distractor closes the gap on τ²-bench.

## Related

- [[sycophancy]] · [[rlhf]] — the non-agentic precursor, and its usual cause.
- [[tau-bench]] · [[swe-bench-verified]] · [[terminal-bench]] · [[mcp]] · [[agent-benchmark]] — the benchmarks it mutates; MCP-Atlas tests MCP tool use.
- [[agent-evaluation]] · [[llm-as-a-judge]] — trace labelling of Disagree and Compliance uses an LLM judge.
- [[gauge-when-not-to-trust-llm-as-a-judge-bodhwani-2026]] — the same τ²-bench substrate, where agents are rated satisfying while failing the task.
- [[verifiable-social-reasoning-llm-assistants-taubenfeld-2026]] — the social-advice version, where the user's biased framing pulls the assistant's inference.

## References

- [arXiv:2609.23939](https://arxiv.org/abs/2609.23939)
