---
title: Institutional Alignment
type: concept
created: 2026-05-13
updated: 2026-05-13
sources:
  - Agentic AI and the next intelligence explosion.md
status: draft
importance: medium
---

# Institutional Alignment

Institutional alignment is the proposal that the way to align AI systems at scale is not (or not only) to align individual models' weights via [[rlhf|RLHF]]-style techniques, but to **scaffold roles, norms, and protocols** in which models operate — analogously to how human societies achieve coordination via institutions (courts, markets, bureaucracies) rather than via individual virtue alone.

Articulated in [[agentic-ai-and-the-next-intelligence-explosion|Evans, Bratton, & Agüera y Arcas (2026)]], drawing on Elinor Ostrom (commons governance) and Douglass North (institutional economics).

## The argument

- **[[rlhf|RLHF]] is dyadic** — a parent-child model of correction. It doesn't scale to ecologies of billions of agents where any given agent interacts with many others.
- **Institutions are role-based**, not identity-based: a courtroom functions because "judge," "attorney," and "jury" are well-defined slots independent of who occupies them. The same agent could play different roles in different contexts.
- **Constitutional structure**: AI agents in adversarial / checking roles across institutional boundaries. A labor-department AI audits a corporation's hiring algorithm; a judicial-branch AI evaluates an executive-branch AI's risk assessments.
- **Protocol-level alignment**: deliberation procedures, delegation mechanisms, conflict-resolution scaffolds — these have as much real-world effect as any laws.

## Relation to other alignment approaches

- **[[rlhf]] / RLAIF / Constitutional AI**: bake values into individual model weights. Institutional alignment is complementary, not opposed — but argues it's insufficient at ecosystem scale.
- **Sandboxing / capability containment**: limit what any one model can do. Institutional alignment is more about *what models do together* and how their actions are checked.
- **Interpretability** ([[mechanistic-interpretability]], [[functional-emotions]]): understand and intervene on individual model internals. Institutional alignment is the macro counterpart.

## Open questions

- Which institutions transfer from human society? Courts, markets, peer review, scientific publication all have human-specific failure modes that may or may not apply to AI ecologies.
- How are AI institutions enforced? Without physical embodiment or social embedding, what gives an "AI judge" actual authority?
- How do you avoid regulatory capture / cartel formation when institutional roles are filled by AIs from a small number of frontier labs?

## References

- [[agentic-ai-and-the-next-intelligence-explosion]]
