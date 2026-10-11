---
title: "Dual process theory and Thinking, Fast and Slow (Wikipedia)"
type: source
created: 2026-10-10
updated: 2026-10-10
sources:
  - dual-process-theory-wikipedia.md
  - thinking-fast-and-slow-wikipedia.md
introduces:
  - "[[dual-process-theory]]"
year: 2026
venue: Wikipedia (fetched 2026-10-10)
url: https://en.wikipedia.org/wiki/Dual_process_theory
tags:
  - encyclopedia
status: complete
importance: medium
---

# Dual process theory and *Thinking, Fast and Slow* (Wikipedia)

**Sources**: `sources/dual-process-theory-wikipedia.md` ([Dual process theory](https://en.wikipedia.org/wiki/Dual_process_theory)) and `sources/thinking-fast-and-slow-wikipedia.md` ([Thinking, Fast and Slow](https://en.wikipedia.org/wiki/Thinking,_Fast_and_Slow)), both fetched 2026-10-10.

**Why secondary sources:** Kahneman's book is not freely available, and the primary psychology papers (Wason & Evans 1974; Evans 1984, 2003, 2008; Sloman 1996; Stanovich & West 2000) are cited here through these articles.

## Dual process theory

- **The idea:** thinking arises in two different ways, or from two processes. It is old: Spinoza distinguished passions from reason, and William James distinguished associative from "true" reasoning.
- **Modern forms:**
  - **Wason & Evans (1974)**;
  - Evans' **heuristic vs analytic** processes (1984);
  - **Sloman (1996)**: associative vs rule-based reasoning;
  - social-psychology variants such as the elaboration likelihood model.
- **The names "System 1" and "System 2"** were "coined by Keith **Stanovich and Richard West**" (2000), as the neutral replacement for "implicit" and "explicit". Kahneman popularized them; he did not coin them.
- **Evidence:** belief-bias studies, where valid logic is pitted against believable conclusions. Loading working memory hurts System 2-dependent answers (De Neys). fMRI and NIRS studies show prefrontal conflict resolution.
- **Critiques** (Evans' five fallacies, and Osman):
  1. Not all dual-process theories are the same.
  2. There are more than two underlying systems.
  3. **"System 1 processes are responsible for cognitive biases; System 2 processes are responsible for normatively correct responding"** is a fallacy: both can be right, and both can be biased.
  4. System 1 is not uniquely contextual; System 2 is influenced by beliefs too.
  5. **"Fast processing indicates the use of System 1"** is a fallacy: experience can make System 2 fast.
  - Moshman proposes **four** types instead of two (implicit/explicit × heuristic/rule-based).
- **Applied to LLMs:** the article now has a section on a taxonomy of "fast thinking" (direct response) vs "slow thinking" (chain-of-thought, tool calls such as RAG, intermediate verification), extended to "tool-augmented thinking".

## *Thinking, Fast and Slow* (Kahneman, 2011)

- **System 1:** "fast, instinctive and emotional"; automatic, frequent, unconscious. Examples: 2+2, completing "war and…", reading a billboard, driving on an empty road, "a good chess move (if one is a chess master)".
- **System 2:** "slower, more deliberative, and more logical"; effortful, infrequent. Examples: 17 × 24, parking in a tight space, checking a complex argument, counting letters.
- **Heuristics and biases (System 1's failure modes):**
  - anchoring;
  - availability;
  - **substitution**: answering an easier question instead, as in the Linda problem;
  - optimism bias;
  - WYSIATI ("what you see is all there is").
  - The book also covers prospect theory and framing.
- **Reception:** a bestseller and award winner. Its **replication** record has been heavily criticized: many cited priming findings failed to replicate, and one analysis says much of the book rests on "scientific literature with shaky foundations".

## Why it matters for the wiki

This is the source of the metaphor behind TypeSafe's "[[system-one-model|System One model]]" name and behind the LLM "System 1 → System 2" reasoning literature. Two of the critiques bear directly on how LLM work uses the metaphor:
- "fast means System 1" is a fallacy;
- "System 1 = biased, System 2 = correct" is a fallacy.

See [[dual-process-theory]] and [[system-one-models-and-dual-process-thinking]].

## Concepts & entities

[[dual-process-theory]] · [[system-one-model]] · [[chain-of-thought-prompting]] · [[large-reasoning-model]]
