# Memecoin Launch Trace Dataset

This repository contains the dataset and code for the submission: **"MELT: A Behavioral Trace Dataset for High-Risk Memecoin Launch Detection"**

## Environment

- Python 3.9
- Install packages using:
``` sh
pip install -r requirements.txt
```

## Part 1: Feature Generation

### Step 1: Download Dataset

Since the raw transaction data is very huge (>1TB), we only provide the parsed transaction datasets on Google Drive:

- [pre_migration_tx.zip](https://drive.google.com/file/d/1rqzkaVoc1FG8XRp-hBxECjiG_PxAn5Cp/view?usp=drive_link) — Pre-migration (bonding curve) transactions. **Required** for feature generation. Download and unzip under `MELT/data/tx/` (so the parsed transactions sit under `data/tx/...`).
- [bundle.zip](https://drive.google.com/file/d/1TdSNm6afU39lW6nk7FnM24gQyOReJy1R/view?usp=sharing) — Bundle trace data. **Required** for feature generation. Download and unzip under `MELT/data/` (expands into `data/bundle/`).
- [post_migration_tx.zip](https://drive.google.com/file/d/11giAd68Mp_ZyTRgKwPVxeOOP5J5Gb_7f/view?usp=drive_link) — Post-migration (Raydium DEX) transactions. **Optional**, very large, **not used by feature generation**. Only download if you want to do your own post-migration analysis.

You can skip Step 2 and download our pre-generated [feature.pkl](https://drive.google.com/file/d/1B9mHhXppgOw33yMSdTMOMXJyMaMjCOzf/view?usp=drive_link) directly. Place it at `MELT/data/feat/feature.pkl` and proceed to Part 2. In this case neither `pre_migration_tx.zip` nor `bundle.zip` is needed.


### Step 2: Feature Generation

``` sh
cd MELT/src
python feat_gen.py
```

This generates `data/feat/feature.pkl` from the pre-migration transactions, bundle trace data, and contextual information.

## Part 2: High-risk Launch Detection

### Step 1: Train a model

``` sh
cd MELT/src
python train.py --model rf
```

`--model` accepts any of: `rf`, `xgb`, `lgbm`, `lr`, `mlp`, or `tcn`, `lstm`, `gru`, `transformer` (time-series models). Prediction CSVs are written to `MELT/results/{model}_pred_*.csv`.

Common flags:

| flag | default | applies to                                                                                                     |
|---|---|----------------------------------------------------------------------------------------------------------------|
| `--model` | `xgb` | all |
| `--epochs` | 20 | DNN models |
| `--batch_size` | 256 | DNN models |
| `--lr` | 1e-3 | DNN models |
| `--seed` | 42 | all (Python `random`, numpy, torch, sklearn `random_state`, DataLoader shuffle) |

`train.py` only reports **AUPRC** (threshold-free) and dumps per-run prediction probabilities to `results/`. Threshold-based metrics (precision / recall / F1) and ensembling are done in the next step.

### Step 2: Evaluate predictions

`evaluate.py` reads a prediction CSV from Step 1 and prints AUPRC plus a `classification_report` at one or more probability thresholds.

``` sh
# single threshold
python evaluate.py --csv lgbm_pred_0.559999.csv --thresholds 0.5

# multi-threshold sweep
python evaluate.py --csv lgbm_pred_0.559999.csv --thresholds 0.3 0.4 0.5 0.6
```

| flag | default | role |
|---|---|---|
| `--csv` | — | prediction CSV (relative paths resolve against `results/`) |
| `--thresholds` | `[0.49]` | one or more probability cutoffs; one `classification_report` per threshold |

## `src/`

| file | role                                                                                                      |
|---|-------------------------------------------------------------------------------------------------------------|
| `feat_gen.py` | Generates `data/feat/feature.pkl` from parsed transactions, bundle traces, and contextual info. |
| `dataset.py` | Data loading & preprocessing. Reads `feature.pkl` + label CSV, merges by `mint_address`, splits & scales. Exposes `load_dataset()`, `TSDataset`, `ts_collate`. |
| `model.py` | All model definitions and factories (sklearn baselines + MLP / TS deep models). |
| `train.py` | Training entry point. Argparse-driven; reports AUPRC and writes per-run prediction CSVs to `results/`. |
| `evaluate.py` | Evaluates prediction CSVs from `train.py` at one or more thresholds; supports weighted ensembling of multiple CSVs. |

## License

This project is released under the [Creative Commons Attribution-NonCommercial 4.0 International (CC BY-NC 4.0)](https://creativecommons.org/licenses/by-nc/4.0/) license. See [LICENSE](LICENSE) for the full text.

You are free to use, share, and adapt the material for non-commercial purposes, provided you give appropriate credit (please cite our paper). Commercial use requires separate permission from the authors.




