---
title: "s3: Simple Scaling of Swarms via Cooperative Multi-agent RL"
type: source
created: 2026-11-17
updated: 2026-11-17
sources:
  - s3-simple-scaling-of-swarms-jha-2026.md
arxiv_id: "2611.08615"
venue: arXiv preprint (cs.MA)
authors:
  - Kunal Jha
  - Gonçalo Faria
  - Shuyue Stella Li
  - Scott Geng
  - Faeze Brahman
  - Noah A. Smith
  - Yulia Tsvetkov
  - Max Kleiman-Weiner
  - Natasha Jaques
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# s3: Simple Scaling of Swarms via Cooperative Multi-agent RL

**Source:** Kunal Jha, Gonçalo Faria, Shuyue Stella Li, Scott Geng, Faeze Brahman, Noah A. Smith, Yulia Tsvetkov, Max Kleiman-Weiner, Natasha Jaques ([[university-of-washington|University of Washington]]; Allen Institute for AI) — [arXiv:2611.08615](https://arxiv.org/abs/2611.08615), cs.MA, v1 5 Nov 2026.

## Summary

Most [[multi-agent]] systems use an organization fixed in advance: an orchestrator, debate rounds or assigned roles. s3 asks whether agents can learn to organize themselves instead. A team is N copies of one model. There is no leader, no role and no turn order. Each agent works on its own copy of the task and has two extra tools, `message(to, text)` and `read_messages()`. The harness runs agents asynchronously, splitting generation into requests of at most 1,024 tokens so messages can land while a teammate is still working.

Training is cooperative multi-agent RL with one shared team reward. For each problem the method samples G = 8 independent team attempts and gives every token from a team the same [[grpo|GRPO]]-style advantage, so the team, not the single response, is the unit compared. The reward is binary team success minus a token-cost term (λ = 0.25), charged only when the team succeeds. The authors compare against best-of-N from a solo policy trained with matched rollouts and per-agent token budgets, which is the baseline that usually erases multi-agent gains.

## Key points

- **Two tasks.** *Information aggregation* (Qwen3-0.6B): each agent holds one private number and some agent must submit the full sorted list. *Decentralized exploration* (Qwen3-1.7B): agents search a tree of 391 rooms for an exit with an 8,192-token allowance each. Swarms train with 4 agents for 200 RL updates.
- **Aggregation at N = 4.** s3 reaches 93% success, against 35% for online SFT on successful swarms and 13% with individual rewards. About 80% of swarms settle on one broadcast per agent, and agents learn to share before they read.
- **Scaling past training size.** At 8 agents s3 succeeds more than 20× as often as online SFT; the abstract reports over 25× the strongest baseline at twice the training size. Its lead holds through 12 agents, and all methods struggle at 16.
- **Exploration.** Removing messaging cuts 4-agent success by 19 points. Communicating agents revisit nearly half as many of the same rooms. At 6 agents s3 beats best-of-six by about 7 points, and the abstract reports up to 9.4 points over best-of-N. Four s3 agents nearly match six solo-trained agents.
- **Social dilemma.** With individual rewards, aggregation agents learn to read teammates' numbers before sending their own, and many never send. It behaves like a public-goods game. A shared reward removes the dilemma. In exploration any agent can win alone, so individual reward still works there.
- **Demonstrations matter.** Without SFT warm-start, the exploration swarm shows no significant advantage over solo best-of-four.
- Off-policy correction uses IcePop with score centering ([[score-centering-stabilizes-off-policy-rl-marek-2026]]).

## Caveats

Two synthetic tasks, small Qwen3 models, and one training run per condition (six evaluation seeds). Several exploration comparisons are significant only before Bonferroni correction.

## Concepts & entities

- [[multi-agent]] · [[agentic-rl]] · [[grpo]] · [[reinforcement-learning]] · [[supervised-fine-tuning]] — learned, not designed, coordination.
- [[inference-time-compute]] — the matched best-of-N comparison.
- [[self-organizing-agent-teams-learn-to-reason-together-pappu-2026]] · [[scaling-discovery-requires-forgetting-tan-2026]] — related 2026 work on learned or self-organizing teams.
- [[university-of-washington]] · [[qwen]].

## References

- [arXiv:2611.08615](https://arxiv.org/abs/2611.08615)
