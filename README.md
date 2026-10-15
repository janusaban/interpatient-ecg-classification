# interpatient-ecg-classification

Inter-patient heartbeat classification on the MIT-BIH Arrhythmia Database,
comparing how **RR timing and context features** change class-specific
performance for four model families, with **record-level uncertainty** and
**per-record error accounting**.

This repository accompanies:

> E. V. Alforja and J. C. Aban, "Timing and Context Features in Machine
> Learning Classification of Inter-Patient ECG Heartbeats on the MIT-BIH
> Benchmark," medRxiv (2026). https://doi.org/10.64898/2026.10.04.26364683

## Study design

- **Data:** 44 non-paced MIT-BIH recordings, split by the inter-patient
  protocol of de Chazal et al. into DS1 (development, 50,992 beats) and DS2
  (evaluation, 49,686 beats), with no shared recordings.
- **Classes:** N (non-ectopic), S (supraventricular ectopic), V (ventricular
  ectopic), and F (fusion).
- **Beats:** 255-sample windows around each annotated beat from lead MLII,
  band-pass filtered from 0.5 to 40 Hz and standardized.
- **Cumulative input stages:**

  | Stage | Added input | Total inputs |
  |---|---|---|
  | S1 | Beat waveform only | 255 |
  | S2 | 6 RR interval features | 261 |
  | S3 | 7 rhythm history and waveform context features | 268 |
  | S4 | 8 waveform descriptors | 276 |

- **Models:** Logistic Regression, Random Forest, XGBoost, and a
  one-dimensional convolutional neural network (PyTorch).
- **Model selection:** five record-grouped DS1 folds, reused for every model
  and stage. DS2 labels were never used for fitting or selection.
- **Uncertainty:** paired bootstrap over the 22 DS2 recordings (5,000
  resamples), giving 95% intervals for macro F1 and for differences between
  conditions.

## Key results

- Adding six RR features raised DS2 macro F1 by **0.025 to 0.093** for every
  model family, with bootstrap intervals excluding zero.
- At the final stage, XGBoost and the CNN reached macro F1 of **0.580** and
  **0.574**. The interval for their difference included zero, so the
  evaluation set cannot rank them.
- **High accuracy hid rare-class failure:** the most accurate pipeline, Random
  Forest at S2 (**94.45%** accuracy), detected only **26 of 1,837**
  supraventricular beats.
- **Minority results rest on single recordings:** record 232 holds 75% of DS2
  supraventricular beats, and record 213 supplied 203 of the 207 correct CNN
  fusion detections.

## Notebooks

| Notebook | Contents |
|---|---|
| `01_data_exploration.ipynb` | MIT-BIH records, DS1 and DS2 partitions, class distribution, and concentration of minority classes in single recordings |
| `02_preprocessing.ipynb` | Filtering, beat windowing, standardization, and record-grouped fold construction |
| `03_feature_engineering.ipynb` | RR timing, rhythm history, waveform context, and waveform descriptor features (stages S2 to S4) |
| `04_baseline_models.ipynb` | Logistic Regression, Random Forest, and XGBoost across all stages |
| `05_deep_learning.ipynb` | Convolutional network across all stages and per-class evaluation |

Run the notebooks in numerical order, since later notebooks use outputs of
earlier ones.

## Data

The data are **not included** in this repository. Download the MIT-BIH
Arrhythmia Database (version 1.0.0) from PhysioNet:

https://physionet.org/content/mitdb/1.0.0/

Records 102, 104, 107, and 217 contain paced beats and are excluded.

## Requirements

Python 3 with `numpy`, `scipy`, `pandas`, `scikit-learn`, `xgboost`, `torch`,
and `matplotlib`, plus a reader for PhysioNet records such as `wfdb`.

## Reproducibility

Classical models use random seed 42. CNN fold training uses seed 42 + k for
fold k, and final training uses seed 42. Exact software versions were not
archived with the reported results, so small numerical differences across
environments are possible.

## Scope and limitations

- Retrospective, offline analysis using reference beat annotations;
  real-time or wearable performance was not evaluated.
- One database, a single training seed per condition, and only 22 evaluation
  recordings.
- Records 201 and 202 come from the same subject, so the split is record
  disjoint rather than strictly subject disjoint.

## License

MIT
