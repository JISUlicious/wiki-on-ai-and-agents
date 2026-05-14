---
title: "Constitutional AI: Harmlessness from AI Feedback"
type: source
created: 2026-05-14
updated: 2026-05-14
sources:
  - bai-constitutional-ai-2022.md
arxiv_id: "2212.08073"
venue: arxiv preprint
authors:
  - Yuntao Bai
  - Saurav Kadavath
  - Sandipan Kundu
  - Amanda Askell
  - "[[chris-olah]]"
  - "[[dario-amodei]]"
  - "[[sam-mccandlish]]"
  - "[[tom-brown]]"
  - "[[jared-kaplan]]"
first_author: Yuntao Bai
introduces:
  - "[[constitutional-ai]]"
  - "[[rlaif]]"
year: 2022
tags:
  - 2022
status: complete
importance: high
---

# Constitutional AI: Harmlessness from AI Feedback

**Source**: `sources/bai-constitutional-ai-2022.md` (extracted from `bai-constitutional-ai-2022.pdf`)
**arXiv**: [2212.08073](https://arxiv.org/abs/2212.08073)
**Venue**: arxiv preprint (Dec 2022)
**Authors**: Yuntao Bai (first), Saurav Kadavath, Sandipan Kundu, Amanda Askell, ..., [[chris-olah]], [[dario-amodei]], [[sam-mccandlish]], [[tom-brown]], [[jared-kaplan]] ([[anthropic]])

## Summary

Constitutional AI is [[anthropic|Anthropic]]'s approach to training a harmless, helpful AI assistant **without using human labels of harm**. The only human input is a written set of principles (a "constitution"). The AI generates its own training signal from those principles, in two phases:

1. **Supervised (Constitutional) phase**:
   - Sample responses from an initial helpful model on a set of (possibly adversarial) prompts.
   - For each response, prompt the model to **critique itself** against a randomly-selected principle from the constitution ("Identify how this response is harmful, deceptive, or unhelpful").
   - Prompt the model to **revise** its original response based on the critique.
   - Fine-tune the original model on the revised responses (Critique-Revision SFT).
2. **RL from AI Feedback (RLAIF) phase**:
   - Sample pairs of responses from the SFT-improved model on adversarial prompts.
   - Prompt a separate evaluator LM to **choose which response is more aligned with the constitution** (the "AI feedback").
   - Train a preference / reward model on these AI-generated comparisons.
   - Run [[rlhf|RLHF]]-style RL against the AI preference model.

The result is a harmless-but-non-evasive assistant: rather than refusing harmful queries opaquely, it **engages and explains its objections**. This is the alignment lineage behind [[claude|Claude]] from 2023 onward.

The deeper claim is **scalable oversight**: as AI systems become more capable, we'll want to use *them* to help supervise themselves. Constitutional AI is a concrete demonstration that AI feedback can substitute for human labels on at least the harmlessness dimension, with the constitution providing the meta-level human direction.

The technique uses [[chain-of-thought-prompting|chain-of-thought]] style reasoning in both the critique-revision step and the AI feedback step, which improves both human-judged performance and the *transparency* of the model's decisions — you can read why the AI thought response A was preferable to response B.

## Key Points

- **The constitution**: a written list of ~16 principles like "Choose the response that is more ethical" or "Choose the response that's least likely to provide misleading information". Drawn from sources like the UN Universal Declaration of Human Rights, Apple's Terms of Service, and original principles.
- **Critique-Revision SFT**:
  - Sample → critique-against-principle → revise → fine-tune on revised.
  - No human labels of harmful outputs — the AI does the critiquing.
- **RLAIF**:
  - Pairwise comparisons by an AI judge based on the constitution.
  - Train preference model on AI preferences.
  - RL against the preference model. Standard [[rlhf]] architecture but with AI feedback substituted for human.
- **Non-evasive**: the trained model engages with harmful queries by explaining objections rather than refusing opaquely.
- **CoT in both phases** improves both performance and transparency.
- **Practical alignment win**: dramatically less human labeling. The constitution + AI feedback can substitute for tens of thousands of human-labeled comparisons.
- **Deeper claim — scalable oversight**: AI feedback as a building block for aligning systems whose capabilities exceed the typical human labeler's.

## Entities Mentioned

- Yuntao Bai — first author (Anthropic; later co-author of Claude papers)
- [[chris-olah]], [[dario-amodei]], [[jared-kaplan]], [[sam-mccandlish]], [[tom-brown]] — Anthropic founders
- Amanda Askell — Anthropic principal researcher; lead on Claude's behavioral character
- [[anthropic]] — origin org

## Concepts Discussed

- [[constitutional-ai]] — the method
- [[rlaif]] — RL from AI Feedback
- [[rlhf]] — what RLAIF substitutes for
- [[chain-of-thought-prompting]] — used in both critique-revision and AI feedback
- [[preference-modeling]] — preference model trained on AI-generated preferences

## Notable Quotes

> "As AI systems become more capable, we would like to enlist their help to supervise other AIs. We experiment with methods for training a harmless AI assistant through self-improvement, without any human labels identifying harmful outputs. The only human oversight is provided through a list of rules or principles, and so we refer to the method as 'Constitutional AI'." — Abstract

> "These methods make it possible to control AI behavior more precisely and with far fewer human labels." — Abstract

## References

_Original source: `sources/bai-constitutional-ai-2022.md`_
