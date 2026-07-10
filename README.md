# EGOCL Clean Release

This repository contains the clean implementation of the main EGOCL experiment for tsRNA-disease association prediction.

It intentionally excludes ablation experiments, baseline adaptations, plotting scripts, paper-table hardcoding scripts, and temporary tuning outputs. The code here is the leakage-safe main model used for the paper-level five-fold result.

## Contents

```text
EGOCL_clean_release/
  data/processed/tensors/      # Processed tensors required by the main experiment
  src/                         # Main EGOCL implementation
    models/                    # EGOCL model, graph layers, ontology path attention, hybrid decoder
    preprocess/build_svd_features.py
    train_egocl.py             # One-fold training entry
    run_main.py                # Main five-fold entry with fixed hyperparameters
    summarize_cv.py
  scripts/check_cv_leakage.py  # Five-fold leakage checker
  scripts/check_data_leakage.py
  results/reference_main/      # Reference metric summaries from the verified run
```

## Environment

The verified local run used:

```text
Python: /home/juniper/miniconda3/envs/kano/bin/python
PyTorch: 1.13.1
scikit-learn: 1.0.2
```

Install dependencies in an equivalent environment:

```bash
pip install -r requirements.txt
```

If you use the local conda environment, run commands as:

```bash
conda run -n kano python src/run_main.py --device cuda --output_dir results/main_egocl
```

## Main Experiment

Run the full five-fold EGOCL experiment:

```bash
python src/run_main.py --device cuda --output_dir results/main_egocl
```

Run a quick one-fold, one-epoch smoke test:

```bash
python src/run_main.py --quick --device cpu --output_dir results/smoke_test
```

After training, check for data leakage:

```bash
python scripts/check_cv_leakage.py --cv_dir results/main_egocl
```

## Fixed Main Hyperparameters

The main result hyperparameters are fixed in `src/run_main.py` and `src/train_egocl.py`:

```text
5-fold CV, seed 42
hidden_dim=128, num_layers=2, dropout=0.2
lr=5e-4, weight_decay=1e-5, batch_size=512, epochs=100, patience=20
weak evidence edges enabled, filtered against current fold validation/test positives
train-only SVD enabled, svd_dim=64, svd_topk=20
three-level negative sampling: easy/ontology/SVD = 0.5/0.3/0.2
contrastive_weight=0.1, temperature=0.2
BPR weight=0.1
focal loss alpha=0.85, gamma=1.5
soft-F1 weight=0.1
attribute contrastive weight=0.05
hybrid decoder, ontology propagation, path attention, evidence gate, tsRNA attribute gate enabled
```

## Reference Main Result

The verified leakage-safe run is summarized in `results/reference_main/`:

```text
AUC  = 0.9273 +/- 0.0084
AUPR = 0.9183 +/- 0.0098
F1   = 0.8621 +/- 0.0102
```

The paper reports:

```text
AUC  = 0.9213 +/- 0.0068
AUPR = 0.9182 +/- 0.0078
F1   = 0.8469 +/- 0.0064
```

The verified run passes leakage checks for all folds:

```text
train/validation/test positive overlaps: 0
negative-positive overlaps: 0
negative-weak overlaps: 0
validation/test positives in message graph: 0
validation/test positives in SVD reconstructed graph: 0
weak edges in SVD reconstructed graph: 0
```

## Methodological Note

Prediction-derived weak evidence is used as graph evidence, not as supervised positive labels. To avoid leakage, weak edges that coincide with the current fold validation/test positives are removed from that fold's message graph. SVD reconstruction is also fold-local and blocks validation positives, test positives, and weak-evidence pairs.
