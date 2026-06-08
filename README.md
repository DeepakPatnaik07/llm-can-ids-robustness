# LLM-based CAN Bus IDS Explanation Robustness Pipeline

**Paper:** LLM-based CAN Bus IDS Explanation Robustness Pipeline  
**Venue:** IEEE ICIDeA 2026  
**Author:** Deepak Patnaik  
**Mentor:** Swagat Das (VCU PhD, Transformer-based CAN anomaly detection)

---

## Research Question

> How consistent are LLM-based CAN IDS explanations under lightweight adversarial perturbations, even when attack classification remains correct?

---

## Architecture

    CAN Bus Telemetry
          |
          v
    SecureBERT (LoRA fine-tuned, 125M params)
          |  classification + confidence
          v
    LLM Explainer
      - Qwen2.5-1.5B-Instruct  (primary)
      - Mistral-7B-Instruct-v0.2  (comparison)
          |
          v
    Explanation Drift Analysis

---

## Datasets

| Dataset | Classes | Windows |
|---|---|---|
| HCRL Car-Hacking | NORMAL, DOS, FUZZY, GEAR, RPM | 500 per class |
| ROAD | AMBIENT, ACCELERATOR, FUZZING, SPEEDOMETER, CORRELATED | 250 per class |

---

## Perturbation Types

| ID | Name | Layer | Description |
|---|---|---|---|
| P1 | Payload Mutation | Data | One byte mutated by plus or minus 1 in middle frame |
| P2 | Benign Frame Insertion | Context | One ambient frame inserted into window |
| P3 | Frame Reorder | Temporal | Two adjacent frames swapped |
| P4 | Confidence Reduction | Prompt | Classifier confidence reduced from 1.000 to 0.623 |
| P5 | Semantic Steering | Prompt | One steering sentence added to prompt |
| P6 | False Context Injection | Prompt | Plausible benign maintenance narrative injected |

---

## Key Findings

- 49.1% of lightweight perturbations cause operationally harmful explanation drift
- SHAP dissociation confirmed: classifier SHAP similarity 0.995-1.000 vs explanation similarity 0.748-0.753
- Attack class explains 49% of variance in drift; perturbation type explains only 3.7%
- P6 false context injection caused Mistral-7B to reclassify 100% of DOS attacks as NORMAL
- Qwen-1.5B showed output collapse on DOS payloads under P6; Mistral-7B showed semantic vulnerability
- Cross-dataset validation on ROAD confirmed prompt-layer vulnerability transcends dataset complexity
- Seed stability confirmed (ANOVA p=0.070, spread=0.028)
- Deterministic generation (do_sample=False) — zero stochastic variance

---

## Models

| Model | Role | Parameters |
|---|---|---|
| SecureBERT (LoRA) | Classifier | 125M |
| Qwen2.5-1.5B-Instruct | Primary explainer | 1.5B |
| Mistral-7B-Instruct-v0.2 | Comparison explainer | 7B |
| TinyLlama-1.1B | Negative baseline | 1.1B |

---

## Repository Structure

    llm-can-ids-robustness/
        README.md
        requirements.txt
        .gitignore
        notebooks/
            00_environment_check.ipynb
            01_section3_dataset_loading.ipynb
            02_section4_schema_standardization.ipynb
            03_section5_data_cleaning.ipynb
            04_section6_window_generation.ipynb
            05_section7_telemetry_text.ipynb
            06_section8_benchmark.ipynb
            07_section9a_securebert_finetune.ipynb
            08_section10_perturbations.ipynb
            09_section11_metrics.ipynb
            10_task1_shap_baseline.ipynb
            11_seed_dataset_creation.ipynb
            11_task3_grounding_check.ipynb
            12_task4_road_experiment.ipynb
        kaggle/
            kaggle_inference_notebook.ipynb
        src/
            perturbations.py
            generation.py
            evaluation.py
            visualization.py

---

## Compute

- **Local:** M2 MacBook Air (8GB, MPS) — analysis, SHAP, figures
- **GPU inference:** Kaggle (Tesla T4 x2) — LLM generation
- **Kaggle dataset:** [deepakpatnaik07/icidea-llm-ids-benchmark](https://www.kaggle.com/datasets/deepakpatnaik07/icidea-llm-ids-benchmark)

---

## Results Summary

### HCRL Qwen — Per-Perturbation Cosine Similarity

| Perturbation | Mean | 95% CI | Cohen's d |
|---|---|---|---|
| P1 Payload | 0.738 | [0.722, 0.753] | 2.155 large |
| P2 Insertion | 0.785 | [0.776, 0.794] | — |
| P3 Reorder | 0.801 | [0.793, 0.808] | — |
| P4 Confidence | 0.776 | [0.766, 0.786] | — |
| P5 Steering | 0.753 | [0.741, 0.766] | 2.543 large |

### P6 False Context — Classification Accuracy

| Model | Dataset | DOS accuracy | Other classes |
|---|---|---|---|
| Qwen-1.5B | HCRL | Output collapse | 96-100% |
| Mistral-7B | HCRL | 0% reclassified as NORMAL | 96-100% |
| Qwen-1.5B | ROAD | 20% overall | — |
| Mistral-7B | ROAD | 40% overall | — |

---

## Citation

    @inproceedings{patnaik2026llm,
      title={LLM-based CAN Bus IDS Explanation Robustness Pipeline},
      author={Patnaik, Deepak},
      booktitle={IEEE ICIDeA 2026},
      year={2026}
    }
