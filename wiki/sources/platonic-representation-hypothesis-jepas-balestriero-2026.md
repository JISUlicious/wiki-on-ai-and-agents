---
title: "Proving and Improving the Platonic Representation Hypothesis with JEPAs"
type: source
created: 2026-11-03
updated: 2026-11-03
sources:
  - platonic-representation-hypothesis-jepas-balestriero-2026.md
arxiv_id: "2610.39302"
authors:
  - Randall Balestriero
  - [[yann-lecun]]
first_author: Randall Balestriero
introduces:
  - [[platonic-representation-hypothesis]]
year: 2026
status: complete
importance: high
tags:
  - 2026
  - paper
---

# Proving and Improving the Platonic Representation Hypothesis with JEPAs

**Randall Balestriero** (Brown University, AMI Labs) and **[[yann-lecun|Yann LeCun]]** (NYU, AMI Labs). [arXiv:2610.39302](https://arxiv.org/abs/2610.39302), v1, cs.AI (also cs.CV, stat.ML), 29 Oct 2026. No venue is listed; this is a preprint.

## Summary

The [[platonic-representation-hypothesis|Platonic Representation Hypothesis]] (PRH; Huh et al., ICML 2024) observed that vision and language models represent the world more alike as they scale, and conjectured that they converge to a shared model of reality. This paper turns that observation into a theorem for [[jepa|JEPAs]] and draws a recipe from it.

The argument is short. A JEPA with whitened, second-order anti-collapse learns the spectral embedding of the graph its positive pairs draw on the world. If two modalities observe one world and their positive pairs relate the **same world states**, their training problems are the same problem in different coordinates, so their solutions agree **up to one rotation**. The authors call this the **Platonic Representation Theorem**. Its practical claim is that the PRH is about *which pairs count as positives*, not about scale.

- **Good views:** two observations of one scene, such as two photographs or two captions of the same image. They share only the scene.
- **Bad views:** two corruptions of one observation, such as two crops of one photo or two maskings of one sentence. They also share that observation's *nuisance* (its wording, viewpoint or lighting), and no rotation can align a nuisance across modalities.

The paper then argues that **next-token prediction has the form of a bad-view JEPA**: its positives are a prefix and its continuation in one document, and its anti-collapse term is the softmax log-partition. This is offered as a reason why language models align with vision only partly, and mostly through scale. The authors say this is a heuristic prediction, not a corollary, and test it empirically.

## Theory

| Result | Statement (informal) |
|---|---|
| Theorem 1 (Platonic Representation Theorem) | Modalities that share one positive-pair distribution, with minimizers unique up to rotation, learn one embedding of the world up to one rotation each. |
| Proposition 1 | With squared-error prediction and whitening, minimizers are exactly rotations of the top-k eigenvectors of the positive-pair graph (Laplacian eigenmaps), unique when the spectrum has a gap. |
| Theorem 2 | When positives differ, misalignment after the best rotation is bounded by the kernel mismatch divided by the eigengap. On 300 random finite worlds the error stays below the bound, at most 18% of it. |
| Proposition 2 | No first- or second-moment statistic of unpaired data finds the rotation; k pairs in general position do (orthogonal Procrustes). |
| Theorem 3 and Proposition 3 | Every direction an encoder spends on its modality's nuisance is lost for alignment. A bad-view JEPA spends directions on nuisance exactly when its corruptions keep more of the nuisance than of the world's k-th direction; a good-view JEPA spends none. |
| Theorem 4 | Views and data together set the "world kernel" a JEPA learns. Good views fix it by design; bad views get it from the data, so weak corruptions need dense data and strong ones mix scenes at any data size. |
| Proposition 4 | The next-token loss has the form of a JEPA. Its minimizers are identified only up to an invertible linear map, not a rotation. |

## Key results

Most experiments vary the text encoder (a BERT trained from scratch with the LeJEPA objective) against a frozen [[dinov2|DINOv2]], the vision model the original PRH paper used. No encoder ever sees the other modality.

- **Flickr30k, matched data and encoder.** With caption views the true caption is retrieved first for 4.4% of test images through one rotation (44× chance). Masked spans and next-token prediction nearly coincide well below that. Next-token prediction trained on *pairs* of captions of one image barely moves (R@1 .021 → .023), so the gain comes from using those pairs as JEPA views. The scene explains a fraction ρ = .804 of the good-view embedding against .655 (bad-view) and .608 (next-token).
- **Trimming makes good views.** Of the settings varied, trimming views to spans matters most: whole captions lose about a quarter of the alignment.
- **Web text scales.** A 33M text JEPA trained on texts of 50M Recap-DataComp images reaches .125 on WIT-1024 (mutual 10-NN with DINOv2), level with [[qwen-2-5|Qwen2.5]]-3B and -7B.
- **42× fewer samples.** A 109M good-view JEPA reaches with 21.6M samples (.109) the alignment next-token prediction reaches with 900M (.111) on the same corpus.
- **Beats much larger language models.** Trained with 900M samples on texts of 940M images, the 109M good-view JEPA reaches .184 on WIT-1024 against Qwen2.5-72B (.154), [[llama-3|Llama-3]]-70B (.151) and [[clip|CLIP]]'s text encoder (.124). With CLIP ViT-H/14 as the vision side it scores .230 against .210 for Qwen2.5-72B.
- **A scaling law of alignment.** Flickr R@1 follows a law in the fraction q of good views and samples seen M. Fitted on 45 runs, it predicted 85 new runs with RMSE .0093, against .0096 between seeds. It fails further out for good views (overestimating at 320M samples) and has no term for repeated data.
- **A captioner with one fitted map.** Frozen DINOv2/DINOv3 encoders, one linear map with orthonormal rows fitted on 30k Flickr pairs, and a decoder trained on text alone give CIDEr .838 on COCO. A rotation transfers to COCO better than an unconstrained linear map or an MLP fitted on the same pairs. The same captioner describes MSR-VTT videos (CIDEr .246) from averaged frame embeddings.
- **Alignment with no pairs.** Using the method of Schnaus et al. (2026), unchanged, the 109M JEPA reaches 64.4% top-5 zero-shot on ImageNet-100, above every number that paper reports with language models up to Qwen3-8B (at most 56%). Rerun under identical conditions, Qwen3-8B gets 47.6%. Next-token prediction on the same texts fails in half its runs.

## Caveats

- **The theorem's conditions are not met by the experiments.** It assumes population optima over a finite world with injective renderings and matched marginals. Uniqueness up to rotation is proved for second-order anti-collapse, not for the LeJEPA (SIGReg) objective actually trained. The authors say so.
- **It does not cover JEPAs used as [[world-model|world models]].** The analysis uses an identity predictor; learned or action-conditioned predictors are explicitly outside it.
- **Large-scale wins rely on captioner-written text.** Two of each Recap-DataComp image's three texts are written by LLaVA (LLaMA-3-8B with a CLIP ViT-L/14 vision encoder), so image–text training leaks in through the data. Without an image captioner, good views reach .060 on WIT against .107 with recaptions. Comparisons *between* views use identical texts and do not depend on this.
- **Tuning and seeds favour the JEPA.** Hyperparameters were tuned only for the good-view JEPA. A fourth seed of the 109M run reached .121; the four-seed mean of .168 still leads every language model.
- **The 42× figure is approximate.** The two runs use different recipes, and only one next-token checkpoint (900M samples) was scored.
- **Some baselines still win.** CLIP's text encoder is better on FOSCTTM and ImageNet-100 in the pair-free test. CapDec and DeCap reach higher COCO CIDEr (.918, .912), though they rely on CLIP encoders trained on pairs.
- **The image side is mostly bad-view.** DINOv2 and the authors' image JEPAs use crops of one photo as positives. Good image views (video frames) were tried only on MSR-VTT, at small scale.
- **Possible interest.** Both authors are affiliated with AMI Labs, a startup built around JEPAs, and also wrote LeJEPA, the objective used here. This is the wiki's observation, not a concern the paper raises.

## Relation to other wiki pages

- [[learn-from-your-own-latents-korchinski-2026]] argues from a sample-complexity theory that predicting latents is far more data-efficient than predicting tokens. This paper reaches a similar conclusion about JEPAs versus next-token prediction from a different angle: alignment across modalities rather than sample complexity on a grammar.
- [[word-embeddings]]: the paper starts from the old result that word embeddings trained separately on different languages are related by a rotation that can be found without parallel data. Its pair-free alignment is the cross-modal version of that.
- [[scaling-laws]]: it adds a scaling law for cross-modal alignment rather than loss.
- [[linear-representation-hypothesis]] is a related, within-model claim that concepts are linear directions; the PRH is a claim about agreement between models.

## Concepts & entities

- [[platonic-representation-hypothesis]] · [[jepa]] · [[world-model]]
- [[yann-lecun]] · [[dinov2]] · [[clip]] · [[vit]]
- [[qwen-2-5]] · [[llama-3]] · [[bert]]
- [[scaling-laws]] · [[word-embeddings]] · [[multimodal-llm]]

## References

- [arXiv:2610.39302](https://arxiv.org/abs/2610.39302)
- Huh, Cheung, Wang and Isola, "Position: The Platonic Representation Hypothesis", ICML 2024 ([arXiv:2405.07987](https://arxiv.org/abs/2405.07987)), the hypothesis being proved
- Balestriero and LeCun, "LeJEPA" ([arXiv:2511.08544](https://arxiv.org/abs/2511.08544)), the training objective
- Schnaus et al., "Shared geometry as a rosetta stone" ([arXiv:2610.09411](https://arxiv.org/abs/2610.09411)), the pair-free alignment method
