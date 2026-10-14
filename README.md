# LLM Explanation Robustness for CAN Intrusion Detection

**Status:** Independent research prototype / working manuscript — **unpublished**. No peer-reviewed acceptance or conference publication is claimed.  
**Researcher:** Deepak Patnaik  
**Research question:** Can an intrusion-detection label remain stable while an LLM-generated explanation changes under small telemetry or prompt perturbations?

This repository documents experiments on explanation consistency using automotive CAN-bus telemetry, a SecureBERT-based classifier and downstream language-model explainers. It is **not** a deployed intrusion-detection product. Read the [reproduction guide](REPRODUCIBILITY.md) and [methodological limitations](LIMITATIONS.md) before using the results.

> **Evidence status:** Numerical results below are transcribed from *stored notebook outputs*. They were not independently rerun or validated during this documentation update. Embedding similarity **does not** establish factual correctness or operational harm.

## Experimental design

- **Datasets:** HCRL Car-Hacking (NORMAL, DOS, FUZZY, GEAR, RPM) and ROAD (AMBIENT, ACCELERATOR, FUZZING, SPEEDOMETER, CORRELATED).
- **Input:** CAN telemetry serialized in 14-frame windows.
- **Models:** SecureBERT with a LoRA-based classification setup; Qwen2.5-1.5B-Instruct for the principal explanation experiment; exploratory use of Mistral-7B-Instruct-v0.2 and TinyLlama-1.1B.
- **Baseline:** HCRL benchmark: 500 windows/class (2,500 total). Principal explanation analysis: 500 baseline windows (100/class), each receiving P1–P5 (2,500 paired perturbation records). ROAD notebook: 50 windows/class (250 total), **not** 250/class.
- **Metrics:** SentenceTransformer embedding cosine similarity, thematic keyword checks and qualitative error analysis. These are proxies, not validated evidence-faithfulness measures.

| ID | Perturbation | Intervention |
| --- | --- | --- |
| P1 | Payload mutation | Change a byte in a middle CAN frame |
| P2 | Benign frame insertion | Insert normal traffic, retaining window length |
| P3 | Frame reorder | Swap neighboring frames |
| P4 | Confidence change | Lower the confidence conveyed to the explainer |
| P5 | Semantic steering | Add a topic-directing prompt instruction |
| P6 | False context injection | Add benign-sounding narrative context; **separate exploratory runs** |

The actual principal P1–P5 experiment uses functions in `notebooks/08_section10_perturbations.ipynb`. The standalone `src/perturbations.py` is **not behaviorally equivalent**: its P2 inserts a *synthetic randomly generated* frame rather than sampled genuine normal traffic, and its P5 text is generic rather than class-specific. Do not treat it as a drop-in replication of the notebook.

## Recorded HCRL/Qwen results (P1–P5)

The saved output of `notebooks/09_section11_metrics.ipynb` provides these **descriptive** mean similarities between paired explanations:

| Perturbation | Paired records | Mean embedding cosine similarity |
| --- | ---: | ---: |
| P1: Payload mutation | 500 | 0.738 |
| P2: Benign insertion | 500 | 0.785 |
| P3: Frame reorder | 500 | 0.801 |
| P4: Confidence change | 500 | 0.776 |
| P5: Semantic steering | 500 | 0.753 |
| **All five (pooled)** | **2,500** | **0.771** |

The notebook's reported "100% prediction stability" is computed as **equality of the LLM's baseline versus perturbed classification strings**. It is **not** a rerun of SecureBERT on every perturbed sample, and it does not establish 100% ground-truth attack-class accuracy. Likewise, a cosine similarity of 0.771 must not be presented as "77.1% of semantic content preserved."

These are paired comparisons reusing baseline windows and potentially temporally related underlying examples; statistical tests that presume all records are independent require further scrutiny. See [LIMITATIONS.md](LIMITATIONS.md).

### Additional, exploratory experiments

Separate notebooks contain SHAP comparisons, repeated-seed checks, heuristic grounding analysis, Mistral runs, ROAD samples and P6 false-context testing. The original README's **49.1% "operationally harmful"** assertion, its **P6 contradictory classification summary**, and definitive explanations of SHAP/model stability are **not retained as verified headline findings**: the repository does not currently provide an auditable, consolidated protocol and underlying outputs to support those statements consistently.

## Code and notebook map

| File or directory | Role |
| --- | --- |
| `notebooks/00_environment_check.ipynb` through `notebooks/07_section9a_securebert_finetune.ipynb` | Data processing, windowing, benchmark, SecureBERT setup |
| `notebooks/08_section10_perturbations.ipynb` | Primary P1–P5 test dataset construction |
| `kaggle/kaggle_inference_notebook.ipynb` | Remote GPU inference and additional explorations |
| `notebooks/09_section11_metrics.ipynb` | Primary explanation-similarity metrics |
| `notebooks/11_seed_dataset_creation.ipynb` | Repeated-seed dataset |
| `notebooks/11_task3_grounding_check.ipynb` | Heuristic grounding checks |
| `notebooks/12_task4_road_experiment.ipynb` | ROAD extraction and sampling |
| `src/perturbations.py` | Limited independent helper functions (not experimental parity) |
| `requirements.txt` | Pinned Python dependencies |

The repository includes additional scratch notebooks. Some listed modules in the former README (`src/generation.py`, `src/evaluation.py` and `src/visualization.py`) **do not exist**; they have been removed from the claimed repository structure.

## Getting started

```bash
git clone https://github.com/DeepakPatnaik07/llm-can-ids-robustness.git
cd llm-can-ids-robustness
python3.11 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
jupyter lab
```

Refer to [REPRODUCIBILITY.md](REPRODUCIBILITY.md) before executing notebooks. They include legacy absolute paths and require datasets, checkpoints and intermediate Parquet files **not stored in Git**. Remote GPU steps reference a [Kaggle dataset](https://www.kaggle.com/datasets/deepakpatnaik07/icidea-llm-ids-benchmark); its present availability, completeness and licensing are **not verified here**. Full end-to-end reproduction is not yet turnkey.

## Research priorities

1. Publish a versioned artifact manifest and independent rerun of the main P1–P5 analysis.
2. Rerun SecureBERT on modified telemetry and explicitly separate **classifier stability** from **LLM answer stability**.
3. Quantify independence/overlap, run grouped statistical analyses and confidence intervals.
4. Develop expert-reviewed factual-grounding and analyst-impact metrics.
5. Validate P6, Mistral and ROAD findings with explicit denominators, error handling and supporting artifacts.

## Attribution and publication status

**Unpublished work in progress.** This repository should not be cited as a published IEEE ICIDeA 2026 proceeding without verified acceptance. Cite the repository URL and commit for any reuse. The original repository credited **Swagat Das** as a mentor; this credit is preserved here, without implying conference acceptance or formal institutional endorsement.
