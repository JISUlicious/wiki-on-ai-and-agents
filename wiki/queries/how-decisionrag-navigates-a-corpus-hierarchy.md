---
title: "How does DecisionRAG walk the corpus tree, and what reaches the generator?"
type: query
created: 2026-11-11
updated: 2026-11-11
sources:
  - retriever-free-retrieval-decision-models-wan-2026.md
  - system-one-models-are-not-better-rerankers-dinzinger-2026.md
status: complete
importance: medium
---

# How does DecisionRAG walk the corpus tree, and what reaches the generator?

**Questions (asked in sequence, 2026-11-10/11):**
1. Explain DecisionRAG.
2. Does it walk the tree by choosing which branch to descend as the retrieval step?
3. Why a tree and not a graph?
4. Are the selections from all levels fed into the model after retrieval?

All answers draw on [[retriever-free-retrieval-decision-models-wan-2026]] (Wan et al., [arXiv:2610.35868](https://arxiv.org/abs/2610.35868)). Where an answer goes beyond the paper, it is marked **(inference)**.

## 1. What DecisionRAG is

DecisionRAG does [[retrieval-augmented-generation|RAG]] retrieval with no search index and no embeddings. A [[system-one-model|decision model]] ([[jev|Jev]] 1.13, untrained for the task) reads the corpus's own structure (help center → page → heading → passage, or topic cluster → page → paragraph) and decides at each level which branches to open.

The motivating problem is the "capability asymmetry". A frontier generator writes the answer, but a much weaker retriever ([[bm25|BM25]] or [[dense-retrieval|dense retrieval]]) decides what it reads, and what the retriever misses the generator cannot recover.

It uses the same two question types as the [[openai-decisions-api|Decisions API]]:
- **yes/no** (`noul`) questions to screen candidates inside the tree;
- **one multiple-choice** question at the end to rank the shortlist.

See [[decision-api-vs-llm-and-server-implementations]] for those types. Headline numbers are on the source page.

## 2. Yes, each level is one step, but it is a beam, not a single path

Algorithm 1 in the paper:

```
C ← children of root
for each level ℓ = 1 … L−1:
    p ← BinaryJudge(q, C)            # P(yes) per candidate, all in one shared context
    C ← children of the top-kℓ candidates
p ← BinaryJudge(q, C)                # passages
R ← top-r passages
optionally: ChoiceJudge(q, R[1:m])   # reorder the top m (m = 20; 5 on NQ)
```

- **Keeps several branches.** Each level keeps a budget of `kℓ` branches. The budget can also be a probability rule, e.g. keep the runner-up help center whenever the top score is below 0.8.
- **Yes/no inside, multiple choice only at the end.** The paper's reasons: yes/no has no option limit, can be split into parallel requests, and favours recall while the search narrows. Multiple choice sharpens the top rank once, at the end (+6.8 Hit@1).
- **Shared context makes scores comparable.** Candidates at a level are judged together, so a passage that looks relevant alone gets a lower P(yes) next to a better one (+7.7 R@1 on MultiDoc2Dial, +6.5 Hit@1 on NQ). Splitting a pool into smaller requests lowers Hit@1 but leaves R@10 unchanged. Split scores are fine for screening; the final ranking needs one shared request.
- **Latency follows depth; cost follows breadth.**
  - **Latency:** T(N) ≈ 0.2 s × ⌈log_b N⌉. With b = 100, 10⁹ passages need 5 decisions, about 1 s.
  - **Cost:** grows with the number of judgments, at most k × b candidates per level.
- **No backtracking.** A branch dropped at one level is never revisited for that query. Budgets therefore lean toward recall, and very tight budgets lose the gold cluster (R@10 63.7% at 1% of NQ clusters).
- **The model scores but does not steer.** Depth and budgets are fixed in advance. No LLM decides when to stop or what to search next. This separates it from [[agentic-search]] and from GPT-5 walking the same tree by writing out its choices, which was 34–145× slower and 21–25× more expensive with no consistent accuracy gain.

### Per-benchmark configurations (Appendix B)

| Benchmark | Tree | Budgets |
|---|---|---|
| MultiDoc2Dial | 4 help centers → 413 pages → 4,110 passages | Help centers described by hand-written one-liners |
| ConditionalQA | 652 guides → 58,483 paragraphs | All guides in one request, keep 3; their paragraphs scored together under heading paths |
| MTRAG | 8,578 page titles → (headings) → passages | Pooled first pass keeps 50 pages; rescored together with product names and headings, keep 10. Pages with more than 60 passages get a heading stage that keeps 8 headings |
| Natural Questions | 1,000 k-means clusters → 108,626 pages → 2.68M paragraphs | Keep 150 clusters (about 17,500 titles, scored in parallel ≤20k-token requests) → ~230 paragraphs from the top 30 pages. Paragraph score × page score |

On NQ the clusters were built offline with recursive [[k-means]]. Each cluster gets a one-sentence description (≤30 words) written by Qwen3-8B from its page titles.

## 3. Why a tree, not a graph

The paper never discusses this and never mentions graphs. The reasons below are an **(inference)** from how the method is built.

1. **The structure already exists as a tree.** Tables of contents, HTML headings and page/paragraph splits are trees, and the selling point is "use the corpus's own structure."
2. **Pruning a tree loses nothing by accident.** Each passage has one path from the root. A graph lets a node be reached by several paths:
   - you need a visited set, or a passage is scored twice or listed twice;
   - the NQ rule "paragraph score × page score" stops working when there is more than one parent.
3. **The cost bounds depend on it.** Fixed depth gives the log_b N latency and the k × b cost per level. Cycles or shortcut edges need a stop rule, and that brings back the agentic stop-or-continue decision the method avoids.
4. **Candidates at one level are the same kind of thing.** All clusters, or all pages, so their P(yes) values are comparable. A graph frontier would mix kinds in one request.
5. **Building one is cheap.** Recursive k-means makes a tree offline when the corpus has no structure.

**What a tree costs:**
- **One parent per node:** a document that belongs under two topics sits under one, and can be pruned if the query fits the other. The paper's fixes are wider budgets and adding dense-retrieval candidates at the last stage, not extra edges.
- **No links between documents:** following a link from one document to another (multi-hop) isn't possible. The benchmarks rarely need it; on NQ the gold paragraphs always sit on one page.

**When a graph would be worth it (inference):**
- Multi-hop questions.
- Heavily linked corpora (wikis, codebases, knowledge graphs).
- Documents that really belong to several topics.

You could keep the decision-model scoring and swap the traversal for a beam search over the graph with a visited set, at the price of the clean latency bound and a needed stop rule. A middle option is a tree where a node may have several parents (a DAG), with duplicates removed at the leaves.

## 4. What reaches the generator: only the final passages

The answer-quality test (Appendix C.1) gives the generator, Qwen3.5-27B (thinking off, greedy), only:
- a short task instruction;
- the **top 5 passages**, numbered in rank order;
- the query (on MTRAG, the last six turns).

No cluster descriptions, page lists or scores from the walk are passed on. The upper levels still matter in three ways:
1. **Which passages can be reached at all:** a passage survives only if every ancestor was kept.
2. **Final scores, on NQ only:** paragraph P(yes) is multiplied by page P(yes).
3. **What the *scoring* model sees.** Passages are shown with their location:
   - on MultiDoc2Dial, "full text + page and parent-heading path";
   - on ConditionalQA, grouped under heading paths;
   - on NQ, grouped under the page title.

The paper doesn't say whether that path is also shown to the generator.

**Practical suggestion (inference, untested in the paper).** Pass each passage's path to the generator (`Help center > Page > Heading > passage`). It costs few tokens, and a bare paragraph often lacks its subject ("It was released in 2000…"). Passing the walk's scores or the pruned alternatives is less clearly useful: it adds tokens the generator can't do much with.

## Related tension

[[system-one-models-are-not-better-rerankers-dinzinger-2026]] finds that decision-fine-tuned models up to 9B are not better rerankers. The two results can both hold. DecisionRAG's gains come from shared-context scoring and tree navigation with a frontier decision model, and navigation beats using Jev only to rerank a dense top 100 by about 7 Hit@1. They do not come from decision fine-tuning of small models.

## Related hierarchical-retrieval work (cited by the paper)

The paper cites several earlier methods that also retrieve by walking a structure:
- **MemWalker:** navigates a tree of summaries built from long documents.
- **PageIndex:** LLM-guided navigation over a document's table of contents.
- **LATTICE:** a semantic tree over a whole corpus, with branches chosen by relevance judgments the LLM writes out.
- **RAPTOR:** recursive summary trees.
- **DocQN:** RL-trained tree navigation.
- **Dense hierarchical retrieval:** trained encoders, document then passage.

DecisionRAG's difference is that each judgment is a prefill-only decision with no generated tokens, made in one shared context per level.

## References

- [[retriever-free-retrieval-decision-models-wan-2026]]
- [[system-one-models-are-not-better-rerankers-dinzinger-2026]]
- [[system-one-model]] · [[jev]] · [[retrieval-augmented-generation]] · [[agentic-search]]

