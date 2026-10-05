# From Silence to Surge: Modeling Zero-Inflated Dengue Outbreaks in Bangladesh

Code and data for the paper accepted at the **12th IEEE International Women in Engineering (WIE) Conference on Electrical and Computer Engineering (WIECON-ECE 2026)**.

**Paper:** Link to be added after publication (IEEE Xplore / DOI)

**Authors:** Md. Maruf Bangabashi, Md. Mostafijur Rahman, Jihadul Islam Rifat, Jhoton Chandra Nath, Nusrat Sorna and Most Jannatun Ferdous Jim. *Department of Computer Science and Engineering, Dhaka International University, Dhaka, Bangladesh.*

## Overview

Daily dengue counts in Bangladesh are zero-inflated and heavily overdispersed. In our data, 42.3% of division-days report no cases, and the variance is 472 times the mean. This repository benchmarks ten models for daily, division-level dengue forecasting under one feature set and one leakage-safe walk-forward protocol:

- **Seven point-estimate regressors:** Linear Regression, Decision Tree, Random Forest, XGBoost, LightGBM, CatBoost and an MLP.
- **Three zero-inflation-aware models:** XGBoost with a Tweedie objective, LightGBM with a Tweedie objective, and a two-stage hurdle model.

The main findings are:

- Tweedie-objective XGBoost has the lowest overall MAE (8.71 patients/day against 8.97 for Random Forest).
- Random Forest has the lowest MAE on outbreak days (37.13).
- No zero-inflation-aware model differs significantly from Random Forest across the three folds.
- SMAPE mainly measures how a model handles zero days, so it should not rank models on this target.

## Dataset

`Dataset.csv` holds one row per division and day: 8 divisions, 1,400 days each (1 January 2022 to 31 October 2025), 11,200 rows in total.

| Column | Description |
|---|---|
| `date` | Calendar date |
| `division` | One of Barishal, Chattogram, Dhaka, Khulna, Mymensingh, Rajshahi, Rangpur, Sylhet |
| `Patients` | Reported dengue patients that day |
| `max temp`, `min temp` | Daily temperature extremes |
| `rainfall` | Daily rainfall |
| `humidity` | Relative humidity |

The first 14 days of each division have no full rolling window and are dropped, so the models use 11,088 division-days.

## Method in brief

- **Features (13):** division code, year, month, ISO week, day of year, the four weather variables, the 1-day and 7-day lags of `Patients`, and the 7-day and 14-day rolling means. Every lag feature uses only past values of the same division.
- **Validation:** expanding-window walk-forward cross-validation with 3 folds, computed per division and then pooled. Fold *k* (k = 0, 1, 2) trains on the first (60 + 10k)% of each division and tests on the next 10%.
- **Metrics:** MAE overall, MAE on outbreak days (the top decile of each test fold) and SMAPE. SMAPE counts a term with y = ŷ = 0 as zero, and all predictions are clipped at zero.
- **Significance:** paired t-tests across the three folds, comparing each zero-inflation-aware model with Random Forest.
- **Seed:** 42 for every model.

## Repository contents

| File | Purpose |
|---|---|
| `Dengue_Zero_Inflation_Benchmark.ipynb` | Main notebook. It generates Tables III to V and Figs. 4 and 5 and checks them against the numbers printed in the paper. |
| `Dataset.csv` | The daily division-level dataset. |

## How to run

1. Install the pinned versions:

   ```bash
   pip install numpy==2.5.3 pandas==3.0.5 scipy==1.18.1 scikit-learn==1.9.1 xgboost==3.1.3 lightgbm==4.7.0 catboost==1.2.10 jupyter
   ```

2. Put `Dataset.csv` next to the notebook, or set the `DENGUE_CSV` environment variable to its path.
3. Open `Dengue_Zero_Inflation_Benchmark.ipynb` and run all cells. It takes about 30 seconds on a laptop CPU.

To run the scripts instead, run `python experiment_cv_reproducible.py`, then `python analysis_extra.py`. The first script saves the per-fold predictions that the second one reads.

**The XGBoost version matters.** The two XGBoost models give different results in other releases. For example, XGBoost-Tweedie reaches 8.71 to 8.93 overall MAE and 37.95 to 39.54 outbreak-day MAE across XGBoost 1.7 to 3.4. The other eight models do not change. The paper reports XGBoost 3.1.3, with Python 3.13.

## Main results (Table III of the paper)

Mean ± standard deviation over the three folds.

| Model | Overall MAE | Outbreak-day MAE | SMAPE (%) |
|---|---|---|---|
| Linear Regression | 9.84 ± 6.50 | 39.67 ± 30.74 | 47.6 |
| Decision Tree | 11.07 ± 8.95 | 52.75 ± 46.76 | 54.5 |
| Random Forest | 8.97 ± 6.53 | **37.13 ± 28.72** | 52.7 |
| XGBoost | 9.52 ± 7.57 | 45.12 ± 39.24 | 52.5 |
| LightGBM | 9.01 ± 6.81 | 40.03 ± 32.25 | 51.8 |
| CatBoost | 9.29 ± 6.64 | 38.24 ± 29.98 | 51.6 |
| ANN/MLP | 10.50 ± 7.88 | 41.47 ± 33.17 | 45.9 |
| XGBoost-Tweedie | **8.71 ± 6.52** | 37.95 ± 29.50 | 53.2 |
| LightGBM-Tweedie | 8.83 ± 6.64 | 37.76 ± 29.07 | 53.1 |
| Hurdle (2-stage) | 9.10 ± 7.18 | 42.75 ± 35.46 | 53.3 |

## Preliminary models

The paper also reports eight earlier models (ANN, LSTM, XGBoost, Random Forest, Linear Regression, CatBoost, Decision Tree and LightGBM). They use different tasks, features and splits, so the paper does not compare them with the benchmark above. They are not part of this notebook.

## Citation

If you use this code or data, please cite the paper:

```bibtex
@inproceedings{bangabashi2026silence,
  title     = {From Silence to Surge: Modeling Zero-Inflated Dengue Outbreaks in Bangladesh},
  author    = {Bangabashi, Md. Maruf and Rahman, Md. Mostafijur and Rifat, Jihadul Islam and Nath, Jhoton Chandra and Sorna, Nusrat and Jim, Most Jannatun Ferdous},
  booktitle = {Proc. 12th IEEE International Women in Engineering (WIE) Conference on Electrical and Computer Engineering (WIECON-ECE)},
  year      = {2026}
}
```

## Contact

Md. Maruf Bangabashi, marufbangabashi@gmail.com
