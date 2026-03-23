# Predictive Maintenance for Wind Turbines Using SCADA Data
**DSMLC Final Competition 2026**
**Team: Thrilling Three — Prabhasees Singh, Karanvir Mann, Carson Chan**
Supported by Enbridge & Databricks

[Live Dashboard](https://final-competition-dashboard.onrender.com/)
[GitHub Repository](https://github.com/KMANN-R2-D2/DSMLC-Final_Competition)


## Overview

Wind turbine failures are expensive, disruptive, and consequential as wind energy
becomes a larger share of our energy mix. This project builds an end-to-end
predictive maintenance system that detects developing faults before they cause
unplanned failures, using only the SCADA sensor data turbines already generate.

We analyzed 36 turbines across 3 wind farms using the CARE to Compare benchmark
dataset: Farm A (onshore, Portugal), Farm B (offshore, Germany), and Farm C
(offshore, Germany). Our system combines supervised and unsupervised machine
learning with statistical signal analysis to provide 47–48 hours of early warning
for thermal and mechanical faults with up to 99% recall.


## The Approach

Our solution runs two parallel tracks that feed into a combined alert system:

### Track 1 — Supervised XGBoost Detection
- Feature engineering: rolling means, z-scores, per-turbine baselines, rate of change
- One XGBoost model per farm, trained on labeled fault history
- Weighted class imbalance handling for rare fault events
- Turbine-specific z-scores consistently outperformed raw sensor readings across
  all three farms

### Track 2 — Unsupervised Autoencoder Detection
- Thermal signal analysis upstream: mean shift per sensor, FFT frequency pattern
  analysis, sensor decoupling detection
- Autoencoder trained only on healthy turbine data — no fault labels needed
- Learns what normal looks like; flags anything it cannot reconstruct
- Identifies which specific sensor deviated first, directing maintenance crews
  to the right component

### Combined Alert System
- 2/3 majority vote: Isolation Forest + XGBoost + rolling mean threshold
- Three alert tiers: Watch (schedule inspection within 72h), Alert (increase
  monitoring), Action (dispatch crew immediately)
- Every alert reports: farm, turbine, severity, persistence duration, and which
  specific sensor to inspect
- SHAP-driven attribution reduces inspection scope by ~60%

**XGBoost detects faults. The autoencoder explains them.**


## Models Built

### Primary Models
| Model | Type | Key Result |
|-------|------|-----------|
| XGBoost | Supervised | 99% recall Farm A, 47-48h early warning |
| Autoencoder | Unsupervised | Reconstruction gap shows gradual fault development |
| Isolation Forest | Unsupervised | Ensemble backbone |
| CUSUM | Statistical | Farm C first alarm at step 7 (99.7% earliness) |
| Rolling Mean Threshold | Statistical | Per-turbine 2.3σ baseline, 53-day gearbox warning |

### Supplementary Models
Hotelling T², VAE, VRAE, BOCPD, XGBoost with Pseudo-Labels, Random Forest,
Spectral Isolation Forest, Correlation Instability Monitor, SAX Discord Detection,
HistGradientBoosting

All models available in the repository.


## Results

### XGBoost Detection Performance

| Metric | Farm A | Farm B | Farm C |
|--------|--------|--------|--------|
| Accuracy | 98.8% | 86.2% | 69.8% |
| Precision | 98.5% | 83.3% | 67.9% |
| Recall | 99.0% | 92.5% | 73.3% |
| Separation (anomaly vs normal) | 92.6pp | 39.1pp | 9.5pp |
| Earliest detection | 48h | 47h | 67h |
| Primary fault type | Hydraulic/gearbox | Bearing damage | Mixed electrical/mechanical |

### Why Farm C Underperforms
Farm C's fault types — pitch encoder failures, communication faults, carbon brush
damage, yaw system issues — are electrical and control system problems. They produce
no thermal signatures and no predictable rotational changes. Our temperature and
speed sensors have almost nothing to say about them. This is a sensor coverage
problem, not a modelling failure. Farm B's transformer cell temperature has an SNR
of 0.85 before faults; Farm C's best sensor is 0.10 — nearly 10x weaker.

### Statistical Detection Results
- Gearbox oil temperature: **53-day early warning** using 7-day rolling mean +
  2.3σ per-turbine threshold (Farm A, Event 72)
- Thermal sensor decoupling detected 216–228 hours before faults on Farms B and C
- Fleet-wide thresholds fail: baseline temperatures differ by up to 10°C across
  turbines in the same farm — per-turbine baselines are essential


## Pre-Modelling Signal Analysis

Before building any models, we confirmed that physical fault signatures exist in
the raw data:
- **Farm B**: generator converter speed runs +3.13°C hotter in the hours before
  bearing failures
- **Farms B and C**: thermal sensors that normally track each other start diverging
  216–228 hours before faults ("sensor decoupling")
- **Farm A**: spectral analysis found excess frequency content on stator winding
  sensor 15 before faults — invisible in the raw time series


## Feature Engineering

Raw sensor readings are ambiguous. A gearbox oil temperature of 52°C might be
normal for one turbine but alarming for another. We engineered four feature types:

| Raw Sensor Data | Engineered Feature |
|----------------|-------------------|
| Gearbox oil temp = 52°C | Z-score = 2.8σ above this turbine's baseline |
| Generator RPM = 1,204 | 6-hour rolling mean trending down 3% |
| Bearing temp = 41°C | Delta: +0.8°C per 10 minutes |
| Nacelle temp = 28°C | 24-hour rolling std dev upward trend |

Z-score features consistently outranked their raw counterparts in XGBoost feature
importance across all three farms.


## Actionable Deployment — Three-Level Alert System

| Level | Trigger | Action |
|-------|---------|--------|
| **Watch** | XGBoost/BOCPD detects potential fault | Hotelling T² identifies component; schedule routine maintenance; engineer reviews data |
| **Alert** | Per-turbine rolling mean exceeds 2.3σ baseline | Increase monitoring frequency on this turbine |
| **Action** | Multiple sensors in same subsystem simultaneously anomalous, signal persists 30+ min | Dispatch crew immediately — specific component to inspect identified |


## Dataset

**CARE to Compare** — Real-world wind turbine SCADA benchmark

- **Download:** https://zenodo.org/records/15846963
- **Companion paper:** https://doi.org/10.3390/data9120138
- 95 datasets, 36 turbines, 3 wind farms
- 89 years of 10-minute interval SCADA data
- 44 labeled anomaly events, 51 normal behavior series
- Features: Farm A (86), Farm B (257), Farm C (957)



## Repository Structure
```
├── notebooks/
│   ├── xgboost/
│   │   ├── farm_a_xgboost.ipynb
│   │   ├── farm_b_xgboost.ipynb
│   │   └── farm_c_xgboost.ipynb
│   ├── autoencoder/
│   │   └── autoencoder_all_farms.ipynb
│   ├── gearbox/
│   │   ├── gearbox_analysis.ipynb
│   │   ├── gearbox_cross_turbine.ipynb
│   │   └── gearbox_explainability.ipynb
│   ├── thermal/
│   │   ├── thermal_2a_strongest_signals.ipynb
│   │   ├── thermal_2b_visualizations.ipynb
│   │   └── thermal_2c_monitoring_strategies.ipynb
│   └── supplementary/
│       └── [all other models]
├── dashboard/
└── README.md
```

---

## Future Work

- **Highest priority:** Add vibration and electrical current sensors to close
  the Farm C sensor gap — directly addressing pitch, communication, and yaw faults
- Deploy as real-time Databricks pipeline scoring every turbine every 10 minutes
  on new SCADA data (Bronze → Silver → Gold architecture already built)
- Quarterly per-turbine baseline refresh as turbines age and seasons change
- Explore Temporal Fusion Transformer for longer-range multi-turbine degradation
  patterns

---

## Limitations

- Farm C's electrical and control system fault types require sensor modalities
  beyond temperature and speed — the performance gap is a sensor gap, not a
  model gap
- Farm B had only 6 labeled anomaly events — addressed via cross-training with
  Farm C's 27 events
- The 67h earliest detection figure for Farm C is flagged as likely unreliable
  given the 9.5pp separation score
- Per-turbine baselines should be recomputed after major maintenance events

---

## Acknowledgements

Dataset: CARE to Compare benchmark — Leahy & Hu (2024)
Competition organized by the **Data Science and Machine Learning Club,
University of Calgary**
Supported by **Enbridge** and **Databricks**