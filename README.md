# Customer Targeting & Incremental Revenue Optimization

*An uplift modeling project: deciding **who** to target with a marketing offer to
maximize incremental revenue — not just predicting who buys.*

An end-to-end **causal inference / uplift modeling** project on a real randomized
email experiment. Instead of predicting *who will buy*, it estimates *whom the
marketing email actually changes* — the incremental (treatment) effect per
customer — so marketing spend goes only where it causes a difference.

> **Demonstration project** on the public, randomized Hillstrom MineThatData email
> dataset, illustrating experimentation + uplift methodology I apply in
> professional marketing/pricing analytics. Not a reproduction of any proprietary
> system.

## The core idea

A campaign's customers fall into four hidden groups:

| | Buys if treated | Doesn't buy if treated |
|---|---|---|
| **Buys if not treated** | Sure Thing (wasted spend) | **Sleeping Dog** (treatment *hurts*) |
| **Doesn't buy if not treated** | **Persuadable** (target these) | Lost Cause |

Uplift modeling finds the **Persuadables** and avoids the **Sleeping Dogs** — a
distinction ordinary "who will convert" models cannot make.

## Dataset

Hillstrom MineThatData email challenge: 64,000 customers randomly assigned to a
men's-merchandise email, a women's-merchandise email, or no email; outcomes
(visit, conversion, spend) tracked over the following two weeks.

Get the data:
```bash
python src/download_data.py        # via scikit-uplift
# or download Hillstrom.csv from Kaggle into data/raw/
```

## Project structure

```
customer-targeting-uplift/
├── data/
│   ├── raw/            # downloaded data (gitignored)
│   └── processed/      # cleaned data (gitignored)
├── notebooks/          # one per sprint
│   ├── 01_experiment_design.ipynb
│   ├── 02_eda_balance_checks.ipynb
│   ├── 03_average_treatment_effect.ipynb
│   ├── 04_uplift_models.ipynb
│   ├── 05_uplift_evaluation.ipynb
│   └── 06_targeting_policy.ipynb
├── src/                # reusable modules
│   ├── download_data.py
│   └── power_analysis.py
├── reports/            # design docs + figures
├── app/                # Streamlit app
├── requirements.txt
└── README.md
```

## Roadmap

- [x] **Sprint 1** — Experiment design & power analysis
- [ ] **Sprint 2** — Load, EDA & randomization balance checks
- [ ] **Sprint 3** — Average treatment effect (did it work overall?)
- [ ] **Sprint 4** — Uplift models (T-learner, X-learner)
- [ ] **Sprint 5** — Uplift evaluation (Qini curve, uplift-by-decile)
- [ ] **Sprint 6** — Targeting policy & incremental revenue
- [ ] **Sprint 7** — Streamlit app & polish

## Stack

Python · pandas · statsmodels · scikit-uplift · causalml · scikit-learn · Streamlit

## Setup

```bash
python -m venv venv && source venv/bin/activate   # Windows: venv\Scripts\activate
pip install -r requirements.txt
python src/download_data.py
```
