---
title: Context Rot
type: concept
created: 2026-10-06
updated: 2026-10-06
sources:
  - lost-in-the-middle-liu-2023.md
  - effective-context-engineering-anthropic-2025.md
  - gemini-1-5-google-2024.md
status: draft
importance: medium
---

# Context Rot

The degradation of a model's ability to find and use information as its **context grows**. A window that *accepts* N tokens does not *use* all N equally well.

## Evidence line

- **Jul 2023: position effects.** [[lost-in-the-middle-liu-2023|Lost in the Middle]] finds a U-shaped curve:
  - information at the start or end of the context is used well; the middle is used worst;
  - GPT-3.5-Turbo with the answer mid-context did worse than with no documents at all;
  - extended-context variants were no better at using their context.
- **Feb 2024: retrieval of one fact.** [[gemini-1-5-google-2024|Gemini 1.5]] reports 99% [[needle-in-a-haystack]] recall at 1M tokens. This shows that *finding one needle* can be solved at length. It does not show that many facts can be used together.
- **2025: the name.** Anthropic attributes "context rot" to "studies on needle-in-a-haystack style benchmarking": recall falls as tokens increase. Other wiki pages cite Chroma's 2025 report (Hong et al.) for the term. The wiki has not ingested that report.
- **2026: compaction.** Agent-memory papers find information lost through summarization. Here the loss is *monotonic*, unlike the U-shape (see [[lost-in-compaction-side-constraint-loss-wang-2026]]).

## Explanations offered

From [[effective-context-engineering-anthropic-2025]], not experimentally separated:
- n² attention spreads thin;
- training data skews toward short sequences;
- positional-encoding interpolation blurs position.

The result is "a performance gradient rather than a hard cliff."

## Practical responses

These are the [[context-engineering]] toolkit:
- put critical instructions and the query at the edges (query-aware placement);
- retrieve fewer, better documents;
- compact the context, keep notes, and isolate work in [[subagents]].

## Related

[[long-context-llm]] · [[needle-in-a-haystack]] · [[context-engineering]] · [[retrieval-augmented-generation]] · [[memory-management]]

## References

- [[lost-in-the-middle-liu-2023]]
- [[effective-context-engineering-anthropic-2025]]
- [[gemini-1-5-google-2024]]
