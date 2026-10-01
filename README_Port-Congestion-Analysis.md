# Pre-Berthing Delay Analysis & Prediction: SMPK Port (Haldia Dock Complex)

Root-cause analysis, predictive modelling and discrete-event simulation of **vessel pre-berthing delays** at Syama Prasad Mookerjee Port (SMPK), Kolkata, with the goal of finding which delays are avoidable and which interventions reduce them most cost-effectively.

> **Capstone project, IISc × Ernst & Young (EY)** · SL202 Capstone in Mobility Systems, Jan–Apr 2026
> Advisors: Prof. Vijay Kovvali, Prof. Tarun Rambha · Team of 5 (team lead: Soham Chakraborty)

![Port simulation](assets/simulation_preview.gif)
*Discrete-event simulation of vessel flow from Sandheads to berth (full video: `port simulation.mp4`).*

---

## Problem

Vessels calling at SMPK's Haldia Dock Complex wait an average of **~2.8 days (69 h)** at anchorage before berthing. 8 in 10 vessels wait more than 24 hours, and the worst case was 19 days. Every day at anchor costs shipping lines **USD 15,000–20,000 per vessel** in demurrage. It also delays essential cargo (LPG, edible oils, fertilisers) and adds emissions from idling ships.

**Objectives**
1. Identify the **controllable and non-controllable factors** behind pre-berthing delay (PBD).
2. **Predict** the expected delay for an incoming vessel.
3. **Simulate** port operations to test interventions before investing in them.

---

## Data

No consolidated dataset existed, so we built one.

| Source | Content |
|---|---|
| SMPK Daily Position sheets (public PDFs) | **1,682 vessel calls** (Apr 2025 – Apr 2026), auto-extracted: arrival and haul-in times, berth, cargo, delay remarks |
| Diamond Harbour tide tables | 395 days of tidal observations (4 tides/day), corrected for the Haldia channel |
| SMPK bore-tide circulars | Bore-tide shutdown schedule and berth draft limits |

After cleaning: **1,343 usable vessel records** with 36 engineered features, including vessel size, tidal state, spring-neap cycle, cargo and terminal type, arrival timing and berth congestion.

---

## Key Findings (Root-Cause Analysis)

Breakdown of the 69.1 h average pre-berthing delay:

| Delay component | Share of delay hours | Controllable? |
|---|---|---|
| Berth queue / congestion | 26.6% | ✅ |
| Tidal / draft wait | 17.5% | ❌ |
| Ullage & cargo readiness | 17.1% | ✅ |
| Documentation & port admin | 13.3% | ✅ |
| Convoy & pilotage transit | 10.7% | — |
| Anchorage holding | 7.7% | — |
| Other (pre-service prep, crew, mechanical) | 7.1% | partly |

- **~59% of delay hours are controllable.** Paperwork and readiness alone account for ~30%, and berth congestion for ~26%.
- **Oil-tanker berths wait about twice as long as dry bulk** (106 h vs. 55 h). LPG reaches 182 h in the monsoon.
- **A higher tide does not mean less delay:** 82% of vessels can only enter at the daily peak tide regardless of its height.
- **Arrival timing matters:** Saturday arrivals wait ~35 h longer than Monday arrivals.
- **Bunching cascades:** when 4+ vessels arrive on the same day, the last one absorbs the whole queue, pushing delays past 200 h.

---

## Predictive Modelling

`ML pipeline.ipynb`, evaluated on a **chronological 80/20 split** (train: Mar 2025 – Jan 2026, test: Jan – Apr 2026) to avoid look-ahead leakage.

**Task A: 3-tier delay classification** (Short ≤ 24 h · Medium 24–120 h · Long > 120 h)

| Model | Test accuracy |
|---|---|
| Majority class (Medium) | 56.4% |
| Logistic Regression | 39.8% |
| LightGBM | 63.9% |
| Random Forest | 64.7% |
| **XGBoost** | **66.9%** |

**Task B: regression on log(delay hours)**, with metrics reported in hours

| Model | MAE (h) | RMSE (h) |
|---|---|---|
| Ridge (baseline) | 41.7 | 59.5 |
| Random Forest | 37.9 | 54.3 |
| **LightGBM** | **37.0** | **52.4** |

SHAP analysis was used to interpret feature contributions.

> **Note:** delays depend heavily on factors not recorded in the data, such as document readiness and queue state at arrival, so point predictions are noisy. The models are most useful for flagging high-risk arrivals early, not for exact ETAs. Recall on the "Long" class (24%) is the main area for improvement.

---

## Discrete-Event Simulation

`Simulation pipeline.ipynb` is a **SimPy** model of the full vessel journey: arrival at Sandheads → anchorage → tidal gate → river pilot and convoy → lock gate (dock-bound) or jetty (river-side) → berth → cargo operations → departure.

**Calibration**
- Inter-arrival and service times are fitted per cargo group from the observed data (Weibull, Lognormal, Gamma and Exponential, selected by AIC).
- Hourly channel depth is interpolated from the real Diamond Harbour tide table.
- Infrastructure constraints are modelled: a single lock gate (3 vessels per tidal slot), a pool of 12 river pilots, tugs, bore-tide shutdowns and per-berth draft limits.
- Unobservable parameters come from the literature (Sinha 2022, Ghosh 1998, PIANC WG55).

**Out-of-sample validation** (10 replications vs. 269 held-out vessels)

| Metric | Value | Threshold | Status |
|---|---|---|---|
| Mean delay, simulated vs. observed | 68.4 h vs. 73.6 h | ratio 0.70–1.30 | ✅ 0.93 |
| Bias | −5.2 h | < 15 h | ✅ |
| Q-Q decile RMSE | 9.6 h | < 20 h | ✅ |
| Q-Q Pearson r | 0.993 | > 0.95 | ✅ |
| KS test p-value | 0.033 | > 0.05 | ⚠️ marginal |

---

## What-If Scenarios & Recommendations

The calibrated simulator was used to test infrastructure, process and stress-test scenarios. Each was then evaluated with a **cost-benefit analysis** (BCR = benefit-cost ratio).

| Scenario | Intervention | Change in PBD | BCR |
|---|---|---|---|
| **Pre-arrival document readiness** | 50% reduction in tanker documentation delay | **−8 to −12 h** | **> 10** |
| **Peak-day pilots** | River pilot pool 12 → 16 on peak days | **−5 to −8 h** | **~3.5** |
| **Channel dredging** | Draft ceiling 7.3 m → 8.3 m | **−12 to −15 h** | **~2.1** |
| Second lock gate | Lock capacity 1 → 4 | −18 to −22 h | ~1.4 (capex ₹800–1,000 Cr) |
| +20% arrival rate (stress test) | No infrastructure expansion | +25 to +35 h | congestion risk |
| Deeper vessel mix (stress test) | +15% Panamax / Capesize | +10 to +18 h | delay risk |

**Top 3 recommendations:** pre-arrival documentation checks, additional peak-day pilots and channel dredging. Together they project a **25–35 h (~36%+) reduction in mean pre-berthing delay** without adding a single new berth.

---

## Repository Structure

```
Port-Congestion-Analysis-for-SMPK-Port/
├── data/
│   ├── haldia_pdf_only_final_master.csv           # 1,682 vessel calls extracted from SMPK PDFs
│   ├── haldia_one_year_merged.csv                 # vessel calls merged with tidal data
│   ├── diamond_harbour_tides_daily.csv            # daily tide observations
│   └── vessel_performance_with_terminal_specs.csv # berth and terminal specifications
├── ML pipeline.ipynb                              # feature engineering, classification, regression, SHAP
├── Simulation pipeline.ipynb                      # SimPy discrete-event simulation, validation, what-if
├── port simulation.mp4                            # simulation walkthrough video
└── README.md
```

## How to Run

```bash
git clone https://github.com/Soham2000/Port-Congestion-Analysis-for-SMPK-Port.git
cd Port-Congestion-Analysis-for-SMPK-Port
pip install numpy pandas scipy scikit-learn xgboost lightgbm shap simpy matplotlib jupyter
jupyter notebook
```

Open `ML pipeline.ipynb` for the predictive models (about 2 min) or `Simulation pipeline.ipynb` for the simulation, validation and what-if analysis (about 10 min).

## Tech Stack

Python · pandas · scikit-learn · XGBoost · LightGBM · SHAP · SimPy · Pygame · SciPy · Matplotlib

## Limitations & Future Work

- Document readiness and queue state at arrival are only partly observed, which caps predictive accuracy.
- The simulator underestimates neaping frequency (1.6% simulated vs. ~12.6% in remarks) and slightly over-smooths the delay distribution (marginal KS test).
- Next steps: add AIS vessel-tracking data for real-time queue features, and use the simulator to generate synthetic training data for rare long-delay cases.

## Acknowledgements

Ernst & Young (EY) for the problem statement and industry guidance, Prof. Vijay Kovvali and Prof. Tarun Rambha (IISc) for supervision, and teammates `<names>`.
