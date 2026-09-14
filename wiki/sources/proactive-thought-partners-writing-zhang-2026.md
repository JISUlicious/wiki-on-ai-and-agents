---
title: "Designing Proactive Thought Partners for Writing"
type: source
created: 2026-09-15
updated: 2026-09-15
sources:
  - proactive-thought-partners-writing-zhang-2026.md
arxiv_id: "2609.01588"
authors:
  - Chao Zhang
  - Abe Davis
  - Chih-Wei Chen
  - Chin-Chia Hsu
year: 2026
status: complete
importance: low
tags:
  - 2026
  - paper
---

# Designing Proactive Thought Partners for Writing

**Chao Zhang, Abe Davis, Chih-Wei Chen, Chin-Chia Hsu** (Zhang and Davis: Cornell; Chen and Hsu: [[deepmind|Google DeepMind]]; Zhang did the work as a DeepMind intern). [arXiv:2609.01588](https://arxiv.org/abs/2609.01588), cs.HC, v1 1 Sep 2026. No venue stated.

> [!important] What this paper is, in one paragraph
> This is an HCI **technology-probe** study, not an evaluation. It targets the gap between proactive tools that only work on text (autocomplete, grammar) and higher-level cognitive help (ideation, argument, reflection) that users must invoke themselves. Writers build **"thought partners"**: user-defined [[llm-agent|agents]], each with a role, one or more **event triggers** (long pause, sentence end, text selection), and a natural-language **contextual heuristic** saying when to step in. On a trigger, a Gemini-2.5-flash-lite decision engine (under 1 s) reads the goal, the draft, and 15 s of keystrokes, and activates at most two partners. Gemini-2.5-flash then writes each suggestion in about 8 s, as an **acknowledgement plus a question**. The suggestion appears as a small, fading tag in a side panel. **16 participants** used it for **one week**.

## Key results

- **Usage.** 66 sessions (4.13 per participant), averaging 390 words and 17.1 minutes each. SUS (derived from UMUX-LITE) was 82.81, "Excellent". Daily satisfaction averaged 8.16/10 and rose significantly with session index (b = 0.33, p = .036). Helpfulness averaged 8.11/10 and timeliness 7.85/10.
- **Partners.** Participants created 54 partners, 3.38 per person; 35 were used in only one session. 43 of 54 (79.6%) targeted higher-level cognition: information seeking 14, argument development 12, critical reflection 10, ideation 7. The rest targeted local text. Participants configured partners by **prospective planning**, anticipating their own blockers before writing.
- **Engagement (1,100 interventions).** **58.3% were ignored** (641), and 64.4% of pause-triggered ones were ignored, because pauses are ambiguous. 19.6% were opened for inspiration, and 22.2% were **executed**, with 84.4% (206/244) of the generated text then accepted. Text selection led to execution 54.5% of the time. Participants used suggestions in two ways: **generative** (new connections) and **regulatory** (self-monitoring: "veering off track", one-sided arguments). They avoided executing suggestions from reflection-type partners.
- **What participants valued.** "Push" rather than "pull": they often don't know what to ask a chatbot. Also valued: **peripheral, fading** tags that are easy to ignore; acknowledgements that signal the partner is "on the same wavelength"; and **question** framing that preserves control. The exception is "unknown unknowns", where a question alone doesn't help.

## Design implications (9, across 4 dimensions)

- **Customization.** Treat it as iterative prospective planning: preview and simulate partners, and suggest refinements from usage traces.
- **Timing.** Treat events as *candidate* moments. Add a contextual-inference layer between event detection and intervention, using draft-state, writing-activity and anticipated-need cues.
- **Engagement.** Make it graduated: ignoring should cost nothing, and editing the document stays a deliberate user action, separate from the system taking initiative.
- **Representation.** Keep suggestions lightweight and non-directive.

## What it does not show

The sample is 16 AI-savvy volunteers (all use AI daily or weekly) from one large software company's mailing list; 11 of the 16 were non-native English speakers. They were told to avoid confidential work content, which the authors admit probably excluded their highest-stakes writing. There is no baseline condition, no reactive-assistant comparison, and no measure of writing quality or long-term agency. The authors say outright that the study establishes no causal effects. Results are tied to one Gemini-based implementation. There is one internal inconsistency: the text says pause-triggered interventions led to execution 20.61% of the time, but Table 3 gives 80/660 = **12.12%**.

## Related

- [[llm-agent]]: partners are lightweight, user-authored agents. An event trigger plus an LLM-checked condition works like the hook pattern in [[hooks-in-modern-agents]] (this wiki's analogy, not the paper's).
- [[how-ai-agents-reshape-knowledge-work-yang-2026]]: the assistant-versus-agent autonomy shift in production data. This paper studies the same shift from pull to push, inside one writing tool.
- [[deepmind]] · [[google]]: affiliation and model provider (Gemini 2.5 Flash and Flash-Lite; no wiki page). The wiki has no human–AI interaction or writing-assistant page yet.

## References

- [arXiv:2609.01588](https://arxiv.org/abs/2609.01588) (v1, 1 Sep 2026).
- Flower & Hayes 1981 (cognitive process theory of writing); Horvitz 1999 (mixed-initiative interfaces); Hutchinson et al. 2003 (technology probes).
- Yin et al. 2026 (Wizard-of-Oz proactive writing AI); Pu et al. 2025, Chen et al. 2025a (proactive programming assistants); Lehmann et al. 2026, Benharrak et al. 2024 (customizable writing agents); Maier et al. 2026 (question-style suggestions).
- UMUX-LITE: Lewis et al. 2013; SUS adjective scale: Bangor et al. 2008, 2009; thematic analysis: Braun & Clarke 2013.
