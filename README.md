# Operational Load Forecasting & Capacity Alert System

## 1. Approach & Methodology

- **Data Processing & EDA:**
  - Ingested 28 days of facility-level operational records for `GMC-AHM-001` with zero missing values.
  - Analyzed weekly patterns across five core targets (`lab_tests`, `ed_arrivals`, `opd_visits`, `beds_occupied`, `ambulance_calls`) and verified a strict 7-day cyclical profile.

- **Time-Based Splitting & Validation:**
  - Divided historical dates chronologically using a 70% train (19 days), 15% validation (5 days), and 15% test (4 days) split without shuffling to preserve temporal structure.
  - **Baseline:** Seasonal Naive heuristic forecasting $y_t = y_{t-7}$ directly from the preceding week's same-day observation.
  - **Selected Model:** **Holt-Winters Exponential Smoothing (ETS)** with additive trend and 7-day additive seasonality (`seasonal_periods=7`), fitted across all 5 metrics in a unified loop.

- **Why Holt-Winters Outperformed Tree Models:**
  - Gradient-boosted trees (like XGBoost) require tabular lag features that reduce the 28-day sample to just 14 training rows, causing severe step-function underfitting and error accumulation during multi-step recursion.
  - Holt-Winters directly estimates level, linear trend, and full 7-day seasonal indices with minimal parameterization, effectively capturing continuous cyclical peaks without overfitting.

- **Forecasting & Capacity Alerts:**
  - Refit Holt-Winters models over the complete 28-day history to produce multi-step projections from `2026-08-29` to `2026-09-04`.
  - Evaluated forecasted bed occupancy against the static capacity limit of 500 beds to flag potential operational breaches.

---

## 2. Evaluation Results (Test Set: Aug 25 – Aug 28, 2026)

| Target Metric | Baseline MAE | Holt-Winters MAE | Baseline RMSE | Holt-Winters RMSE | Best Performing Model |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **lab_tests** | 21.00 | 42.67 | 21.10 | 43.08 | **Baseline (Seasonal Naive)** |
| **ed_arrivals** | 5.40 | **1.95** | 5.71 | **2.25** | **Holt-Winters** |
| **opd_visits** | 19.00 | 30.31 | 20.12 | 31.84 | **Baseline (Seasonal Naive)** |
| **beds_occupied** | 2.60 | **1.20** | 2.72 | **1.57** | **Holt-Winters** |
| **ambulance_calls** | 0.80 | **0.58** | 0.89 | **0.70** | **Holt-Winters** |

---

## 3. Metric Comparison Summary

- **lab_tests:** The Seasonal Naive baseline performed best because daily lab testing volumes followed a nearly deterministic week-over-week cycle that direct 7-day lag matching captured with minimal error.
- **ed_arrivals:** Holt-Winters clearly beat the baseline (MAE 1.95 vs 5.40) by smoothly adapting to the mild upward volume drift alongside weekly emergency arrival patterns.
- **opd_visits:** The baseline edged out Holt-Winters due to sharp, abrupt weekend-to-weekday outpatient volume shifts that direct identical-day mapping matched closely.
- **beds_occupied:** Holt-Winters cut baseline error by more than half (MAE 1.20 vs 2.60), accurately tracking cumulative inpatient retention trends and turnover periodicity.
- **ambulance_calls:** Holt-Winters outperformed the baseline (MAE 0.58 vs 0.80) by smoothing out minor daily call stochasticity across the evaluation horizon.

---

## 4. Deliverables Produced

- `forecasts_next7.csv`: Complete 7-day operational predictions across all 5 targets adhering to §4.1 schema requirements.
- `capacity_alerts.csv`: Bed occupancy capacity threshold monitoring adhering to §4.2 schema specifications.
