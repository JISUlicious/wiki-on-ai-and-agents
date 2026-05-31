---
title: "ExpeL: LLM Agents Are Experiential Learners"
type: source
created: 2026-05-29
updated: 2026-05-29
sources:
  - expel-zhao-2023.md
arxiv_id: "2308.10144"
venue: AAAI 2024
authors:
  - Andrew Zhao
  - Daniel Huang
  - Quentin Xu
  - Matthieu Lin
  - Yong-Jin Liu
  - Gao Huang
first_author: Andrew Zhao
year: 2023
introduces:
  - "[[experiential-learning]]"
tags:
  - 2023
status: complete
importance: medium
---

# ExpeL: LLM Agents Are Experiential Learners

[arXiv:2308.10144](https://arxiv.org/abs/2308.10144) · AAAI 2024 · [project page](https://andrewzh112.github.io/expel) · [code](https://github.com/LeapLabTHU/ExpeL)

## Summary

ExpeL (Experiential Learning) is a gradient-free LLM agent that learns from its own trial-and-error experience across a collection of training tasks, then applies that learning to unseen tasks at inference — **without any parameter updates**. During a training phase the agent gathers success and failure trajectories using [[reflexion]] over a [[react]] base policy, storing them in an experience pool. From this pool it does two things: (1) it **distills cross-task insights as natural-language rules** (a growing list of "good practices" and failure patterns, edited via ADD/EDIT/UPVOTE/DOWNVOTE operators), and (2) it builds a [[faiss]] vector store of successful trajectories for **task-similarity retrieval**. At evaluation, each task prompt is augmented with the full insight list plus the top-k retrieved trajectories as few-shot demonstrations, and solved in a single attempt — like a student who studies practice problems, derives insights, then sits the exam once. Because it touches only the prompt, ExpeL is compatible with closed-source API models like GPT-4 and Claude.

This paper is the **natural-language counterpoint to code-based skill libraries** such as [[voyager]]. Both accumulate reusable experience across tasks, but the "skill" representation differs fundamentally: in Voyager a skill is an *executable code module* added to a programmatic [[skill-library]] and re-invoked verbatim; in ExpeL a "skill" is a *distilled experiential insight* expressed in natural language plus a retrievable past trajectory — there is no executable code and nothing is run. ExpeL thus stakes out one pole of the "code skills vs. distilled experience" axis for [[agent-skills]]: human-readable, inspectable, editable text rules learned off-policy from experience, managed entirely through prompt construction rather than a parameter update or a code repository. This makes it a load-bearing reference point for the wiki's [[memory-management]] and [[skill-library]] discussions.

## Key Points

- **Cross-task natural-language insight extraction.** The agent iteratively builds a list of insights `ι̂` by comparing failure/success pairs for the same task (to surface concrete shortcomings) and by mining sets of `L` successes across *different* tasks (to surface common good practices). An LLM (`gpt-4-0613` by default) applies four operators — ADD, EDIT, UPVOTE, DOWNVOTE — with an importance counter that removes an insight when its count hits zero, robustifying against misleading/suboptimal trajectories.
- **Trajectory retrieval as episodic recall.** Successful trajectories live in a [[faiss]] vector store, embedded with `all-mpnet-base-v2`; a kNN retriever pulls the top-k by maximum-inner-product task similarity to serve as dynamic few-shot demonstrations. This is the [[episodic-memory-llm]] half of ExpeL, complementing the abstracted-insight half.
- **Gradient-free, off-policy, prompt-only.** No weights change. The framing is explicitly off-policy learning (Watkins & Dayan style) over an experience pool (Lin 1992): the agent learns from a behavior policy's experiences. Strengths claimed: interpretability (insights/trajectories are inspectable, editable, and removable text — unlike a fine-tuned model), low data/compute cost, model-agnosticism, and free gains as base foundation models improve.
- **Inter-task vs. intra-task learning — the Reflexion contrast.** [[reflexion]] does *intra-task* self-improvement via repeated retries on the *same* task; ExpeL does *inter-task* learning by accumulating and abstracting experience across *many* tasks, then solving each test task in **one shot** with no retries. Notably, ExpeL uses Reflexion internally during the training/experience-gathering phase to manufacture richer success/failure pairs.
- **Both learning modes are synergistic (ablation).** Retrieval-only and insights-only each underperform the full agent; their relative importance is domain-dependent (insights dominate on HotpotQA's knowledge tasks; retrieval matters more for ALFWorld's embodied execution). Learned insights beat hand-crafted ones; adding raw reflections into insight extraction *hurts* (hallucination noise); a stronger insight-extraction LLM (`gpt-4` > `gpt-3.5-turbo`) helps; task-similarity retrieval beats reason-similarity and random retrieval.
- **Results.** Evaluated on four text benchmarks — HotpotQA, FEVER, ALFWorld, WebShop — with four-fold validation. ExpeL consistently beats Act and ReAct baselines, and in a single attempt approaches/matches Reflexion despite Reflexion's multiple retries (HotpotQA 40% vs. Reflexion R3 39%; ALFWorld 54% vs. Reflexion R3 59%). Combining ExpeL with Reflexion is synergistic on ALFWorld (54.5% → 64.2% across rounds).
- **Transfer learning.** Insights distilled from a source distribution (HotpotQA) "fine-tuned" with a few target demonstrations transfer with positive forward transfer to a target distribution (FEVER): ExpeL Transfer reaches 70% SR vs. 63% ReAct / 58% Act; transfer with target task demos beats transfer without.
- **Emergent behaviors.** Manual trajectory inspection surfaced unprompted abilities: hypothesis formulation / constraint adaptation (proposing a best-guess answer instead of "Unknown"), world-model belief updates (revising priors on where a pan is likely to be in ALFWorld), and self-correction (putting back a wrongly-taken object and resuming).
- **Experiential-memory lineage.** ExpeL sits in the [[self-improving-agent]] lineage between Reflexion's single-task reflection and Voyager's code-skill accumulation, and anticipates later memory-centric agents that separate a write-time consolidation phase from a read-time retrieval phase.

## Entities

- [[gao-huang]] — corresponding author; Tsinghua University (LeapLab).
- [[tsinghua-university]] — affiliation of all authors (Dept. of Automation / Dept. of Computer Science, BNRist).
- [[gpt-4]] — default LLM for insight extraction (`gpt-4-0613`) and a motivating example of a proprietary, API-only model that gradient-free methods must accommodate.
- [[faiss]] — vector store backing the experience pool / trajectory retrieval.

## Concepts

- [[experiential-learning]] — the paradigm this paper introduces and names.
- [[skill-library]] — contrasting representation: ExpeL's distilled NL insights vs. an executable code-skill repository.
- [[agent-skills]] — ExpeL anchors the "distilled experience" pole of the code-skills-vs-experience axis.
- [[memory-management]] — write-time insight consolidation + read-time trajectory retrieval.
- [[episodic-memory-llm]] — the retrieval-of-past-successes half of the method.
- [[reflexion]] — used internally for experience gathering; contrasted as intra-task vs. ExpeL's inter-task learning.
- [[voyager]] — the executable-skill counterpart in the cross-task experience-accumulation lineage.
- [[react]] — base planning policy.
- [[retrieval-augmented-generation]] — ExpeL retrieves self-generated experiences rather than gold/external corpora.
- [[in-context-learning]] — the mechanism through which insights and retrieved trajectories take effect.
- [[self-improving-agent]] — broader family.

## Quotes

> "Our agent autonomously gathers experiences and extracts knowledge using natural language from a collection of training tasks. At inference, the agent recalls its extracted insights and past experiences to make informed decisions." (Abstract)

> "Unlike self-improvement methods like Reflexion (Shinn et al. 2023), our approach emphasizes the importance of retaining experiences across multiple tasks to enhance agent performance. Moreover, ExpeL learns without parameter updates, making it compatible with powerful closed-source models like GPT-4 or Claude." (Introduction)

> "The student tackles practice problems multiple times to derive insights. At the exam, the student rely solely on these insights and draw memories of similar problems to answer the questions with one attempt." (Sec. 4)

> "ExpeL offers inherent interpretability, as both the extracted experiences and successful trajectories are presented in natural language. This design allows users to easily inspect, modify, or remove potentially harmful trajectories/insights — a challenge in finetuned models." (Sec. 4.5, ExpeL's Strengths)

## References

- Zhao, Huang, Xu, Lin, Liu, Huang. "ExpeL: LLM Agents Are Experiential Learners." AAAI 2024. [arXiv:2308.10144](https://arxiv.org/abs/2308.10144).
- Builds on [[reflexion]] (Shinn et al. 2023) and [[react]] (Yao et al. 2023) as base mechanisms.
- Contrast with [[voyager]] (Wang et al. 2023) on skill representation.
- Source document: `expel-zhao-2023.md`.
