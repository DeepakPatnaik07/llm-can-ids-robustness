# Reproducibility and artifact guide

**Scope:** Documentation of the existing research workflow. This is not a guarantee that the entire pipeline runs as-is in a fresh environment. The notebooks include local paths, exploratory cells, downloaded datasets and model artifacts that are not checked into Git.

## Environment

- Python 3.11 was used in stored notebook outputs.
- The pinned packages in `requirements.txt` are an approximate historical environment, not a tested lockfile. Some existing notebooks also use `jupyterlab`/`notebook`, `pyarrow` and `ipykernel`, which are not explicitly pinned there.
- A local Apple Silicon environment was used for preprocessing/analysis. Some inference cells use Kaggle GPU and hard-coded `/kaggle/input/...` paths.
- One stored SHAP notebook output reports `shap==0.44.1` although `requirements.txt` pins 0.46.0. Version differences need reconciliation.
- Model checkpoints, Hugging Face models and raw data files may require separate downloads, credentials/terms and substantial compute.

## Suggested exploration setup (not verified end-to-end)

```bash
git clone https://github.com/DeepakPatnaik07/llm-can-ids-robustness.git
cd llm-can-ids-robustness
python3.11 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
python -m pip install jupyterlab ipykernel pyarrow
jupyter lab
```

Do not treat installation success as experimental reproduction. Run only on datasets you have permission to access and keep source data out of Git.

## Layout expected by original notebooks

Several notebooks define `PROJECT_ROOT = Path.home() / "icidea_llm_ids"`. Their local paths expect:

```bash
~/icidea_llm_ids/
    data/         # raw HCRL and ROAD inputs
        ROAD/     # ROAD candump log files, per notebook naming
    artifacts/    # generated .parquet files
    checkpoints/  # local SecureBERT/LoRA checkpoint
    results/      # summary tables and figures
```

Adapt these paths before running in another directory. These directories and binary artifacts are excluded from version control. Notebook outputs show absolute user paths: they are *historical records*, not portable instructions.

## Outline of primary workflow

1. `notebooks/00_environment_check.ipynb`: environment checks.
2. `notebooks/01_section3_dataset_loading.ipynb` through `05_section7_telemetry_text.ipynb`: load, normalize, clean and serialize HCRL data.
3. `notebooks/06_section8_benchmark.ipynb`: create a 2,500-window HCRL benchmark, 500/class, partitioned into test/validation/reserve (300/100/100 per class).
4. `notebooks/07_section9a_securebert_finetune.ipynb` and `notebooks/09a.ipynb`: train/load SecureBERT classifier checkpoint.
5. `kaggle/kaggle_inference_notebook.ipynb`: remote explanation inference. This is an exploratory notebook with multiple stages; **do not run all cells blindly**. Inspect the input paths, model loading, experimental branch and saved outputs first.
6. `notebooks/08_section10_perturbations.ipynb`: assemble 500 baseline explanations and 2,500 HCRL perturbed prompts across P1–P5.
7. The Kaggle inference notebook produces per-perturbation explanation outputs; these were downloaded locally in the original workflow.
8. `notebooks/09_section11_metrics.ipynb`: compare paired explanation strings and tabulate P1–P5 similarities.

The numbering does not encode a perfect runnable dependency DAG; some cell outputs rely on previous local analysis and manual transfers to/from Kaggle.

## Input and output inventory

| Key file (under `artifacts/`) | Documented purpose |
| --- | --- |
| `section7_windows_with_text.parquet` | 14-frame telemetry windows |
| `section8_benchmark.parquet` | Sampled benchmark and partition metadata |
| `section9a_classifier_results.parquet` | Classifier predictions for sampled windows |
| `section9b_baseline_explanations.parquet` | Baseline Qwen explanations (500 records) |
| `section10_perturbed_dataset.parquet` | P1–P5 perturbation prompts (2,500 records) |
| `section10_perturbed_p1p2.parquet` | Inference outputs for P1/P2 (1,000 records) |
| `section10_perturbed_p3p4p5.parquet` | Inference outputs for P3/P4/P5 (1,500 records) |
| `section11_metrics.parquet` | Comparison metrics from the primary analysis |

Counts come from saved notebook outputs, not a newly executed audit. The data artifacts are excluded from Git; verify the [external Kaggle dataset](https://www.kaggle.com/datasets/deepakpatnaik07/icidea-llm-ids-benchmark) manually before claiming full public reproducibility. Record source versions, SHA256 hashes, datasets' terms, checkpoint provenance and model revisions in a future public manifest.

## Comparison semantics

The main metrics notebook defines:

```bash
pred_stable = (row["baseline_classification"] == row["perturbed_classification"])
```

This checks *LLM-reported classification equality*; it does not rerun SecureBERT on changed input. Its embedding cosine comparison uses baseline and perturbed explanations, and its keyword alignment is an unvalidated proxy rather than an annotated correctness metric.

Window construction in a ROAD notebook uses window length 14 and stride 5 (9 frames shared between adjacent raw windows). The sampled-window independence analysis is incomplete; avoid claims of independent observations or a universal maximum-overlap percentage.

## What is required for a verified reproduction release?

- Create a source dataset and file-manifest with licenses, versions, hashes and acquisition instructions.
- Reconcile the model/checkpoint versions, Python package versions and any missing execution steps.
- Separate the original primary P1–P5 protocol from P6 and cross-model exploratory runs.
- Publish de-identified/anonymized result artifacts, where licensing permits.
- Run the pipeline cleanly from source, document expected outputs, failures and hardware; validate numerical tables against saved hashes.
- Independently evaluate ground-truth correctness, effective sample size and factual explanation grounding.

See [LIMITATIONS.md](LIMITATIONS.md) for current interpretive limits.
