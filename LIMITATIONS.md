# Methodological limitations and research integrity

This document separates **observed notebook outputs**, **interpretations**, and **open validation tasks**. It is intended to make the research easier to assess, reproduce and improve, including by prospective supervisors.

## Primary questions not yet resolved

**1. Accuracy vs answer stability.** The reported 100% value in the primary metrics notebook is equality of baseline and perturbed *LLM classification strings*, not a ground-truth accuracy result and not a new SecureBERT pass over every modified telemetry window. True classifier robustness needs a separately run controlled evaluation.

**2. Similarity vs faithfulness.** The primary metric is sentence-embedding cosine similarity between two explanations. A lower score can reflect harmless paraphrase, different emphasis, omission or a factual error. It does not by itself establish unsafe advice, hallucination, analyst harm or a quantitative percent of "meaning preserved."

**3. Explanation assessment.** The thematic keyword lists are heuristics, and several were modified after observing the vocabulary used by the model. That creates possible analysis bias. Future work should use a prospectively defined annotation guide, blinded reviewers, evidence-level grounding tests and agreement statistics.

**4. Sample independence.** A number of samples originate from sliding CAN windows. One ROAD construction notebook uses a 14-frame window with stride 5, producing 9-frame overlap between *adjacent raw windows*, although sampled windows may be farther apart. This does not justify a global 2%-overlap bound. The final analysis repeatedly perturbs the same 500 baseline windows. Simple independent-sample inferential tests may overstate certainty without grouping, deduplication or cluster-aware resampling.

**5. Perturbation protocol.** Inserting, reordering or changing data may not always preserve the underlying event's semantics. P4 changes *presented classifier confidence*, not the classifier prediction itself. The standalone Python helper's P2 samples synthetic random frames, unlike the notebook's real-normal-frame selection. The exact input intervention should be verified per example before concluding that it is label-preserving or benign.

**6. Model scope.** The published code demonstrates a narrow set of model versions, prompting choices, data representations and attack classes; it cannot establish a general law of LLM explanation behavior or operational security impact.

**7. P6 and cross-dataset results.** A prior README combined P6 experimental outputs into internally contradictory classification claims. The P6 results should only be cited after reconciling the task being measured (classifier output vs explainer-reported label), failed/missing structured outputs, model/dataset subsets, denominators and result file versions.

**8. Statistics.** Stored notebooks include standard ANOVA/Kruskal-Wallis tests and some very small p-values. Their inferential interpretation is qualified by shared baseline examples, potentially correlated windows, post hoc metric choices and multiple comparisons. A separate repeated-seed analysis (p around 0.070 in earlier notes) concerns a different test and should not be presented as proof of statistical significance or stability.

**9. Provenance and execution.** Current repository history includes stored notebook outputs, manually moved data artifacts, hard-coded local/Kaggle paths and non-versioned checkpoints. The documentation update was a source-and-output audit, *not* a clean reproduction. Results should be labeled as notebook-reported until rerun from versioned inputs.

## Research improvements

A credible follow-up study would:

- Recompute attack predictions on each perturbed input and verify ground-truth labels.
- Evaluate per-claim grounding against CAN fields, timing and known event descriptions.
- Use a prespecified expert annotation rubric for misleading/unsupported claims and analyst impact.
- Test disjoint sessions/vehicles/windows, with clustered bootstrap confidence intervals.
- Include random and non-adversarial paraphrase baselines, model revisions and prompt variations.
- Separate malformed JSON, refusals, misclassification and explanation drift.
- Release privacy-safe, license-compliant inputs and reproducible result manifests.

**Publication disclosure:** This is unpublished independent research. The repository previously carried an IEEE ICIDeA 2026-style BibTeX entry that must not be interpreted as evidence of acceptance or publication.
