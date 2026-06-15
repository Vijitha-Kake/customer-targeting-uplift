# Customer Targeting & Incremental Revenue Optimization

*An uplift modeling project: deciding **who** to target with a marketing offer to
maximize incremental revenue — not just predicting who buys.*

I built this end-to-end **causal inference / uplift modeling** project on a real
randomized email experiment. Instead of predicting *who will buy*, I estimate *whom the
marketing email actually changes* — the incremental (treatment) effect per customer —
so marketing spend goes only where it causes a difference.

> **Demonstration project** on the public, randomized Hillstrom MineThatData email
> dataset, illustrating the experimentation + uplift methodology I apply in
> professional marketing/pricing analytics. Not a reproduction of any proprietary
> system.

## The core idea

A campaign's customers fall into four hidden groups:

| | Buys if treated | Doesn't buy if treated |
|---|---|---|
| **Buys if not treated** | Sure Thing (wasted spend) | **Sleeping Dog** (treatment *hurts*) |
| **Doesn't buy if not treated** | **Persuadable** (target these) | Lost Cause |

Uplift modeling finds the **Persuadables** and avoids the **Sleeping Dogs** — a
distinction ordinary "who will convert" models cannot make, because they rank by
*outcome* rather than by *treatment effect*.

## Dataset

Hillstrom MineThatData email challenge: 64,000 customers randomly assigned to a
men's-merchandise email, a women's-merchandise email, or no email; outcomes
(visit, conversion, spend) tracked over the following two weeks. I combine the two
email arms into a single "treated" group vs. the no-email control.

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
│   ├── 02_eda_balance_checks.ipynb
│   ├── 03_average_treatment_effect.ipynb
│   ├── 04_uplift_models.ipynb
│   ├── 05_uplift_evaluation.ipynb
│   └── 06_targeting_policy.ipynb
├── src/                # reusable modules (imported by notebooks + app)
│   ├── download_data.py
│   ├── power_analysis.py
│   └── uplift_models.py    # S-, T-, and hand-rolled X-learner
├── reports/
│   ├── sprint1_experiment_design.md   # Sprint 1: design + power analysis
│   └── figures/
├── app/                # Streamlit app (Sprint 7)
├── requirements.txt
└── README.md
```

## Scope: design **and** analysis

In production, the data scientist owns experiment **design** (eligible population,
stable-hash randomization, power-based sample sizing, guardrails) *and* the
**analysis**. The Hillstrom data is pre-randomized, so this repo analyzes an existing
experiment — but `reports/sprint1_experiment_design.md` documents my full from-scratch
design approach, so the project demonstrates both halves of the skill.

## What I found

**1. The email works — on average.** Conversion lifts from 0.57% to 1.07% (a 0.50pp
absolute, ~87% relative lift; 95% CI [0.355, 0.636]pp, p ≈ 4e-10). Statistically solid.

**2. But the average hides who responds.** Broken down by recency, the lift ranges
~0.25–0.73pp — a ~3× spread. A single average is a verdict on the campaign, not a
targeting rule. This motivates modeling the *individual* treatment effect.

**3. An honest modeling result.** Modeling uplift on the rare **conversion** outcome
(~1%), no meta-learner reliably beat random targeting (Qini AUC ≈ 0). I diagnosed the
bottleneck as **outcome density**, not algorithm strength — a rare outcome leaves the
per-customer effect buried in noise, and a stronger learner only overfits that noise.
So I switched the modeling target to the denser **visit** outcome, where the signal is
detectable. Conversion stays the business goal; **visit is the modeling proxy** the
data can support.

> Reporting the conversion result honestly — a model that does *not* beat random — and
> diagnosing *why*, is part of the analysis. Knowing when not to trust a model is the
> point of evaluating it.

**4. On visit, the models beat random — and the X-learner wins.** Qini AUC: X = 0.038,
S = 0.035, T = 0.034. The X-learner's edge is consistent with its propensity-weighted
blend handling the 2:1 treatment:control imbalance, as I expected.

**5. Targeting the right customers pays off.** Valuing a conversion at Hillstrom's real
average spend (~$116) and assuming $0.10/email, the profit-maximizing cutoff targets
the top ~57% of customers. That beats blanket sending, and beats random targeting of the
*same size* by more than 2× — the value is in emailing the *right* people, not fewer
people. The optimal cutoff tightens as per-contact cost rises, so uplift targeting
matters most for expensive channels (direct mail, outbound calls).

## App preview

A Streamlit app turns the analysis into a decision tool. *(Run `streamlit run app/streamlit_app.py` after `python src/train_model.py`.)*

![Targeting policy with cost slider](assets/app_targeting_policy.pdf)
*Targeting-policy view. Drag the cost-per-email slider and the optimal cutoff, profit, and profit curve update live. At $0.10/email and $116/conversion, targeting the top ~57% of customers beats blanket sending — and beats random targeting of the same size by more than 2×. The value is in emailing the **right** people, not fewer people.*

![Cost sensitivity](assets/app_cost_sensitivity.pdf)
*Raising the per-contact cost tightens the optimal cutoff — the policy responds to channel economics. Uplift targeting is marginal for cheap email but decisive for expensive channels (direct mail, outbound calls), where contacting a non-responder wastes real budget.*

![Per-customer scoring](assets/app_customer_scoring.pdf)
*Per-customer scoring. Enter a customer's features and the app returns a predicted uplift and a plain-language target / target-if-cheap / skip recommendation — the model's decision for one individual, framed for a non-technical user.*

![Method story](assets/app_story.pdf)
*The "Story" tab. A plain-language walkthrough of the method — experiment validation, the average effect, the three uplift models, the honest conversion→visit pivot, and the targeting policy — so a non-technical stakeholder understands what the tool does and why to trust it.*

## Roadmap

- [x] **Sprint 1** — Experiment design & power analysis
- [x] **Sprint 2** — Load, EDA & randomization balance checks
- [x] **Sprint 3** — Average treatment effect (did it work overall?)
- [x] **Sprint 4** — Uplift models (S-, T-, hand-rolled X-learner)
- [x] **Sprint 5** — Uplift evaluation (Qini curve, uplift-by-decile)
- [x] **Sprint 6** — Targeting policy, cost sensitivity & incremental revenue
- [x] **Sprint 7** — Streamlit app & polish

## References & data

**Dataset**
- Kevin Hillstrom, *MineThatData E-Mail Analytics And Data Mining Challenge* — the
  public, randomized email dataset used here.
  (blog.minethatdata.com; also mirrored on Kaggle as "Hillstrom Email Marketing").

**Methodology**
- Künzel, Sekhon, Bickel & Yu (2019), *Metalearners for estimating heterogeneous
  treatment effects using machine learning* (PNAS) — the S-, T-, and X-learner framework.
- Gutierrez & Gérardy (2017), *Causal Inference and Uplift Modelling: A Review of the
  Literature* — uplift evaluation, Qini curves, and why classification metrics don't apply.

**Key libraries**
- `scikit-uplift` — Qini curve / Qini AUC and uplift-by-percentile metrics.
- `statsmodels` — power analysis and the two-proportion / proportion-effect-size tools.
- `scikit-learn` — base learners for the meta-learners (the X-learner is implemented
  from scratch in `src/uplift_models.py`).

## Stack

Python · pandas · statsmodels · scikit-uplift · scikit-learn · Streamlit

*(The X-learner is implemented from scratch in `src/uplift_models.py` — pure
scikit-learn, no compiler/causalml dependency, so the repo clones and runs anywhere.)*

## Setup

```bash
python -m venv venv && source venv/bin/activate   # Windows: venv\Scripts\activate
pip install -r requirements.txt
python src/download_data.py
```

Then run the notebooks in order (`02` -> `06`). Sprint 1 is the design document in
`reports/`.


## Run the app (Sprint 7)

```bash
pip install -r requirements.txt
python src/download_data.py        # if not done yet
# run notebooks 02-06 once to create data/processed/*.csv
python src/train_model.py          # trains & saves the X-learner
streamlit run app/streamlit_app.py
```

The app has three tabs: a live **targeting-policy** view with a cost slider (watch the
optimal cutoff move), a per-customer **scoring** tool with a target/skip recommendation,
and a plain-language **story** of the method and the honest conversion→visit pivot.
