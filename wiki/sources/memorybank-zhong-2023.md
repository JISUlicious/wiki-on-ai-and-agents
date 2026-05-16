---
title: "MemoryBank: Enhancing Large Language Models with Long-Term Memory"
type: source
created: 2026-05-16
updated: 2026-05-16
sources:
  - memorybank-zhong-2023.md
arxiv_id: "2305.10250"
venue: AAAI 2024
authors:
  - Wanjun Zhong
  - Lianghong Guo
  - Qiqi Gao
  - He Ye
  - Yanlin Wang
first_author: Wanjun Zhong
year: 2023
introduces:
  - "[[memorybank]]"
  - "[[ebbinghaus-forgetting]]"
tags:
  - 2023
status: complete
importance: high
---

# MemoryBank: Enhancing Large Language Models with Long-Term Memory

- **Source**: `sources/memorybank-zhong-2023.md`
- **arXiv**: [2305.10250](https://arxiv.org/abs/2305.10250)
- **Authors**: Wanjun Zhong, Lianghong Guo, Qiqi Gao, He Ye, Yanlin Wang (Sun Yat-Sen University; Harbin Institute of Technology; KTH)
- **Venue**: AAAI 2024
- **Year**: 2023

## Summary

MemoryBank proposes a long-term memory mechanism for LLMs, motivated by the observation that base models like ChatGPT and ChatGLM cannot sustain meaningful state across the kinds of extended interactions required by personal AI companions, psychological counselors, or secretarial assistants. The system is built around three pillars: (1) a **memory storage** that records timestamped multi-turn conversations, distills them into hierarchical daily/global event summaries, and continuously refines a user-personality profile; (2) a **dual-tower dense retriever** (DPR-style, with FAISS indexing) that pulls relevant memory chunks given the current conversational context; and (3) a **memory updater** governed by a psychologically motivated forgetting policy.

The paper's most distinctive contribution is the **forget+reinforce policy** drawn from the [[ebbinghaus-forgetting]] curve. Each memory item carries a strength `S`, initialized to 1 on first mention; retention decays as `R = e^(-t/S)`, where `t` is time since the last interaction with the item. When the retriever surfaces a memory during a conversation, the system increments `S` by 1 and resets `t` to 0 — recalled memories become harder to forget, mirroring the spacing effect. To the best of the authors' framing, this is the first principled, time-dependent forgetting policy applied to LLM memory rather than naive fixed-size context windows or pure similarity-based eviction.

To demonstrate practicality, the authors build **SiliconFriend**, an AI-companion chatbot that wraps MemoryBank around three backbones (ChatGPT, ChatGLM-6B, BELLE-7B). The open-source variants are first LoRA-tuned on 38k psychological dialogues for empathetic response, then augmented with MemoryBank. Evaluation covers qualitative real-user dialogs plus a quantitative suite of 194 probing questions over a 10-day simulated dialog history (15 virtual users role-played by ChatGPT, English + Chinese). SiliconFriend-ChatGPT achieves 0.763 retrieval accuracy / 0.716 correctness / 0.912 coherence on English, with the open-source variants showing comparable retrieval but weaker response quality — establishing that the MemoryBank mechanism itself generalizes across backbones.

## Key Points

- **Hierarchical storage**: raw timestamped turns → per-day event summaries → global event summary; in parallel, per-day personality insights → global user portrait. Summaries are produced by prompting the underlying LLM ("Summarize the events and key information in the content [dialog/events]" and similar for personality).
- **Retrieval**: dual-tower dense retrieval in the [[dpr|dense passage retrieval]] style. Each memory piece `m` is pre-encoded into `h_m`; current context `c` is encoded into `h_c`; FAISS does nearest-neighbor over `M = {h_m^0, ..., h_m^|M|}`. Encoder is interchangeable — MiniLM for English, Text2vec for Chinese in the released SiliconFriend.
- **Forgetting formula**: `R = e^(-t/S)` (Ebbinghaus exponential decay). `S` is discretized: init 1, increment by 1 on recall, simultaneously reset `t = 0`. Authors explicitly flag this as "exploratory and highly simplified."
- **Three Ebbinghaus principles modeled**: (1) rate of forgetting (exponential decay), (2) initial-steepness of the curve, (3) spacing effect via reinforcement on recall. Overlearning and meaningful-material effects are explicitly out of scope.
- **Backbone-agnostic**: same mechanism plugs into closed-source ChatGPT and open-source ChatGLM/BELLE; LangChain used for retrieval glue in the released code.
- **Bilingual**: English + Chinese, with language-specific embedding models.
- **Prompt assembly at inference**: relevant retrieved memory + global user portrait + global event summary are concatenated into the conversation prompt as a "memory-augmented prompt."
- **Evaluation metrics**: retrieval accuracy, response correctness, contextual coherence, and a pairwise ranking score across the three SiliconFriend variants.
- Code and demo released at `github.com/zhongwanjun/MemoryBank-SiliconFriend`.

## Entities Mentioned

- [[chatgpt]] — closed-source backbone
- [[chatglm]] — open-source bilingual 6.2B backbone, GLM family
- BELLE — LLaMA-7B-derived open-source bilingual backbone (instruction-tuned via ChatGPT-synthesized data)
- [[llama]] — base for BELLE
- [[faiss]] — vector index for memory retrieval
- LangChain — retrieval orchestration library
- MiniLM — English sentence encoder
- Text2vec — Chinese sentence encoder

## Concepts Discussed

- [[memorybank]]
- [[ebbinghaus-forgetting]]
- [[memory-management]]
- [[memory-decay]]
- [[long-term-memory-llm]]
- [[dpr|dense passage retrieval]] — retrieval substrate
- [[lora]] — used for psychological-dialogue fine-tuning of open-source backbones (rank 16, 3 epochs on A100)
- [[rag]] — broader family the retrieval step belongs to
- [[user-modeling]] — global user portrait as evolving summary

## Notable Quotes

> "To mimic anthropomorphic behaviors and selectively preserve memory, MemoryBank incorporates a memory updating mechanism, inspired by the Ebbinghaus Forgetting Curve theory. This mechanism permits the AI to forget and reinforce memory based on time elapsed and the relative significance of the memory."

> "The Ebbinghaus forgetting curve is expressed using an exponential decay model: R = e^(−t/S) … To simplify memory updating process, we model S as a discrete value and initialize it with 1 upon its first mention in a conversation. When a memory item is recalled during conversations, it will persist longer in memory. We increase S by 1 and reset t to 0, hence forget it with a lower probability."

## References

_Original source: `sources/memorybank-zhong-2023.md`_
