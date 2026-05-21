---
title: "UI-TARS: Pioneering Automated GUI Interaction with Native Agents"
type: source
created: 2026-05-21
updated: 2026-05-21
sources:
  - ui-tars-1-qin-2025.md
arxiv_id: "2501.12326"
authors:
  - Yujia Qin
  - Yining Ye
  - Junjie Fang
  - Haoming Wang
  - Shihao Liang
  - Shizuo Tian
  - Junda Zhang
  - Jiahao Li
  - Yunxin Li
  - Shijue Huang
  - Wanjun Zhong
  - Kuanye Li
  - Jiale Yang
  - Yu Miao
  - Woyu Lin
  - Longxiang Liu
  - Xu Jiang
  - Qianli Ma
  - Jingyu Li
  - Xiaojun Xiao
  - Kai Cai
  - Chuang Li
  - Yaowei Zheng
  - Chaolin Jin
  - Chen Li
  - Xiao Zhou
  - Minchao Wang
  - Haoli Chen
  - Zhaojian Li
  - Haihua Yang
  - Haifeng Liu
  - Feng Lin
  - Tao Peng
  - Xin Liu
  - Guang Shi
year: 2025
introduces:
  - "[[ui-tars-1]]"
  - "[[computer-use-agent]]"
tags:
  - 2025
status: complete
importance: high
---

# UI-TARS: Pioneering Automated GUI Interaction with Native Agents

[arXiv:2501.12326](https://arxiv.org/abs/2501.12326) — ByteDance Seed (with Tsinghua University interns), January 2025. Corresponding authors: **Yujia Qin** and **Guang Shi**. Open-sourced at <https://github.com/bytedance/UI-TARS>.

## Summary

UI-TARS is [[bytedance-seed]]'s first-generation **native GUI agent model** — an end-to-end vision-language model that perceives only raw screenshots and emits human-like mouse and keyboard actions, without DOM parsing, accessibility trees, or a separate planner/grounder stack. The paper frames the field's evolution as four stages (rule-based RPA → modular LLM agent frameworks → **native agent models** → active/lifelong agents) and positions UI-TARS at the third stage: a single model that unifies the four core capabilities of GUI agency — **perception, action, reasoning, and memory** — within one set of weights. It is the direct predecessor to [[ui-tars-2]] (arXiv:2509.02544), which scales this same blueprint with stabilized multi-turn RL.

The model is built by continually training Qwen-2-VL 7B and 72B on roughly 50B tokens spanning four data pillars: (1) **enhanced GUI perception** via a large-scale screenshot dataset (element descriptions, dense captioning, state-transition captioning, QA, set-of-mark); (2) **unified action modeling** that standardizes click/type/scroll/drag across mobile, desktop, and web into one action space with large-scale grounded traces; (3) **System-2 reasoning** infused by injecting task decomposition, reflection, milestone recognition, and trial-and-error "thoughts" before each action, supported by ~6M crawled GUI tutorials; and (4) an **iterative training data flywheel** in which hundreds of virtual machines roll out new traces, multi-stage filtering (heuristics → VLM scoring → human review) cleans them, and DPO-based **reflection tuning** on error-correction and post-reflection pairs lets the model learn to recover from its own mistakes. The result reaches SOTA on 10+ GUI benchmarks at the time of release: **OSWorld 24.6 @ 50 steps / 22.7 @ 15 steps** (vs Claude Computer Use 22.0 / 14.9), **AndroidWorld 46.6** (vs GPT-4o 34.5), **ScreenSpot Pro 38.1**, **VisualWebBench 82.8** for the 72B variant.

## Key Points

- **Native end-to-end GUI agent — pure pixels in, actions out.** No DOM, HTML, or a11y tree dependency; the model perceives screenshots directly and outputs mouse/keyboard primitives. Framed as the path beyond modular agent frameworks (which the paper criticizes as fragile, design-driven, and non-scalable).
- **Perception · Action · Reasoning · Memory framing.** UI-TARS-1 codifies the four-component decomposition of a native GUI agent that [[ui-tars-2]] later inherits wholesale and trains end-to-end with multi-turn RL.
- **Architecture.** Interaction is a sequence `(instruction, (o₁, t₁, a₁), …, (oₙ, tₙ, aₙ))` — observation, explicit "thought," action — inspired by [[react]] but with more structured System-2 deliberation. Context window holds the last N observations; full thought/action history kept as short-term memory.
- **Enhanced perception data**: element description, dense captioning, state-transition captioning, QA, and set-of-mark prompting, built from auto-crawled screenshots + metadata (element type, bounding box, depth, text content) across web/app/OS.
- **Unified action space** across mobile, desktop, and web: atomic actions (click, type, scroll, drag, hotkey) plus compositional sequences; large-scale grounded action traces train precise coordinate prediction.
- **System-2 reasoning injection**: ~6M crawled GUI tutorials filtered/refined, plus augmentation of action traces with task decomposition, long-term consistency, milestone recognition, trial-and-error, and reflection patterns.
- **Iterative data flywheel**: hundreds of VMs explore tasks, generate traces, multi-stage filter (rules → VLM scoring → human review), feed back into training. **Reflection tuning** uses paired error-correction and post-reflection annotations trained via DPO so the agent learns recovery, not just success.
- **Training**: continual training of Qwen-2-VL 7B/72B on ~50B tokens to produce UI-TARS-7B / UI-TARS-72B.
- **Benchmark headlines**: OSWorld 24.6 (50 steps) / 22.7 (15 steps) beating Claude Computer Use; AndroidWorld 46.6 beating GPT-4o; ScreenSpot Pro 38.1 SOTA; VisualWebBench 82.8 (72B) > GPT-4o 78.5; ScreenSpot v2 91.6 (7B) / 90.3 (72B) > OS-Atlas-7B 87.1.
- **Stage-4 prospect.** The paper explicitly positions UI-TARS-1 as a stepping stone toward "active and lifelong" agents that propose tasks, self-reward, and learn autonomously — the program [[ui-tars-2]] then pursues with large-scale multi-turn RL.
- **Predecessor to [[ui-tars-2]].** Same four-capability framing, same native end-to-end philosophy, same data-flywheel ethos; UI-TARS-2 replaces SFT+DPO with stabilized multi-turn PPO and broadens the environment to include file systems and terminals.

## Entities

- [[bytedance]] — parent organization.
- [[bytedance-seed]] — research lab that produced UI-TARS.

## Concepts

- [[computer-use-agent]] — the broader category UI-TARS pioneers in the open.
- [[gui-agent]] — narrower screen-driven framing; UI-TARS is the canonical native instantiation.
- [[ui-tars-2]] — direct successor; scales the same framework with multi-turn RL and hybrid GUI/file/terminal environments.
- [[osworld]] — primary desktop GUI benchmark; UI-TARS-72B scores 24.6 / 22.7.
- [[react]] — cited as the inspiration for the explicit "thought before action" pattern, here generalized into structured System-2 reasoning.

## Notable Quotes

> This paper introduces UI-TARS, a native GUI agent model that solely perceives the screenshots as input and performs human-like interactions (e.g., keyboard and mouse operations). Unlike prevailing agent frameworks that depend on heavily wrapped commercial models (e.g., GPT-4o) with expert-crafted prompts and workflows, UI-TARS is an end-to-end model that outperforms these sophisticated frameworks.

> UI-TARS incorporates several key innovations: (1) Enhanced Perception … (2) Unified Action Modeling … (3) System-2 Reasoning … (4) Iterative Training with Reflective Online Traces, which addresses the data bottleneck by automatically collecting, filtering, and reflectively refining new interaction traces on hundreds of virtual machines.

> Frameworks are design-driven, meaning they lack the ability to learn and generalize across tasks without continuous human involvement … The future of autonomous agent development lies in the creation of native agent models, where workflow knowledge is embedded directly within the agent's model through orientational learning.

> In OSWorld, UI-TARS-72B achieves scores of 24.6 with 50 steps and 22.7 with 15 steps, outperforming Claude's 22.0 and 14.9 respectively. In AndroidWorld, it achieves a score of 46.6, outshining GPT-4o's 34.5.

## References

- Paper: [arXiv:2501.12326](https://arxiv.org/abs/2501.12326)
- Code / project: <https://github.com/bytedance/UI-TARS>
- Successor: [[ui-tars-2-wang-2025]] (arXiv:2509.02544) — multi-turn RL, hybrid GUI/file/terminal environments, OSWorld 47.5.
- Base model: Qwen-2-VL (7B, 72B), continually trained on ~50B tokens.
- Benchmarks cited: [[osworld]], AndroidWorld, ScreenSpot / ScreenSpot v2 / ScreenSpot Pro, VisualWebBench, WebSRC, ScreenQA, OfficeBench, AssistGUI, LlamaTouch, B-MOCA.
- Compared against: Claude Computer Use (Anthropic 2024b), GPT-4o, UGround, OS-Atlas, Aguvis, ShowUI, SeeAct.
