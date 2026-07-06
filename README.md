<div align="center">

# Electricity Price Forecasting in Restructured Power Systems
### Short-Window Machine Learning for Day-Ahead Market Prediction

**A leakage-free forecasting system that replaces long-history statistical models with engineered, short-term temporal signals — built for electricity markets where volatility, not stability, is the norm.**

![Python](https://img.shields.io/badge/Python-3.10-blue?logo=python&logoColor=white)
![ENTSO--E](https://img.shields.io/badge/ENTSO--E-Live_Data-1E3A8A)
![XGBoost](https://img.shields.io/badge/XGBoost-Best_Model-EB5E28)
![scikit--learn](https://img.shields.io/badge/scikit--learn-ML-F7931E?logo=scikit-learn&logoColor=white)
![SHAP](https://img.shields.io/badge/SHAP-Explainability-6A5ACD)
![License](https://img.shields.io/badge/License-MIT-lightgrey)
![Status](https://img.shields.io/badge/Status-Complete-success)


</div>

---

### ▸ Overview

The **Greek and Belgian day-ahead electricity markets** — two grids with almost
nothing in common structurally. Greece runs on a renewable-heavy, volatility-prone
mix; Belgium sits inside the tightly interconnected Central Western European market
coupling zone. Traditional forecasting approaches train on years of historical price
data and assume the market behaves tomorrow the way it did last year. In markets
this fast-moving, that assumption breaks down.

This project takes a different approach. All price, load, and generation data was
retrieved directly and manually from the **live ENTSO-E Transparency Platform** — the
real system operators, retailers, and traders across Europe use to publish and access
day-ahead prices. Instead of leaning on deep historical archives, the pipeline builds
**23 engineered short-term features** — price memory, rolling volatility, demand
ramps — and benchmarks three model families to find what actually predicts price
movement in these conditions.

> No synthetic benchmark data. No long-history assumptions. Just live market data,
> engineered signals, and a model that knows why it made its prediction.

---

### ▸ Key Highlights

| | |
|---|---|
| **Live market data** | Collected directly from the ENTSO-E Transparency Platform, not a static benchmark |
| **2 structurally distinct grids** | Renewable-volatile Greece vs. interconnection-heavy Belgium |
| **23 engineered features** | Price lags, rolling volatility, demand ramps, cyclical time encoding |
| **3 model families benchmarked** | Linear Regression, Random Forest, XGBoost |
| **87.3% R²** | on held-out data, using strictly time-ordered validation |
| **12.6% lower error** | vs. baseline Linear Regression (MAE) |
| **Zero data leakage** | every split respects chronological order via `TimeSeriesSplit` |
| **Self-explaining predictions** | SHAP attribution replaces black-box feature importance |

---

### ▸ The Core Idea

Most forecasting pipelines look like this:

```
Price(t) = f(Price(t-1), Price(t-2), ..., years of history)   (long-history, static features)
```

This project replaces that with a short-window, signal-driven approach:

```
Live ENTSO-E Data  →  Feature Engineering  →  Model Benchmark  →  Tuned XGBoost  →  SHAP Explanation
  (price, load,         (23 short-term          (LR / RF /          (Randomized-       (feature-level
   generation)            temporal signals)       XGBoost)            SearchCV)          attribution)
```

The result: a model that reacts to *recent* market memory and volatility rather than
assuming multi-year historical patterns still hold — and one that can explain exactly
which signal drove any individual prediction.

---

### ▸ Data Sources

| Source | Variable | Resolution |
|---|---|---|
| **ENTSO-E Transparency Platform** | Day-ahead market clearing price | Hourly, EUR/MWh |
| **ENTSO-E Transparency Platform** | Actual & forecasted system load | Hourly, MW |
| **ENTSO-E Transparency Platform** | Wind generation output | Hourly, MW |
| **ENTSO-E Transparency Platform** | Solar generation output | Hourly, MW |
| **Derived** | Renewable penetration ratio | Hourly, % of total generation |

All series were manually pulled from the platform's public bidding-zone datasets for
Greece (GR) and Belgium (BE), chronologically ordered, and merged into a single
16,000+ row hourly dataset spanning both markets.

---

### ▸ Pipeline

```
1. Data Acquisition        →  ENTSO-E Transparency Platform (GR & BE bidding zones)
2. Preprocessing           →  Timestamp parsing, chronological sort, null handling
3. Feature Engineering     →  23 features: price lags, rolling stats, load ramp, cyclical time
4. Model Benchmarking      →  Linear Regression, Random Forest, XGBoost — TimeSeriesSplit CV
5. Hyperparameter Tuning   →  RandomizedSearchCV on XGBoost (depth, learning rate, subsampling)
6. Explainability          →  SHAP value attribution on the tuned model
7. Deployment Simulation   →  Serialized model tested end-to-end on unseen future data
```

---

### ▸ Model Performance

| Model | MAE (EUR/MWh) | RMSE (EUR/MWh) | R² |
|---|---|---|---|
| **XGBoost (tuned)** ■ | **12.56** | **20.42** | **0.873** |
| Random Forest | 13.67 | 22.35 | 0.848 |
| Linear Regression *(baseline)* | 14.37 | 22.61 | 0.845 |

**XGBoost was selected** as the final model for its balance of accuracy, training
speed, and compatibility with SHAP-based interpretability — outperforming the
baseline by **12.6% lower MAE**.

---

### ▸ Results

- The tuned model explains **87.3% of price variance** across both markets using only
  information available at day-ahead bidding time — no lookahead, no actuals-as-input
- **Price lag features (1h, 24h, 168h) and load ramp** consistently rank as the
  strongest predictors across both SHAP and native feature importance, confirming
  that short-term market memory dominates over long historical trends
- Performance held consistently across **two structurally different grids**, indicating
  the feature engineering approach generalizes rather than overfitting to one market's
  specific dynamics
- A held-out inference test on unseen future data confirmed the full pipeline —
  feature reconstruction through prediction — behaves consistently outside the
  original training notebook

---

### ▸ Tech Stack

<table>
<tr>
<td valign="top">

**Platform**
- ENTSO-E Transparency Platform
- Jupyter / Google Colab

**Language**
- Python 3.10

</td>
<td valign="top">

**Machine Learning**
- scikit-learn
- XGBoost
- Random Forest
- Linear Regression (baseline)
- SHAP

</td>
<td valign="top">

**Data & Viz**
- Pandas / NumPy
- Matplotlib / Seaborn
- Joblib

</td>
</tr>
</table>

---

### ▸ Repository Structure

```
Electricity_Price_Forecasting/
├── notebook/
│   └── Electricity_prices.ipynb         # Single end-to-end notebook (all stages combined)
├── data/
│   ├── dataset.csv                      # Raw data collected from ENTSO-E Transparency Platform
│   └── features_dataset.csv             # Final engineered feature set (23 features)
├── models/
│   └── final_xgboost_model.pkl          # Serialized, tuned XGBoost model
├── results/
│   ├── model_comparision.csv            # MAE, RMSE, R² across all models
│   ├── predictions_lr.csv
│   ├── predictions_rf.csv
│   └── predictions_xgb.csv
├── docs/
│   └── Results_and_Discussion.docx      # Detailed write-up of findings
├── .gitignore
└── README.md
```

> Raw ENTSO-E exports are excluded from version control due to size — see the
> notebook for the full data retrieval and preprocessing logic.

---

### ▸ Study Markets

**Greece (GR) & Belgium (BE)**
- Greece: renewable-heavy generation mix, higher day-ahead price volatility
- Belgium: Central Western European (CWE) market coupling, high interconnection
- Combined dataset: 16,000+ hourly records across both bidding zones

---

### ▸ Future Work

- [ ] Extend the benchmark to LightGBM and CatBoost for a full boosting-family comparison
- [ ] Introduce an LSTM with feed-forward error correction for a sequence-model comparison
- [ ] Break down performance by season and by peak vs. off-peak hours
- [ ] Automate live ingestion directly from the ENTSO-E API, replacing manual exports
- [ ] Expose the trained model through a lightweight API for real-time forecasting

---

### ▸ Author

**Chinthalapalli Manasa**
· chinthalapalli.manasa15@gmail.com

---

<div align="center">
<sub>Built on live ENTSO-E market data — because forecasting electricity prices should reflect the market as it actually behaves, not as historical archives assume it does.</sub>
</div>
