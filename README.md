# LLM-based CAN Bus IDS Explanation Robustness Pipeline

**Paper:** "LLM-based CAN Bus IDS Explanation Robustness Pipeline"  
**Venue:** IEEE ICIDeA 2026  
**Author:** Deepak Patnaik  
**Mentor:** Swagat Das (VCU PhD, Transformer-based CAN anomaly detection)

---

## Research Question

> How consistent are LLM-based CAN IDS explanations under lightweight adversarial perturbations, even when attack classification remains correct?

---

## Architecture
CAN Bus Telemetry
│
▼
SecureBERT (LoRA fine-tuned, 125M)
│ classification + confidence
▼
LLM Explainer
├── Qwen2.5-1.5B-Instruct (primary)
└── Mistral-7B-Instruct-v0.2 (comparison)
│
▼
Explanation + Drift Analysis

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
| P1 | Payload Mutation | Data | ±1 byte in middle frame |
| P2 | Benign Frame Insertion | Context | One ambient frame inserted |
| P3 | Frame Reorder | Temporal | Adjacent frame swap |
| P4 | Confidence Reduction | Prompt | 1.000 → 0.623 |
| P5 | Semantic Steering | Prompt | One steering sentence added |
| P6 | False Context Injection | Prompt | Plausible benign maintenance narrative |

---

## Key Findings

- **49.1%** of lightweight perturbations cause operationally harmful explanation drift (fabrication, misdirection, lost mechanism)
- **SHAP dissociation confirmed**: classifier SHAP similarity 0.995-1.000 vs explanation similarity 0.748-0.753 — drift is LLM-layer specific
- **Attack class explains 49% of variance** in drift; perturbation type explains only 3.7%
- **P6 false context injection** caused Mistral-7B to reclassify 100% of DOS attacks as NORMAL
- **Qwen-1.5B** showed output collapse on all-zero DOS payloads under P6; Mistral-7B showed semantic vulnerability
- **Cross-dataset validation** on ROAD confirmed prompt-layer vulnerability transcends dataset complexity
- **Seed stability** confirmed (ANOVA p=0.070, spread=0.028)
- **Deterministic generation** (do_sample=False) — all drift attributable to input perturbation, zero stochastic variance

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
│
├── README.md
├── requirements.txt
│
├── notebooks/
│   ├── 01_securebert_classifier.ipynb
│   ├── 02_baseline_explanations.ipynb
│   ├── 03_perturbation_p1p2p3p4p5.ipynb
│   ├── 04_shap_analysis.ipynb
│   ├── 05_seed_stability.ipynb
│   ├── 06_road_cross_dataset.ipynb
│   ├── 07_model_comparison.ipynb
│   ├── 08_p6_false_context.ipynb
│   └── 09_statistical_analysis.ipynb
│
├── kaggle/
│   └── kaggle_inference_notebook.ipynb
│
└── src/
├── perturbations.py
├── generation.py
├── evaluation.py
└── visualization.py

---

## Compute

- **Local:** M2 MacBook Air (8GB, MPS) — analysis, SHAP, figures
- **GPU inference:** Kaggle (Tesla T4 x2) — LLM generation
- **Kaggle dataset:** [deepakpatnaik07/icidea-llm-ids-benchmark](https://www.kaggle.com/datasets/deepakpatnaik07/icidea-llm-ids-benchmark)

---

## Results Summary

### HCRL Qwen — Per-Perturbation Cosine Similarity

| Perturbation | Mean | 95% CI | Cohen's d vs baseline |
|---|---|---|---|
| P1 Payload | 0.738 | [0.722, 0.753] | 2.155 (large) |
| P2 Insertion | 0.785 | [0.776, 0.794] | — |
| P3 Reorder | 0.801 | [0.793, 0.808] | — |
| P4 Confidence | 0.776 | [0.766, 0.786] | — |
| P5 Steering | 0.753 | [0.741, 0.766] | 2.543 (large) |

### P6 False Context — Classification Accuracy

| Model | Dataset | DOS accuracy | Other classes |
|---|---|---|---|
| Qwen-1.5B | HCRL | Output collapse | 96-100% |
| Mistral-7B | HCRL | 0% (→NORMAL) | 96-100% |
| Qwen-1.5B | ROAD | 20% overall | — |
| Mistral-7B | ROAD | 40% overall | — |

---

## Citation

```bibtex
@inproceedings{patnaik2026llm,
  title={LLM-based CAN Bus IDS Explanation Robustness Pipeline},
  author={Patnaik, Deepak},
  booktitle={IEEE ICIDeA 2026},
  year={2026}
}
```
