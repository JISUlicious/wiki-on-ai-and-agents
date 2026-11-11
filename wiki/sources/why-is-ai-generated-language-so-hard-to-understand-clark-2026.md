---
title: "Why is AI-generated language so hard to understand?"
type: source
created: 2026-11-10
updated: 2026-11-11
sources:
  - why-is-ai-generated-language-so-hard-to-understand-clark-2026.md
arxiv_id: "2610.29162"
venue: arXiv preprint (cs.CL)
authors:
  - Thomas Hikaru Clark
  - Matias B. Martin
  - Kyle Mahowald
year: 2026
status: complete
importance: high
tags:
  - 2026
  - paper
---

# Why is AI-generated language so hard to understand?

**Source:** Thomas Hikaru Clark (NYU), Matias B. Martin and Kyle Mahowald (UT Austin) — [arXiv:2610.29162](https://arxiv.org/abs/2610.29162), cs.CL, v1 21 Oct 2026. Preprint; no venue given.

## Summary

A psycholinguistics paper that takes the common complaint that recent LLM prose has become harder to read and turns it into two testable claims. **H1:** frontier and near-frontier models overuse constructions that the human sentence-processing literature already links to comprehension difficulty. **H2:** a pressure for *information density* during post-training (rewarding correct answers in fewer tokens, with no reward for clarity) pushes model language away from human-like choices.

The argument rests on a contrast. Human language is shaped by communicative success under human limits (lossy working memory, incremental parsing), which is why human text minimises dependency length and spreads information evenly (uniform information density, UID). A model trained only on human text should inherit that profile. But modern models are heavily post-trained on math, code and reasoning with verifiable rewards ([[outcome-based-reward]]), where nobody checks whether the prose is easy to follow and long contexts are expensive. The authors argue this lets model language drift toward a compressed register that is cheap in tokens and costly for human readers.

Both hypotheses get support. Experiment 1 finds the difficult features overrepresented in a new multi-domain corpus of model output. Experiment 2 shows that a length penalty alone, applied during [[grpo|GRPO]] fine-tuning, is enough to make a small model categorically prefer the shorter member of syntactic alternations, including on constructions it never trained on.

## Experiment 1: what models write

- **Corpus.** 14 models: 9 frontier or near-frontier (Claude Sonnet 5, Opus 5, Opus 5.5 and Fable 5; GPT-5.6 Luna, Terra and Sol; [[glm-5|GLM-5]]; Kimi K2.5) and 5 small open models (Llama 3.1 8B base and instruct; Olmo 3 7B base, instruct and Think). Three settings: 120 multi-turn chat conversations (10 seeds, up to 12 turns), 45 agentic coding sessions (5 public GitHub repos, find-and-fix bugs in a sandbox), and 980 single-turn PERSUADE 2.0 essays. Data was collected in September 2026.
- **Features measured** (from spaCy dependency parses, with an automated classifier vetting parser candidates for two features):
  - **Inanimate subjects with animate verbs**, e.g. a function that "swallows" an error or a pattern that "bites". These break a selectional restriction; readers use animacy as a parsing cue. The paraphrase usually needs more words ("bites" ≈ "causes a problem").
  - **Object relative clauses (ORCs)**, split into with and without the optional relativizer *that/which*. ORCs are harder than subject relatives; the reduced form saves a word and compresses a longer passive.
  - **Dependency length**, standardised across sentence lengths. Long dependencies tax working memory and humans avoid them.
  - **Rare n-grams**: the rate of bigrams unattested in a genre-matched reference corpus, and trigram perplexity. Dropping predictable words raises density and reading cost.
  - **Stacked noun compounds** (three or more nouns, e.g. "reader-patience budget") and **ad-hoc denominal verbs** ("footnote it", "calendar it").
- **Two baselines.** (1) Human text matched by genre: OpenSubtitles for chat, StackExchange for code, top-scored PERSUADE student essays, plus Wikipedia. (2) **Round-trip paraphrases**: each model sentence translated English → Japanese → English with NLLB-200 (3.3B). This keeps the meaning and swaps the wording for frequent forms. A feature that drops after the round trip is one the model *chose* over an available equivalent.
- **Against human text:** all nine frontier and near-frontier models use more inanimate-subject/animate-verb pairs and more low-frequency n-grams than the human baselines. Most also show longer dependencies, more stacked nouns and more reduced ORCs in conversation.
- **Against round-trip paraphrases:** nearly all frontier models overuse rare n-grams, denominal verbs, inanimate subjects and reduced ORCs, and *underuse* ORCs that keep the relativizer. So the model consistently skips the optional word. Stacked nouns and dependency length follow the same pattern in chat and code but not in essays.
- **Setting matters.** Effects are smaller for single-prompt essays than for long chat and coding sessions. The authors guess that long back-and-forth contexts add pressure to compress.
- **Training-stage trend (the key circumstantial evidence for H2).** The Llama models and Olmo base and instruct change least under round-trip translation and sit closest to the student essays. Within Olmo the effect grows **base → instruct → Think**, and the reasoning model looks like the frontier models.
- **Model-family variation.** [[claude-opus-5-5|Claude Opus 5.5]] scores lower than its predecessor Opus 5 on 19 of 21 difficulty-linked comparisons (8 with non-overlapping 95% intervals). The authors read this as evidence that training *can* reward clearer communication.

## Experiment 2: a length penalty is enough (proof of concept)

- **Setup.** [[qwen-2-5|Qwen2.5-1.5B-Instruct]] fine-tuned with [[grpo|GRPO]] (Dr. GRPO loss, [[lora|LoRA]]) on a task that turns JSON-like event descriptions into one English sentence. Every item can be expressed with either member of a syntactic alternation, one shorter by a known number of tokens: dative, object relative (with or without *that*), complement clause (with or without *that*), and a verb-particle control where both orders have equal length. Both variants are correct; only length differs. 24 runs cross a length penalty λ with KL weight β ∈ {0, 0.04}, three seeds each.
- **Result.** The share of short variants rises with λ, with or without KL. Without KL, every seed converges for any λ > 0 to a near-categorical preference (over 98% short for both trained *that* alternations). With λ = 0, seeds range from 0% to 100% with no stable preference. Human speakers instead choose by context, for example keeping *that* when the next material is less predictable.
- **Transfer.** The preference spreads to untrained *that* constructions (*sure (that)*, *so (that)*), which is near transfer. Far transfer to unrelated alternations is weak: only the genitive (*X's Y* vs *Y of X*) shifted, and only without KL. Passive and purpose-clause alternations did not move, and the quantifier one was already at ceiling.
- **Interpretation.** The model was never told to compress and never shown the alternations. The preference emerged from the length penalty. The authors argue that verifiable-reward training separates correctness from human-like communication. RLHF, by contrast, is known to reward *longer* outputs, so they tie compression to the shift from [[rlhf|RLHF]] to verifiable rewards, not to RLHF itself.

## Limitations

- **No human comprehension experiment.** The paper measures features that the literature *associates* with processing difficulty. It never asks people to read model text and records no reading times, comprehension scores or ratings. "Hard to understand" is inferred, not shown directly.
- English only, a subset of commercial models, few seeds (10 chat topics, 5 repos).
- H2 is causal only for a 1.5B model on a toy task. Frontier post-training could not be observed, so Experiment 1's support is circumstantial (proxies like the Olmo stage trend).
- Round-trip translation also normalises toward frequent forms, so it is a conservative but imperfect paraphrase control.

## Why it matters for this wiki

It gives a mechanistic story, with a small causal test, for the "LLM house style" that readers notice. It also links training choices ([[reasoning-effort]] and token budgets, [[outcome-based-reward|RLVR]]) to a cost that benchmark scores never see: legibility for humans and monitorability. A useful pairing is with work on [[chain-of-thought]] faithfulness and [[latent-reasoning]]: if reward pressure compresses visible prose, it plausibly compresses visible reasoning too. The paper hints at this when it mentions monitorability, but it does not test it.

## Concepts & entities

- Training: [[grpo]] · [[rlhf]] · [[outcome-based-reward]] · [[reinforcement-learning]] · [[lora]] · [[large-reasoning-model]] · [[reasoning-effort]]
- Models: [[claude-opus-5-5]] · [[claude]] · [[glm-5]] · [[llama-3]] · [[qwen-2-5]] · [[gpt-family]]
- Related ideas: [[chain-of-thought]] · [[latent-reasoning]] · [[distributional-semantics]]

See also: [[avoiding-hard-to-read-ai-prose-and-other-languages]]. It covers the absence of prompts or guides in the paper, a checklist prompt derived from its measures, and Korean and other-language evidence.

## References

- [arXiv:2610.29162](https://arxiv.org/abs/2610.29162)
