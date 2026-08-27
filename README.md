
# Predictive AI Operational Load Forecasting & Capacity Alert System

## 1. Approach & Methodology

- **Data Processing & Leakage Prevention:**
  - Loaded daily hospital operational metrics across 28 days for facility `GMC-AHM-001`.
  - Engineered 6 leakage-free features: `day_of_week`, `is_weekend`, `month`, `lag_1`, `lag_7`, and `rolling_mean_7`.
  - Avoided temporal data leakage by explicitly shifting historical series (`shift(1)`) prior to computing the 7-day rolling average.
  - Cleanly handled missing values from initial lag shifts, leaving 21 chronological observations.

- **Time-Based Splitting & Modeling:**
  - Applied a strict chronological split (70% train, 15% validation, 15% test) without shuffling to preserve time-series ordering.
  - **Baseline:** Seasonal Naive model using the exact same-day-of-week observation from the prior week (`lag_7`).
  - **ML Model:** Regularized XGBoost Regressors (`max_depth=2`, `reg_lambda=1.5`, `n_estimators=30`) trained across all 5 targets in a unified training loop.

- **Forecasting & Alerts:**
  - Generated next 7-day forecasts recursively from `2026-08-29` to `2026-09-04`.
  - Evaluated bed occupancy against the static hospital capacity limit (500 beds) to flag capacity breach risks.

---

## 2. Evaluation Results (Test Set: Aug 25 – Aug 28, 2026)

| Target Metric | Baseline MAE | XGBoost MAE | Baseline RMSE | XGBoost RMSE | Best Performing Model |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **lab_tests** | 21.25 | 57.35 | 21.36 | 69.21 | **Baseline (Seasonal Naive)** |
| **ed_arrivals** | 4.75 | 11.94 | 4.97 | 16.32 | **Baseline (Seasonal Naive)** |
| **opd_visits** | 20.00 | 70.78 | 21.21 | 88.84 | **Baseline (Seasonal Naive)** |
| **beds_occupied** | 2.75 | 5.37 | 2.87 | 7.77 | **Baseline (Seasonal Naive)** |
| **ambulance_calls** | 0.75 | 2.64 | 0.87 | 3.43 | **Baseline (Seasonal Naive)** |

---

## 3. Metric Comparison Summary
**With only 14 training rows, XGBoost could not learn continuous time relationships and over-smoothed the sharp weekly cycles by averaging just 2–3 samples per leaf node.**
- **lab_tests:** The Seasonal Naive baseline outperformed XGBoost because laboratory demand follows an exact weekly cycle that gradient boosting blunted due to having only 14 training rows.
- **ed_arrivals:** The baseline beat the ML model as emergency arrivals repeat standard midweek peaks that were captured directly by 7-day lags without tree approximation errors.
- **opd_visits:** The baseline achieved significantly lower error because outpatient visits exhibit extreme weekday-vs-weekend variance that tree-based step functions over-smoothed on a small sample.
- **beds_occupied:** The baseline edged out XGBoost due to the high auto-correlation and strict 7-day periodicity in inpatient bed turnover.
- **ambulance_calls:** The baseline won over XGBoost as small discrete counts (28–53 calls) are more accurately preserved via direct historical matching than fractional tree splits.

---

## 4. Deliverables Produced

- `forecasts_next7.csv`: 7-day multi-target operational forecast matching the §4.1 schema.
- `capacity_alerts.csv`: Bed occupancy capacity threshold monitoring matching the §4.2 schema.
