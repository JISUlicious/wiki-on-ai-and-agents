---
title: "Does the hard-to-read AI prose paper give prompts to avoid it, and do the patterns appear in Korean and other languages?"
type: query
created: 2026-11-11
updated: 2026-11-11
sources:
  - why-is-ai-generated-language-so-hard-to-understand-clark-2026.md
status: complete
importance: medium
---

# Does the hard-to-read AI prose paper give prompts to avoid it, and do the patterns appear in other languages?

**Questions (2026-11-10):**
1. Does arXiv:2610.29162 provide guides or example prompts to avoid AI-style generated text?
2. Do those patterns also appear in Korean or other languages?

The paper is [[why-is-ai-generated-language-so-hard-to-understand-clark-2026]] (Clark, Martin & Mahowald). Parts marked **(inference)** are not from any source.

## 1. No guide and no prompts

- **Its only prompts collect data.** The prompts in the appendix produce the corpus, for example a system prompt that makes a model play a casual human user in the chat setting. None asks a model to write more clearly.
- **Its only advice is for model trainers.** The conclusion says future work may consider aligning outputs "with the needs of human comprehenders." It points to [[claude-opus-5-5|Claude Opus 5.5]] scoring lower than Opus 5 on 19 of 21 difficulty-linked measures as evidence that training can fix this.
- **No mitigation was tested.** The proposed cause is post-training pressure for short, correct answers ([[outcome-based-reward]], a [[grpo|GRPO]] length penalty). So "write clearly" in a prompt may only partly counter it **(inference)**.

### A checklist prompt built from the paper's measures (inference, untested)

Each rule targets one feature the paper measured:

```
Write for a human reading once, start to finish.
- Give actions to people or animate agents. Don't say "the function swallows the error";
  say "the function catches the error and ignores it".
- Keep "that"/"which" in relative and complement clauses ("the file that the script reads").
- No stacks of 3+ nouns ("reader-patience budget"); unpack them with "of"/"for".
- Don't turn nouns into verbs ("footnote it", "calendar it").
- Prefer common phrasings over unusual word pairings.
- Keep subject and verb close; split long sentences instead of nesting.
- Don't drop small words to save space; clarity beats brevity.
```

| Rule | Paper's measure |
|---|---|
| Animate agents | Inanimate subject + animate verb |
| Keep "that" | Object relative clauses with the relativizer left out |
| No noun stacks | Stacked noun compounds (3+ nouns) |
| No noun-to-verb | Ad-hoc denominal verbs |
| Common phrasing | Rare bigrams and trigram perplexity |
| Subject near verb | Dependency length |

## 2. Other languages: nobody has measured the same features

The Clark paper is **English only** (stated in its limitations). Japanese appears only as the pivot of its translation round trip (NLLB-200), not as a language it studied.

Studies in other languages (web search, 2026-11-10; **not yet ingested**) measure different features:
- **Korean, KatFishNet** (Park, Kim, Kim & Han, ACL 2025, [arXiv:2503.00032](https://arxiv.org/abs/2503.00032)):
  - **Spacing:** models space words with a rigid consistency human writers don't show.
  - **Variety of word types:** less variety (more nouns, fewer verbs and adjectives).
  - **Commas:** more of them, placed as in English, though Korean uses fewer commas than English.

  The user's `humanizer` skill is built on this paper. A third-party summary notes the detector may degrade as new models appear.
- **Chinese and French, "Do LLMs have an English accent?"** ([arXiv:2410.15956](https://arxiv.org/abs/2410.15956)): in one measure, Llama-3.1's Chinese output was structurally closer to human English than to human Chinese. Every model, in every language tested, diverged from native syntax. The authors blame English-centric training data, including text translated from English.
- **German news text** (LM4DH 2025): some models come close to human syntax on some measures, but only partly and depending on context. The authors also blame interference from English.

### How the English patterns carry over to Korean (inference)

| Clark's English pattern | Korean counterpart |
|---|---|
| Inanimate subject + action verb ("this asymmetry bites") | Lifeless subjects ("이 함수는 에러를 삼킨다") are a classic marker of translation-style Korean (번역투), so likely transfers |
| Stacked nouns | Long noun chains ("데이터 처리 효율성 개선 방안"), the same as KatFishNet's "too many nouns, too few verbs" |
| Rare word pairings | Plausible, but no Korean study measures it |
| Dropping "that" | No direct counterpart: Korean relative clauses have no optional word to drop |

### Two explanations

| | Proposed cause | Evidence |
|---|---|---|
| Clark et al. (English) | Post-training pressure to compress (short, verifiable answers) | Olmo base → instruct → Think trend; GRPO length-penalty experiment |
| Non-English studies | An "English accent" from mostly English training data | Structural similarity to English; translationese |

For Korean, both probably apply at once. Nobody has separated them. A Korean version of Clark's study (round-trip baseline plus Olmo base, instruct and reasoning versions) would test this **(inference)**.

## Follow-ups

- Ingest KatFishNet and "Do LLMs have an English accent?" as sources and link them to the Clark page.
- A concept page for AI-generated text intelligibility could join Clark, KatFishNet and the accent paper.

## References

- [[why-is-ai-generated-language-so-hard-to-understand-clark-2026]]
- [[outcome-based-reward]] · [[grpo]] · [[claude-opus-5-5]]
