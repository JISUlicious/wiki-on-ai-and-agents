---
title: "The Agentic Scientist: Orchestrating Coding Agents for Automated Scientific (Co-)Discovery"
type: source
created: 2026-10-29
updated: 2026-10-29
sources:
  - agentic-scientist-orchestrating-coding-agents-lange-2026.md
arxiv_id: "2610.18933"
authors:
  - Robert Tjarko Lange
  - Luca Grillotti
  - Yutaro Yamada
  - Rujikorn Charakorn
  - David Ha
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# The Agentic Scientist: Orchestrating Coding Agents for Automated Scientific (Co-)Discovery

**Robert Tjarko Lange, Luca Grillotti, Yutaro Yamada, Rujikorn Charakorn, David Ha** ([[sakana-ai|Sakana AI]]). [arXiv:2610.18933](https://arxiv.org/abs/2610.18933), v1, cs.AI, 13 Oct 2026. Code: SakanaAI/AI-Scientist-v3 and SakanaAI/Autoillustrate.

## Summary

This is the successor to The AI Scientist v1 (linear, Aider-based) and v2 (tree search, AIDE). The paper argues that fully autonomous research agents have a limited **"ideation radius"** and fall into myopic hill-climbing, behaving like "flexible hyperparameter optimization engines". The Agentic Scientist breaks research into four stages: task setup, parallel investigation, evidence generation and paper writing. Each step is run by a headless CLI coding agent (Claude Code, Codex, Pi or OpenCode). A **meta-controller** runs parallel workers in a beam search and enforces **self-drafted rubric gates**: an attempt only "passes" when it meets all rubrics, and it is "best" only with a strictly higher score. Workers share memory and conclusions. The system can run as a human-steered **co-scientist** or end to end.

## Key points

- **ALIFE 2026 acceptance (co-scientist mode).** A development version helped produce [[diffusing-blame-yamada-2026|*Diffusing Blame*]], on Error Diffusion and Dale's-principle networks. Humans chose the direction, baseline and initial rubrics, verified the results and edited the manuscript. It was the **only paper submitted** and was accepted as a talk, with reviews of Strong Accept, Weak Reject and Weak Accept.
- **Autonomous case studies.**
  - **Meta-learned ES.** A hypernetwork-based inductive bias for fast Transformer adaptation, and a meta-learned evolution strategy (CMA-Anchored LearnedES, now in evosax v0.3.2). The Opus 5 + Claude Code candidate **matches or beats CMA-ES on 6 of 8 Brax environments**, which is 5 of 7 unseen.
  - **RL fine-tuning.** An RL-based LLM fine-tuning program improved by combining recent methods.
- **Harness × model sweep.** Harness choice matters less than model choice. Model developers' own harnesses show "no consistent advantage": Codex ties Pi with GPT-5.6-Sol, and Claude Code falls between OpenCode and Pi with Opus 5.
- **autoillustrate**, an agentic figure generator, scores 37.5 vs 25.0 for PaperBanana on PaperBananaBench.

The newsletter blurb ("staged pipeline + meta-controller with rubric checks; co-wrote ALIFE 2026 accepted paper") is accurate. "Co-wrote" is the right verb; the authors stress that the paper was not unaltered system output.

## What it does not show

- **n = 1 external validation.** The single ALIFE paper had heavy human selection and verification, and the reviews were split.
- **No head-to-head with AI Scientist v2** or other end-to-end systems on shared tasks. The Table 1 comparison with v1 and v2 is a feature checklist.
- **Rubric-specification sensitivity.** The authors list this as a failure mode: rubrics drafted by the system can be under- or over-constrained.
- The autonomous discoveries are judged against the system's own evaluators and baselines. There is no external replication.

## Concepts & entities

- [[sakana-ai]] · [[diffusing-blame-yamada-2026]] · [[dales-principle]] · [[backprop-free-learning]]
- [[plan-execute-verify-loop]] · [[verifier]]: rubric gates as self-verification.
- [[claude-code]] · [[opencode]] · [[pi-mono]]: interchangeable worker harnesses.
- Related automated-science sources: [[scientisttwo-autonomous-ai-human-knowledge-frontier-nam-2026]], [[training-ai-scientists-replicate-research-falck-2026]], [[accelerating-scientific-research-gemini-real-world-schmidgall-2026]], [[discoveryrsi-recursively-learning-discover-hypotheses-zhong-2026]]
- [[multi-agent]] · [[self-improving-agent]]

## References

- [arXiv:2610.18933](https://arxiv.org/abs/2610.18933)
