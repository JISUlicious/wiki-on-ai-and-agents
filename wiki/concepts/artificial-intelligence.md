---
title: Artificial Intelligence
type: concept
created: 2026-10-06
updated: 2026-10-06
sources:
  - ai-ml-dl-difference-nvidia-2016.md
  - ai-ml-dl-nn-ibm.md
  - intelligent-agent-wikipedia.md
status: draft
importance: high
---

# Artificial Intelligence

The outermost circle of the field map: **AI ⊃ [[machine-learning]] ⊃ [[deep-learning]] ⊃ [[large-language-model|LLMs]]**.

- The term was adopted when "a handful of computer scientists rallied around the term at the Dartmouth Conferences in 1956" ([[ai-ml-dl-difference-nvidia-2016|NVIDIA 2016]]).
- IBM: "AI is the overarching system" ([[ai-vs-ml-vs-dl-vs-nn-ibm|IBM]]).
- The classical textbook definition is agent-centred. Russell & Norvig treat AI as "the study of agents that receive percepts from an environment and perform actions" ([[intelligent-agent]]).

## The field map

```
Artificial intelligence          (1956 —)  any technique that makes machines act intelligently,
│                                           including hand-coded rules, search and logic
└─ Machine learning              (learned from data rather than hand-coded)
   └─ Deep learning              (many-layer neural networks; the boom from ~2012)
      └─ Large language models   (Transformer LMs trained on text at scale; GPT-3 2020 →)
         └─ LLM agents           (LLMs in a perceive–act loop with tools: ReAct 2022 →)
```

The bottom two levels are the wiki's own extension. The 2016 and IBM sources stop at deep learning and neural networks.

## Distinctions the sources draw

- **Narrow vs general AI.** Narrow AI does specific tasks (image classification, face recognition) as well as or better than humans. General AI, the human-like kind, remained out of reach as of 2016.
- **Boom and bust.** AI was "heralded… and tossed on technology's trash heap" for decades until about 2012. The boom then came from GPUs, cheap storage and big data (NVIDIA, a vendor with a stake in that story).
- **Not all AI is ML.** Symbolic AI (search, logic, expert systems) and classical [[planning]] are AI without learning. LLM agents sometimes call them as tools, for example LLM+P handing plans to a PDDL planner, cited in [[llm-powered-autonomous-agents-weng-2023]].

## Related

[[machine-learning]] · [[deep-learning]] · [[large-language-model]] · [[intelligent-agent]] · [[llm-agent]]

## References

- [[ai-ml-dl-difference-nvidia-2016]]
- [[ai-vs-ml-vs-dl-vs-nn-ibm]]
- [[intelligent-agent-wikipedia]]
