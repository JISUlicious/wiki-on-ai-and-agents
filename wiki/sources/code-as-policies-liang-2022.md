---
title: "Code as Policies: Language Model Programs for Embodied Control"
type: source
created: 2026-05-25
updated: 2026-05-25
sources:
  - code-as-policies-liang-2022.md
arxiv_id: "2209.07753"
venue: ICRA 2023
authors:
  - Jacky Liang
  - Wenlong Huang
  - Fei Xia
  - Peng Xu
  - Karol Hausman
  - Brian Ichter
  - Pete Florence
  - Andy Zeng
first_author: Jacky Liang
year: 2022
introduces:
  - "[[code-as-policies]]"
tags:
  - 2022
status: complete
importance: high
---

# Code as Policies: Language Model Programs for Embodied Control

[arXiv:2209.07753](https://arxiv.org/abs/2209.07753)

## Summary

Code as Policies (CaP) repurposes code-writing LLMs as robot policy generators: given a natural language command and a few-shot prompt of (comment, code) examples, the LLM emits executable Python that calls perception APIs (e.g., open-vocabulary object detectors) and parameterizes control primitive APIs (pick-and-place, set_velocity, impedance control). The generated programs — termed Language Model Programs (LMPs) — chain classic control flow, reference third-party libraries (NumPy, Shapely) for arithmetic and geometry, and recursively define undefined helper functions through hierarchical code generation. The result is a robot policy that exhibits spatial-geometric reasoning, generalizes to unseen instructions, and assigns precise numeric values (velocities, offsets) to vague qualifiers like "faster" or "a bit to the left" using behavioral commonsense baked into the pretraining corpus.

CaP is the seminal "code for acting" work in embodied settings: it elevates code from a side-channel reasoning trace to the primary interface between an LLM and the physical world. The [[code-as-agent-harness-ning-2026]] survey traces its §2.2 Code for Acting lineage back to this paper, alongside [[program-of-thoughts]] and [[pal-program-aided-language-models]] on the reasoning side. By making Python the policy language, CaP bridges symbolic LLM reasoning to real robot execution — demonstrated across tabletop manipulation, mobile robots, and whole-body control — and seeds the broader [[code-as-harness]] paradigm later generalized to non-embodied agents and to [[vision-language-action-model]] systems.

## Key Points

- **Python as robot policy language.** LLMs emit executable Python whose statements call a small set of perception + control primitive APIs; the program *is* the policy, evaluated at runtime against the live robot state.
- **Few-shot prompting with comment → code examples.** Natural language commands appear as comments, followed by reference policy code; new commands are appended and the LLM autoregressively writes new code that re-composes the APIs.
- **Hierarchical code generation.** When the LLM emits a call to an undefined helper (e.g., `stack_objects`, `is_empty`), CaP recursively prompts the LLM to define that function, enabling decomposition of complex tasks into reusable sub-policies. This recursion also lifts HumanEval performance to 39.8%, a code-benchmark side effect.
- **Perception + control APIs as primitives.** `detect_objects`, `get_obj_pos`, `pick_place`, `set_velocity`, `say` — small, language-namable building blocks the LLM composes; perception is grounded via open-vocabulary detectors (e.g., ViLD, MDETR).
- **Spatial-geometric reasoning via code.** Calls to NumPy / Shapely let LMPs reason over coordinates, bounding boxes, polygons — translating phrases like "a bit to the left of the pyramid" into concrete metric offsets.
- **Reactive and waypoint-based policies.** Generated code can express feedback loops (`while not detect_object("apple"): robot.set_velocity(...)`) as well as open-loop waypoint trajectories and impedance controllers.
- **Behavioral commonsense from pretraining.** Pretraining on billions of lines of code+comments gives LLMs priors over plausible numeric values (a "small" offset ≈ a few centimeters, "faster" ≈ a velocity bump), grounding ambiguous adverbs without task-specific data.
- **Real-world demos across platforms.** Tabletop manipulation, mobile pick-and-place, whole-body control on real robots — not just sim.
- **Q&A and dialogue for free.** Exposing `robot.say(text)` as an API lets the LLM mix verbal responses with motor actions in the same program ("tell me why you stopped" → `robot.say("I saw a banana")`).

## Entities

- [[google]]
- [[google-robotics]]
- [[jacky-liang]]

## Concepts

- [[code-as-policies]]
- [[code-as-harness]]
- [[embodied-agent]]
- [[vision-language-action-model]]

## Notable Quotes

> "Code-writing LLMs can be re-purposed to write robot policy code, given natural language commands. Specifically, policy code can express functions or feedback loops that process perception outputs and parameterize control primitive APIs."

> "Central to our approach is prompting hierarchical code-gen (recursively defining undefined functions), which can write more complex code and also improves state-of-the-art to solve 39.8% of problems on the HumanEval benchmark."

> "By chaining classic logic structures and referencing third-party libraries (e.g., NumPy, Shapely) to perform arithmetic, LLMs used in this way can write robot policies that (i) exhibit spatial-geometric reasoning, (ii) generalize to new instructions, and (iii) prescribe precise values to ambiguous descriptions depending on context."

## References

- [arXiv:2209.07753](https://arxiv.org/abs/2209.07753) — Liang et al., 2022. *Code as Policies: Language Model Programs for Embodied Control*. ICRA 2023.
- Project page: https://code-as-policies.github.io
- Source file: `code-as-policies-liang-2022.md`
