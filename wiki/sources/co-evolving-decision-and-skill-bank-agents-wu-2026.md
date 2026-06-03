---
title: "Co-Evolving Decision and Skill Bank Agents (COS-PLAY)"
type: source
created: 2026-06-03
updated: 2026-06-03
sources:
  - co-evolving-decision-and-skill-bank-agents-wu-2026.md
arxiv_id: "2604.20987"
authors:
  - Xiyang Wu
  - Zongxia Li
  - Guangyao Shi
  - Alexander Duffy
  - Tyler Marques
  - Matthew Lyle Olson
  - Tianyi Zhou
  - Dinesh Manocha
first_author: Xiyang Wu
year: 2026
tags:
  - 2026
status: complete
importance: medium
---

# Co-Evolving Decision and Skill Bank Agents (COS-PLAY)

*Surfaced via the alphaXiv / NLP-newsletter digest.*

[arXiv:2604.20987](https://arxiv.org/abs/2604.20987) — Wu et al. (Univ. of Maryland, USC, et al.), 2026.

## Summary

Long-horizon interactive environments (games) demand multi-step reasoning, chaining many skills over many timesteps, and decision-making under delayed rewards and partial observability. LLM game agents tend to struggle here because they lack a mechanism to discover, retain, and reuse structured skills across episodes. **COS-PLAY** is a co-evolution framework with two cooperating agents: an LLM **decision agent** that retrieves skills from a learnable **skill bank** to guide action-taking, and an **agent-managed skill pipeline** that discovers reusable skills from the agent's unlabeled rollouts to populate that bank. The two co-evolve — the decision agent learns better skill retrieval and action generation, while the skill-bank agent continually extracts, refines, and updates skills together with their "contracts."

## Key points

- Targets the explore–learn–reuse loop: the bottleneck is organizing, refining, and retrieving skills for reliable reuse, not just acquiring them.
- Maintains an external **skill bank** of reusable, temporally extended behavior protocols with associated contracts.
- **Decision agent** retrieves from the bank to take actions; **skill-bank agent** mines reusable skills from unlabeled rollouts — the two co-evolve.
- Skills are continually extracted, refined, and updated rather than fixed once learned.
- With an **8B base model**, achieves >25.1% average reward improvement over four frontier LLM baselines on single-player game benchmarks.
- Remains competitive on multi-player social reasoning games; evaluated across six game environments.

## Concepts & entities

- [[skill-acquisition]]
- [[skill-optimization]]
- [[llm-agent]]
- [[self-improving-agent]]

## References

- [arXiv:2604.20987](https://arxiv.org/abs/2604.20987)
- Code: https://github.com/wuxiyang1996/COS-PLAY
- Local source: `sources/co-evolving-decision-and-skill-bank-agents-wu-2026.md`
