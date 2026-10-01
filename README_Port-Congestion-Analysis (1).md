# Pre-Berthing Delay Analysis & Prediction: SMPK Port (Haldia Dock Complex)

Predictive modelling and discrete-event simulation of **vessel pre-berthing delay (PBD)** at Syama Prasad Mookerjee Port (SMPK), Kolkata.

> **Capstone project, IISc × Ernst & Young (EY)**, Jan–Apr 2026
> Advisors: Prof. Vijay Kovvali, Prof. Tarun Rambha · Team: Soham Chakraborty (lead), Shreya Ghosh, Kartikeya Gaur, Revathy Ramesh, Sagnika Mukhopadhyay

---

## Data

| File | Content |
|---|---|
| `haldia_pdf_only_final_master.csv` | 1,682 vessel calls (Apr 2025 – Apr 2026) extracted from SMPK port PDF notices |
| `diamond_harbour_tides_daily.csv` | 395 days of Diamond Harbour tide observations |
| `haldia_one_year_merged.csv` | Vessel calls merged with tidal data |
| `vessel_performance_with_terminal_specs (1).csv` | Berth and terminal specifications |

After filtering, **1,343 vessel records** are used. The mean pre-berthing delay is **69.1 h**: about 96 h for river-side (oil jetty) vessels and 55 h for dock-bound vessels.

---

## Predictive Modelling: `ML pipeline.ipynb`

36 engineered features, evaluated on a **chronological 80/20 train-test split** (1,074 / 269 vessels).

**3-class delay classification** (Short ≤ 24 h · Medium 24–120 h · Long > 120 h)

| Model | Test accuracy |
|---|---|
| Logistic Regression | 39.8% |
| LightGBM | 63.9% |
| Random Forest | 64.7% |
| **XGBoost** | **66.9%** |

**Regression on log(delay)**, with metrics in hours

| Model | MAE (h) | RMSE (h) |
|---|---|---|
| Ridge | 41.7 | 59.5 |
| Random Forest | 37.9 | 54.3 |
| **LightGBM** | **37.0** | **52.4** |

SHAP is used to interpret the models.

---

## Discrete-Event Simulation: `Simulation pipeline.ipynb`

A **SimPy** model of the vessel journey: anchorage → tidal gate → convoy and pilotage → lock gate or river-side jetty → berth → departure. It is calibrated with distributions fitted to the observed data (selected by AIC) and the real tide table.

**Out-of-sample validation** (10 replications vs. 269 held-out vessels)

| Metric | Value | Status |
|---|---|---|
| Mean PBD, simulated vs. observed | 68.4 h vs. 73.6 h (ratio 0.93) | Pass |
| Q-Q decile Pearson r | 0.993 | Pass |
| Q-Q decile RMSE | 9.6 h | Pass |
| KS test p-value | 0.033 | Marginal |

**What-if scenarios** (5 replications each, change in mean PBD vs. the base of 71.0 h)

| Scenario | PBD reduction |
|---|---|
| Night navigation | 28.9% |
| Channel dredging (+0.8 m draft) | 11.1% |
| More cranes and berths | 7.2% |
| Larger river-pilot pool | 6.5% |
| Combined reforms | 40.5% |

`port simulation.mp4` is a video walkthrough of the simulation.

---

## How to Run

```bash
pip install numpy pandas scipy scikit-learn xgboost lightgbm shap simpy matplotlib jupyter
jupyter notebook
```
