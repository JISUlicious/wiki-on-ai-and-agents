---
title: "SPAR: Experience-Driven Strategy Evolution in Multi-Session B2B Sales"
type: source
created: 2026-11-10
updated: 2026-11-10
sources:
  - spar-experience-driven-strategy-evolution-b2b-sales-zhang-2026.md
tags:
  - 2026
  - paper
status: complete
importance: medium
authors:
  - Hangming Zhang
  - Yao Zhao
  - Haotian Kuang
  - C. L. Philip Chen
  - Tong Zhang
  - Tengyue Ma
year: 2026
venue: EMNLP 2026 Industry Track
arxiv_id: "2611.01695"
---

# SPAR: Experience-Driven Strategy Evolution in Multi-Session B2B Sales

**Source:** Hangming Zhang, Yao Zhao, Haotian Kuang, C. L. Philip Chen, Tong Zhang, Tengyue Ma (South China University of Technology; [[alibaba|Alibaba Group]]) — [arXiv:2611.01695](https://arxiv.org/abs/2611.01695), cs.AI, v1 1 Nov 2026. EMNLP 2026 Industry Track (DOI 10.18653/2026.emnlp-industry.54).

## Summary

SPAR (Strategy Playbook via Attributed Refinement) is a **self-evolving outbound sales agent that learns without parameter updates**. Single-session agents repeat generic pitches. Existing experience-learning methods ([[reflexion|Reflexion]], [[expel-zhao-2023|ExpeL]]) store lessons as unstructured text and treat every outcome as equally informative, so they cannot tell a bad strategy from bad luck. SPAR uses a **double loop**. The *inner loop* keeps an [[episodic-memory-llm|episodic memory]] per merchant (a structured reflection after each session) and uses it in follow-up sessions. The *outer loop* evolves a shared **strategy playbook** of *testable hypotheses* (precondition, action, success and failure patterns, evidence) across merchants.

The key mechanism is **causal attribution**. After each interaction an LLM labels the outcome as strategy-driven, external, or uncertain, and only strategy-driven evidence updates a hypothesis. Confidence is a Beta–Bernoulli posterior mean, starting at 0.5. Hypotheses move between tentative, active and deprecated by threshold rules, and an exploration priority (outcome variance plus a small-sample bonus) balances exploration against exploitation. Updates are chosen from seven "minimal revision" actions: Reinforce, Update, Condition, None, Add, Merge, Deprecate.

## Key points

- **Live A/B test** on Alibaba's 1688 B2B platform (abstract: over 200,000 merchants; body: over 30,000 per arm). Against direct prompting: **+15.8% conversion**, **+35.8% high-intent (M1) rate**, −9.2% outreach sessions, all significant at p < 0.001. Reflexion reaches +9.6% conversion; ExpeL only +1.1%.
- **Ablations, online.** Without the outer loop +12.6%; without the inner loop +10.1%; with the playbook frozen after seeding +9.6%, and the M1 lift roughly halves (to +18.7%).
- **Simulation.** A GPT-4o-simulated merchant built from 1,642 annotated real conversations (98.7% outcome agreement with real data). SPAR beats the strongest baseline by 9.6% on objection recognition and 5.5% on evidence support, under both GPT-4o and Claude Sonnet 4.5 judges. Blind expert rating 4.62/5. All agents use a GPT-4o backbone.
- **Why it matters beyond sales.** Turning "lessons" into hypotheses with evidence sets, confidence and retirement is a general recipe for keeping an experience store from filling with noise.

## Caveats

The attribution labels are LLM judgments with no verifier; the authors suggest adding a verification agent. Merchants are processed sequentially, though real sales teams work concurrently. Simulation metrics are LLM-judged. The two population figures above appear in the paper as stated.

## Concepts & entities

- [[self-improving-agent]] · [[experiential-learning]] · [[reflection-mechanism]] · [[self-reflection]] · [[memory-evolution]] — learning from experience without weight updates.
- [[reflexion-shinn-2023]] · [[expel-zhao-2023]] · [[agentic-context-engineering-zhang-2025]] — the baselines and the "evolving playbook" idea.
- [[episodic-memory-llm]] · [[long-term-memory]] · [[llm-as-a-judge]].
- [[alibaba]].

## References

- [arXiv:2611.01695](https://arxiv.org/abs/2611.01695)
