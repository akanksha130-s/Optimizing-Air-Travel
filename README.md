# ✈️ Optimizing Air Travel: A Data-Driven Approach to Flight Delay Analysis and Prediction

> Uncovering the hidden patterns behind flight delays and building predictive models to help airlines act on what they can actually control.
---

## Overview

Flight delays cost airlines billions annually and erode passenger trust. This project uses historical U.S. airline on-time performance data to:

1. **Uncover hidden patterns** behind flight delays through exploratory data analysis (EDA)
2. **Build predictive models** that classify whether a flight will be delayed and estimate delay duration
3. **Generate actionable recommendations** for airlines, prioritized by what's actually within their operational control

A custom metric — the **Operational Adjustability Index (OAI)** — and **SHAP-based explainability** are used to separate *controllable* delay causes (carrier issues, late aircraft) from *external* ones (weather, NAS, security), so recommendations focus on where airlines can realistically intervene.

This project was built for the **Society of Business Open Projects 2025 — Analytics Track**.

---

## Objectives

| Goal | Description |
|---|---|
| **Uncover Hidden Patterns** | In-depth EDA to identify recurring trends and significant correlations contributing to flight delays |
| **Develop Predictive Capability** | Models to predict delay likelihood (classification) and delay duration (regression) |
| **Generate Actionable Insights** | Data-backed recommendations for delay mitigation and operational resilience |

**Prediction Targets:**
- **Classification:** Will a flight be delayed? (`arr_del15`)
- **Regression:** How long will the delay be, in minutes? (`arr_delay`)

---

## Dataset

The dataset is the **Airline Delay Cause** dataset (U.S. Bureau of Transportation Statistics), containing monthly, carrier- and airport-level flight delay records broken down by cause.

| Column | Description |
|---|---|
| `year`, `month` | Reporting period |
| `carrier`, `carrier_name` | Airline carrier code and full name |
| `airport`, `airport_name` | Airport code and full name |
| `arr_flights` | Number of arriving flights |
| `arr_del15` | Number/indicator of flights delayed 15+ minutes |
| `carrier_ct`, `weather_ct`, `nas_ct`, `security_ct`, `late_aircraft_ct` | Delay counts by cause |
| `arr_cancelled`, `arr_diverted` | Cancelled / diverted flight counts |
| `arr_delay` | Total arrival delay in minutes |
| `carrier_delay`, `weather_delay`, `nas_delay`, `security_delay`, `late_aircraft_delay` | Delay minutes by cause |

Dataset Link - https://drive.google.com/drive/folders/16-kvGEH5IE2jt5nIJPqkWfMDRDXqCBl-

---

## Methodology

```
Data Collection → Preprocessing → EDA → Feature Engineering → Modeling → Explainability & OAI
```

1. **Data Cleaning & Preprocessing** — dropped rows with missing `arr_delay`; imputed numerical features with median and categorical features with mode; engineered a `season` feature from `month`; one-hot encoded `carrier`, `airport`, and `season`.
2. **Exploratory Data Analysis** — delay distribution, cause breakdown, temporal (monthly) trends, correlation matrix.
3. **Root Cause Analysis** — total & average delay minutes by cause; controllable vs. external delay split.
4. **Predictive Modeling** — Logistic Regression / Random Forest / Decision Tree for classification; Linear Regression / Random Forest / Decision Tree for regression.
5. **Operational Adjustability Index (OAI)** — a custom weighted metric that prioritizes controllable delay causes (carrier, late aircraft) over external ones (weather, NAS, security) when evaluating model impact.
6. **Explainability (SHAP)** — SHapley Additive exPlanations used to interpret feature contributions to predicted delays, re-weighted by OAI to highlight actionable drivers.

---

## Key Findings (EDA & Root Cause Analysis)

- Arrival delay distribution is **highly right-skewed** — most flights are on time or only mildly delayed, with a long tail of extreme outliers.
- **Late Aircraft Arrival (~39%)** and **Carrier Delays (~34%)** are the top two contributors to total delay minutes, followed by **NAS delays (~21%)** and **Weather delays (~5%)**. Security delays are negligible (<1%).
- **Controllable delays** (carrier + late aircraft) account for **over 70%** of total delay minutes — meaning the majority of delays are actionable from an operational standpoint.
- Delays occur year-round, with outlier spikes concentrated in **summer and winter** months, suggesting seasonal effects.
- `arr_delay` correlates most strongly with **carrier delay** and **late aircraft delay**; security delays are largely isolated/uncorrelated events.

---

## Model Performance

### Classification — "Will the flight be delayed?"

| Model | Accuracy | ROC-AUC |
|---|---|---|
| Logistic Regression | 98.5% | 0.996 |
| Random Forest | 100%* | 1.000* |
| Decision Tree | 100%* | 1.000* |

*\*Near-perfect scores stem from delay-cause counts being present as features, which are strongly deterministic of the `arr_del15` target — see [Limitations](#-limitations--future-work).*

### Regression — "How long will the delay be (minutes)?"

| Model | MAE (min) | RMSE (min) | R² |
|---|---|---|---|
| Linear Regression | 880.9 | 2583.7 | 0.948 |
| **Random Forest** ⭐ | **650.7** | **2050.2** | **0.967** |
| Decision Tree | 801.4 | 2703.8 | 0.943 |

Random Forest was the best-performing model for both tasks, with ensemble methods outperforming linear baselines throughout.

### SHAP Explainability

- `late_aircraft_ct` and `carrier_ct` are the dominant drivers of predicted delay duration, followed by `nas_ct` and `weather_ct`.
- Under OAI weighting, **controllable delay features explain over 90%** of the model's explained variance, while external causes contribute **less than 7%** — reinforcing that the model (and resulting recommendations) is grounded in what airlines can actually change.

---

## Actionable Recommendations

1. **Prioritize Controllable Delay Factors** — focus mitigation on `carrier_delay` and `late_aircraft_delay`, which together account for ~73% of total delay minutes.
2. **Improve Ground Operations & Resource Allocation** — audit turnaround processes, standardize ground operations, invest in digital tools for crew coordination.
3. **Adjust Flight Schedules** — introduce schedule slack for routes/times with recurring delays; use the predictive model to proactively flag high-risk flights.
4. **Enhance Passenger Communication** — real-time delay/gate-change notifications and two-way messaging for rebooking and support.
5. **Collaborate on External Delay Reduction** — for the ~27% of delays from weather/NAS/security, share real-time data with airports and authorities for joint contingency planning.

---

## Repository Structure

```
flight-delay-analysis/
├── data/
│   ├── raw/                          # Place Airline_Delay_Cause.csv here (not tracked)
│   └── processed/                    # cleaned_flight_delay_data.csv (generated)
├── notebooks/
│   └── flight_delay_analysis.ipynb   # Full analysis: EDA → Modeling → SHAP
├── docs/
│   ├── project_brief.pdf             # Open Projects 2025 problem statement
│   ├── column_definitions.xlsx       # Data dictionary
│   └── presentation_deck.pdf         # Summary presentation
├── reports/
│   └── figures/                      # Exported charts (generated)
├── requirements.txt
├── .gitignore
├── LICENSE
└── README.md
```

---

## Setup & Usage

### 1. Clone the repository
```bash
git clone https://github.com/<your-username>/flight-delay-analysis.git
cd flight-delay-analysis
```

### 2. Create a virtual environment & install dependencies
```bash
python -m venv venv
source venv/bin/activate      # Windows: venv\Scripts\activate
pip install -r requirements.txt
```

### 3. Add the dataset
Download `Airline_Delay_Cause.csv` and place it in `data/raw/`.

### 4. Run the notebook
```bash
jupyter notebook notebooks/flight_delay_analysis.ipynb
```

---

## Tech Stack

- **Language:** Python 3.10+
- **Data Handling:** pandas, numpy
- **Visualization:** matplotlib, seaborn
- **Modeling:** scikit-learn (Logistic/Linear Regression, Random Forest, Decision Tree)
- **Explainability:** SHAP
- **Environment:** Jupyter Notebook

---

## Limitations & Future Work

- Classification models achieving 100% accuracy indicate likely **target leakage** — delay-cause count columns (`carrier_ct`, `late_aircraft_ct`, etc.) are near-deterministic of `arr_del15` since they're derived from the same reporting record. Future iterations should model at the flight level using only pre-departure features (schedule, weather forecast, aircraft rotation) for a realistic early-warning system.
- Add time-series cross-validation to better capture seasonal effects.
- Deploy the Random Forest regression model behind a simple API/dashboard for real-time delay risk scoring.

---

## Author

**Akanksha Sahni**
📧 Akanksha_s@hs.iitr.ac.in

Submitted to **Society of Business — Students' Technical Council, Open Projects 2025 (Analytics Track)**

---

## License

This project is licensed under the [MIT License](LICENSE).
