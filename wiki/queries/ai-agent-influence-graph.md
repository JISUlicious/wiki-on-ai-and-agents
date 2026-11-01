---
title: "AI Agent Influence Graph: Dates and Evidence"
type: query
created: 2026-11-01
updated: 2026-11-01
sources:
  - react-yao-2022.md
  - schick-toolformer-2023.md
  - reflexion-shinn-2023.md
  - generative-agents-park-2023.md
  - swe-agent-yang-2024.md
status: complete
importance: high
---

# AI Agent Influence Graph: Dates and Evidence

Evidence for a graph of how AI agents developed, from classical agents to 2026. Page numbers are PDF pages of the local copies in `sources/`. An edge is only as strong as its type: a citation can mean a comparison or background mention, so read the "what it shows" column.

## Dates

| Item | Date | Source |
|---|---|---|
| [[eliza]] | 1966-01 | [CACM](https://web.stanford.edu/class/cs124/p36-weizenabaum.pdf) |
| [[strips]] | 1971 (paper not read) | [IJCAI](https://mlanthology.org/ijcai/1971/fikes1971ijcai-strips) |
| [[bdi-agent|BDI (Rao & Georgeff)]] | 1995 | [ICMAS](https://www.aaai.org/Papers/ICMAS/1995/ICMAS95-042.pdf) |
| [[intelligent-agent|AIMA 1st ed.]] | 1995 (not from a primary source) | [[intelligent-agent]] |
| [[word2vec]] | 2013-01 | [arXiv:1301.3781](https://arxiv.org/abs/1301.3781) |
| [[seq2seq]] | 2014-09 | [arXiv:1409.3215](https://arxiv.org/abs/1409.3215) |
| [[attention-mechanism|Attention]] | 2014-09 | [arXiv:1409.0473](https://arxiv.org/abs/1409.0473) |
| [[rlhf|RLHF]] | 2017-06 | [arXiv:1706.03741](https://arxiv.org/abs/1706.03741) |
| [[transformer]] | 2017-06 | [arXiv:1706.03762](https://arxiv.org/abs/1706.03762) |
| [[gpt-1]] | 2018-06 | [OpenAI PDF](https://cdn.openai.com/research-covers/language-unsupervised/language_understanding_paper.pdf) |
| [[gpt-2]] | 2019-02 | [OpenAI PDF](https://cdn.openai.com/better-language-models/language_models_are_unsupervised_multitask_learners.pdf) |
| [[gpt-3]] | 2020-05 | [arXiv:2005.14165](https://arxiv.org/abs/2005.14165) |
| [[webgpt]] | 2021-12 | [arXiv:2112.09332](https://arxiv.org/abs/2112.09332) |
| [[chain-of-thought]] | 2022-01 | [arXiv:2201.11903](https://arxiv.org/abs/2201.11903) |
| [[instructgpt]] | 2022-03 | [arXiv:2203.02155](https://arxiv.org/abs/2203.02155) |
| [[react]] | 2022-10 | [arXiv:2210.03629](https://arxiv.org/abs/2210.03629) |
| [[langchain]] | 2022-10 (repo) | [GitHub](https://github.com/langchain-ai/langchain) |
| [[toolformer]] | 2023-02 | [arXiv:2302.04761](https://arxiv.org/abs/2302.04761) |
| [[gpt-4]] | 2023-03 | [arXiv:2303.08774](https://arxiv.org/abs/2303.08774) |
| [[autogpt]] | 2023-03 (repo) | [GitHub](https://github.com/Significant-Gravitas/AutoGPT) |
| [[reflexion]] | 2023-03 | [arXiv:2303.11366](https://arxiv.org/abs/2303.11366) |
| [[chatgpt-plugins]] | 2023-03 | [OpenAI](https://openai.com/blog/chatgpt-plugins) |
| [[babyagi]] | 2023-04 (repo) | [GitHub](https://github.com/yoheinakajima/babyagi) |
| [[generative-agents]] | 2023-04 | [arXiv:2304.03442](https://arxiv.org/abs/2304.03442) |
| [[function-calling]] | 2023-06 | [OpenAI](https://openai.com/blog/function-calling-and-other-api-updates) |
| [[autogen]] | 2023-08 | [arXiv:2308.08155](https://arxiv.org/abs/2308.08155) |
| [[crewai]] | 2023-10 (repo) | [GitHub](https://github.com/crewAIInc/crewAI) |
| [[langgraph]] | 2024-01 | [LangChain](https://blog.langchain.com/langgraph/) |
| [[devin]] | 2024-03 | [Cognition](https://cognition.ai/blog/introducing-devin) |
| [[swe-agent]] | 2024-04 (repo), 2024-05 (arXiv) | [arXiv:2405.15793](https://arxiv.org/abs/2405.15793) |
| [[computer-use-agent|Claude computer use]] | 2024-10 | [Anthropic](https://www.anthropic.com/news/3-5-models-and-computer-use) |
| [[mcp]] | 2024-11 | [Anthropic](https://www.anthropic.com/news/model-context-protocol) |
| [[operator]] | 2025-01 | [OpenAI](https://openai.com/index/introducing-operator/) |
| [[claude-code]] | 2025-02 | [[claude-code]] |
| [[manus]] | 2025-03 (secondary sources) | [MIT TR](https://www.technologyreview.com/2025/03/11/1113133/manus-ai-review/) |
| [[openai-agents-sdk]] | 2025-03 | [OpenAI](https://openai.com/index/new-tools-for-building-agents/) |
| [[a2a]] | 2025-04 | [Google](https://developers.googleblog.com/en/a2a-a-new-era-of-agent-interoperability/) |
| [[codex-cli]] | 2025-04 | [OpenAI](https://openai.com/index/introducing-o3-and-o4-mini/) |
| [[acp]] | 2025-08 | [[acp]] |
| [[agent-skills]] | 2025-10 | [Anthropic](https://claude.com/blog/skills) |

## Confirmed edges

Direct citation with stated use:

| A → B | Where in B | What it shows |
|---|---|---|
| Transformer → GPT-1 | p. 2 | Uses the Transformer as its architecture |
| GPT-2 → GPT-3 | p. 4 | Builds on "in-context learning" |
| GPT-3 → WebGPT | p. 1 | Fine-tunes GPT-3 |
| GPT-3, RLHF → InstructGPT | p. 2 | Fine-tunes GPT-3 with RLHF |
| RLHF → GPT-4 | p. 2 | Fine-tuned with RLHF |
| CoT → ReAct | p. 2 | Critiques CoT as not grounded in the world, and compares against it |
| WebGPT → ReAct | pp. 2, 5 | Its acting-only baseline loosely resembles WebGPT |
| ReAct → Reflexion | p. 1 and experiments | Uses ReAct as the action generator |
| ReAct → SWE-agent | p. 3 | Its thought-and-command loop is attributed to ReAct |
| ReAct → MetaGPT | p. 4 | Agents follow ReAct-style behaviour |
| LSP → MCP | [Spec](https://modelcontextprotocol.io/specification/2025-06-18) | "takes some inspiration from" LSP |
| MCP → A2A | [Google post](https://developers.googleblog.com/en/a2a-a-new-era-of-agent-interoperability/) | Designed as a complement, not a descendant |
| Swarm → Agents SDK | [Swarm README](https://github.com/openai/swarm) | Direct replacement |
| MCP → Agents SDK | [README](https://github.com/openai/openai-agents-python) | Supported tool type and acknowledged |
| CUA → Operator → computer-use tool | [Operator](https://openai.com/index/introducing-operator/), [tools post](https://openai.com/index/new-tools-for-building-agents/) | Same model |

Weaker links, stated but not inheritance:

| A → B | Type | Where | What it shows |
|---|---|---|---|
| Seq2seq → Attention | citation, background | Bahdanau p. 1 | Names it among neural translation approaches |
| Attention → Transformer | citation, background | Vaswani p. 2, ref. [2] | Attention is the prior art it reworks |
| Seq2seq → Transformer | citation, background | Vaswani p. 12, ref. [35] | In the reference list; the citing sentence was not checked |
| GPT-1 → GPT-2 | citation | Radford 2019 p. 12 | In the reference list; the citing sentence was not checked |
| ELIZA → Generative Agents | citation, background | Park p. 3 | Formative work, not technical ancestry |
| WebGPT → Toolformer | citation, related work | Schick p. 10 | Listed; TALM is named as closest |
| ReAct → Toolformer | citation, related work | Schick p. 10 | Listed among search-engine tool users |
| Toolformer, WebGPT, Generative Agents → Reflexion | citation, context | Shinn p. 1 | Examples of autonomous decision-making |
| RLHF summarization (Stiennon 2020) → WebGPT | citation, method | Nakano p. 1 | Same training approach; not the 2017 paper directly |
| ReAct → LangChain | naming in code | [Issue #2284](https://github.com/langchain-ai/langchain/issues/2284) | `zero-shot-react-description` exists, but the issue says it follows MRKL |
| ReAct → AutoGPT, BabyAGI | developer mention (third party) | [Chase lecture](https://fullstackdeeplearning.com/llm-bootcamp/spring-2023/chase-agents/) | Chase, not the projects' authors, frames them as building on ReAct-style agents |
| AutoGPT → LangGraph | developer mention | [LangGraph post](https://blog.langchain.com/langgraph/) | Same simple loop; LangGraph adds control |
| LangChain ReAct → AutoGen | evaluation baseline | Wu p. 6 | A comparison only |
| GPT-3 in-context learning → Manus | developer mention | [Ji 2025](https://manus.im/blog/Context-Engineering-for-AI-Agents-Lessons-from-Building-Manus) | Design choice, stated by the team |
| Function calling → Agent Skills | contrast | [[anthropic-agent-skills-2025]] (a summary page) | Skills are contrasted with upfront tool declaration; primary wording not re-checked |

## Not confirmed

No direct evidence was found for these. Draw them dashed or leave them out.

| Relation | Status |
|---|---|
| AIMA → LLM-agent papers | No "Russell" in ReAct's text. Conceptual background only. |
| Sutton & Barto RL model → LLM agents | Inferred; the ReAct-style observation loop resembles it. |
| STRIPS, BDI → ReAct, LangChain, etc. | Inferred. None found in the checked papers. |
| word2vec → seq2seq | Shared authorship circle only; the seq2seq text search found Mikolov's other works, not a confirmed word2vec citation. |
| ChatGPT plugins → function calling → MCP | Plausible sequence by date. The Anthropic post does not say so. |
| Claude computer use → Operator | Date order only. |
| ReAct or SWE-agent → Devin | Devin's post cites neither; SWE-bench is shared. |
| Devin → Claude Code, Codex CLI; Claude Code ↔ Codex CLI | Same category; no stated influence found. |
| LangChain or AutoGen → CrewAI | README says standalone; not stated elsewhere in what was read. |
| Skills → MCP (either direction) | Not checked in primary wording. |

## Suggested 2026 additions

These already have wiki pages: [[context-engineering]], [[agentic-harness-engineering]], [[dynamic-workflows]], [[recursive-agent-harnessing]], [[self-improving-agent]], and [[cross-agent-comparison-2026]].

## Related

[[from-transformer-to-modern-agents]] · [[common-features-of-modern-agents]] · [[llm-agent]]
