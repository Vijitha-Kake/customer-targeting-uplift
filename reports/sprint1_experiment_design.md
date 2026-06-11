# Customer Targeting & Incremental Revenue Optimization — Sprint 1: Experiment Design & Power Analysis

## 1. Business question

Given a marketing email (treatment), **which customers should we target to
maximize incremental revenue** — and which should we leave alone because the
email does nothing or actively reduces their value?

The operative word is *incremental*. The goal is not to predict who buys, but to
estimate whom the email **changes** (the treatment effect).

## 2. Experiment setup

| Element | Definition |
|---|---|
| **Treatment** | Customer receives a promotional email (men's or women's campaign) |
| **Control** | Customer receives no email |
| **Unit of randomization** | Individual customer |
| **Assignment** | Random (Hillstrom: ~1/3 men's email, ~1/3 women's email, ~1/3 control) |
| **Observation window** | Two weeks following the campaign |

Because assignment is random and treatment vs. control are observed over the same
period, any difference between groups is attributable to the email — not to
seasonality, paydays, or other time effects. This is why A/B testing is
**treatment vs. control over the same window**, not before-vs-after the same group.

## 3. Hypotheses

- **H0 (null):** the email has no effect — conversion rate(treatment) = conversion rate(control).
- **H1 (alternative):** the email changes conversion — conversion rate(treatment) ≠ conversion rate(control). (Two-sided: an email can help OR hurt — "sleeping dogs.")

## 4. Metrics

- **Primary metric:** conversion rate (binary). Stable, interpretable, enough events.
  - Test: two-proportion z-test / chi-square (binary outcome → proportion test).
- **Secondary outcome:** visit rate (binary) — higher base rate, useful for power.
- **Guardrail metric:** average spend (continuous) — watched to ensure we don't win
  conversions while losing revenue. Skewed, so interpret with care (robust methods /
  rely on conversion). Test for spend would be a t-test, with caution re: skew.

## 5. Power analysis

Four interlocking quantities — fix three, solve the fourth:

- **Effect size** — smallest lift worth detecting (the MDE)
- **Significance level (alpha)** — false-positive tolerance (0.05)
- **Power (1 - beta)** — chance of catching a real effect (0.80)
- **Sample size (n)** — usually solved for

Since Hillstrom already ran, we work **backwards**: given n per group, what is the
minimum detectable effect (MDE)? And forward: for a target lift, what n is needed?

**Results (illustrative baseline = 1.0% control conversion; replace with real rate in Sprint 2):**

- Per group (one email arm vs control): ~21,333 customers
- Minimum detectable effect at alpha=0.05, power=0.80:
  - ~0.29 percentage points (1.00% -> ~1.29%), ~29% relative lift
- To detect a 0.3pp lift (1.0% -> 1.3%): ~19,743 per group required
  - Hillstrom (~21,333/group) is **adequately powered** for a realistic email effect.

**Interpretation:** the experiment can reliably detect a meaningful email lift, so a
null result would be trustworthy rather than an underpowered miss. This validates
spending modeling effort on the data.

## 6. Randomization / sanity checks (to run in Sprint 2)

- Group sizes roughly equal across arms.
- Covariate balance: recency, history/spend, channel distributions similar across arms.
- No systematic differences pre-treatment (confirms randomization held).

## 7. Next step

Sprint 2: load the real data, recompute the power analysis with the actual control
conversion rate, and run the balance checks above.

---

## Appendix A — Effect size: choose by outcome type

The effect size measures how far apart the two groups are on a *standardized*
scale. The right formula depends on the outcome type — this is the part people
most often get wrong.

| Outcome type | Effect size | Formula | Standardization |
|---|---|---|---|
| **Proportion** (conversion, visit) | Cohen's **h** | h = 2·arcsin(√p₂) − 2·arcsin(√p₁) | arcsin-√ variance-stabilizing transform |
| **Continuous** (spend) | Cohen's **d** | d = (mean₁ − mean₂) / SD_pooled | divide by pooled standard deviation |

**Why two formulas.** For a continuous outcome you estimate the mean and the
spread separately, so you standardize by dividing by the pooled SD (Cohen's d).
For a *proportion*, the variance is locked to the rate itself — Var = p(1−p) — so
there is no free SD to pool. The arcsin-√ transform in Cohen's h does the
standardizing instead, putting the difference on a scale where variance is
constant regardless of the base rate.

**Worked values (this project):**
- Conversion 1.0% → 1.3%: Cohen's h ≈ 0.0282 (primary metric, what we powered on).
- Spend (illustrative): with a large skew-driven SD, Cohen's d comes out tiny —
  which is the quantitative reason spend is a guardrail, not a primary metric.
  Detecting an effect on skewed spend would need an impractically large sample.

## Appendix B — Sample-size formula (two proportions)

Classic two-proportion sample size, n **per group** (equal allocation):

    n = ( z_(1−α/2)·√(2·p̄·(1−p̄)) + z_(1−β)·√(p₁(1−p₁) + p₂(1−p₂)) )²  /  (p₂ − p₁)²

where p̄ = (p₁ + p₂) / 2.

- z_(1−α/2) = 1.96 for α = 0.05 (two-sided) — controls false positives.
- z_(1−β)   = 0.84 for power = 0.80 — controls ability to detect a real effect.
- (p₂ − p₁)² in the denominator → sample size scales with 1/(difference)².
  Halving the effect you want to detect requires ~4× the sample. Small effects
  are quadratically expensive.

Computed with statsmodels (`NormalIndPower.solve_power` + `proportion_effectsize`),
this gives ~19,700 per group to detect 1.0% → 1.3% at α=0.05, power=0.80 — and
Hillstrom provides ~21,300 per group, so the experiment is adequately powered.

Note: a bare Cohen's-h formula n = (z_a + z_b)² / h² gives ~9,900 here — it uses a
different variance convention and understates n by ~2×. The two-proportion formula
above (matching statsmodels) is the one to cite.
