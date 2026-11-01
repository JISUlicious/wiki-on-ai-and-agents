---
title: "Coding Agents are Strong Prompt Optimizers (CASD)"
type: source
created: 2026-10-29
updated: 2026-10-29
sources:
  - coding-agents-strong-prompt-optimizers-singh-2026.md
arxiv_id: "2609.26261"
authors:
  - Agamdeep Singh
  - Nikita Mehrotra
  - Srishti Gautam
  - Tanmay Bakshi
  - Priyanshu Gupta
  - Sumit Gulwani
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# Coding Agents are Strong Prompt Optimizers (CASD)

**Agamdeep Singh, Nikita Mehrotra, Srishti Gautam, Tanmay Bakshi, Priyanshu Gupta, Sumit Gulwani** ([[microsoft|Microsoft]]) — [arXiv:2609.26261](https://arxiv.org/abs/2609.26261), v1, cs.AI, dated 13 Aug 2026. Preprint.

## Summary

**Coding-Agent Skill Distillation (CASD)** replaces the iterative loop of search-based [[prompt-optimization|prompt optimizers]] (propose, roll out, score, keep) with **one offline pass**. An unmodified coding agent ([[claude-code|Claude Code]] on Claude Sonnet 5) is pointed at a directory containing a static corpus of agent rollouts and given a one-paragraph instruction: analyze the corpus and write a skill markdown file. That file becomes the optimized system prompt. There is no environment access, no validation set and no acceptance gate.

The claimed key variable is **reflection scope**. Minibatch reflection in [[gepa|GEPA]]-style optimizers sees only a few trajectories at a time. The coding agent instead writes code to compute corpus-wide statistics: per-category pass rates, tool-call histograms, duplicate calls and argument hallucinations. It then reads the episodes those statistics flag.

## Key points

- **Setup.** The target agent is GPT-5.4-mini with reasoning off. The optimizer is Claude Sonnet 5 for all methods. Benchmarks are ALFWorld, τ²-bench retail and telecom, and SpreadsheetBench-Verified. Pools are small: 35–50 tasks.
- **Limited-data regime (headline).** Every optimizer sees the same static pool. CASD scores 83.3 / 40.0 / 39.2 / 51.3, against the baseline's 56.7 / 32.5 / 19.2 / 39.3. **Mean gain is +16.6 pp, versus +10.9 for GEPA and +5.3 for [[skillopt-yang-2026|SkillOpt]].** CASD beats GEPA on 3 of 4 benchmarks; GEPA wins SSB, 60.7 vs 51.3.
- **Telecom shows overfitting.** GEPA drops *below* baseline (17.5 vs 19.2) because it selects on the same pool it reflects on. CASD gains +20.0.
- **Corpus-level absences.** In one telecom case, the needed `make_payment` call is never made anywhere in the pool. Only CASD writes a rule for it, because the gap shows up only as an absence across the whole corpus.
- **Head-to-head regime.** When GEPA and SkillOpt also get a validation split and environment access, CASD still leads on 2 of 4.
- **Cost.** CASD costs about **$1.60 per skill**, $6.4 in total across the four benchmarks, versus GEPA $8.4 and SkillOpt $142.5.
- **Skill vs thinking.** The CASD skill on no-think GPT-5.4-mini *exceeds* the thinking mode on ALFWorld (83.3 vs 71.3) and retail. It recovers 78% and 55% of the no-think→think gap on telecom and SSB, at 0.6–0.8k output tokens per episode.
- **Process.** 24 logged distillation runs made 816 tool calls (16–51 per run) and produced 5–8 KB skill files. The 12 CASD skills contain 54 quantitative evidence citations in total. The four GEPA prompts contain none, and the four SkillOpt prompts one.

> [!warning] Newsletter vs paper
> "22× cheaper" is relative to **SkillOpt only** ($6.4 vs $142.5 in total). Against GEPA, CASD is only about **1.3× cheaper** ($6.4 vs $8.4). The +16.6 / +10.9 / +5.3 figures are correct. SkillOpt's ALFWorld cell uses its native scaffold (baseline 58.7), which the paper flags as not directly comparable, so SkillOpt's +5.3 is partly an artifact of that cell.

## What it does not show

- **One target model and one distiller.** Sensitivity to distiller capability is untested; the authors say so.
- **Variance is not like-for-like.** CASD's SD is the spread across three distilled skills, while the baselines' SD is seed variance of one prompt. Test sets are small: about 7 pp binomial SE per seed.
- **"Think" accuracies come from an earlier study** on the same splits, not from re-runs.
- **CASD cannot discover behaviours absent from failure-rich logs.** It needs an existing rollout corpus, which the cost figure treats as free.

## Concepts & entities

- [[prompt-optimization]] · [[skill-optimization]] · [[agent-skills]] · [[skill-md-format]] — prompt optimization as one-shot skill distillation.
- [[gepa]] · [[gepa-agrawal-2025]] · [[skillopt-yang-2026]] — the baselines.
- [[pal]] · [[program-of-thoughts]] — "offload the counting to an interpreter", applied to the optimizer.
- [[alfworld]] · [[tau-bench]] · [[claude-code]] · [[microsoft]].
- Related: [[wikiskill-compiling-agent-experience-tang-2026]] · [[from-memory-to-skills-tang-2026]] (experience → skill compilation). The [[tasteful-agent-taste-bench-pan-2026|Taste-Bench]] paper (also Microsoft) mines trajectory corpora for decision forks.

## References

- [arXiv:2609.26261](https://arxiv.org/abs/2609.26261)
