# EGOCL Clean Release

This repository contains the clean implementation of the main EGOCL experiment for tsRNA-disease association prediction.

It intentionally excludes ablation experiments, baseline adaptations, plotting scripts, paper-table hardcoding scripts, and temporary tuning outputs. The code here is the leakage-safe main model used for the paper-level five-fold result.

## Environment

The verified local run used:
Python
PyTorch: 1.13.1
scikit-learn: 1.0.2

Install dependencies in an equivalent environment:

bash
pip install -r requirements.txt

If you use the local conda environment, run commands as:

bash
conda run -n kano python src/run_main.py --device cuda --output_dir results/main_egocl

## Main Experiment

Run the full five-fold EGOCL experiment:

bash
python src/run_main.py --device cuda --output_dir results/main_egocl


Run a quick one-fold, one-epoch smoke test:

bash
python src/run_main.py --quick --device cpu --output_dir results/smoke_test


After training, check for data leakage:

bash
python scripts/check_cv_leakage.py --cv_dir results/main_egocl
