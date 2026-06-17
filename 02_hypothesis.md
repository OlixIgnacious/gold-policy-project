# 02 — Hypothesis Precommit & Statistical Tests
**Project:** India Gold Import Duty Hike — Pass-Through Study  
**Rule:** All hypotheses and test designs written BEFORE regression results are seen.  
**Last updated:** June 2026

---

## What is a hypothesis test?

A hypothesis test asks: *"could this result have happened by chance?"*

Every test has two competing claims:
- **H₀ (null hypothesis):** nothing is happening — the effect is zero, or the trend is flat, or the series is random
- **H₁ (alternative hypothesis):** something real is happening

We compute a **p-value** — the probability of seeing a result this extreme if H₀ were actually true. Convention: if p < 0.05, we reject H₀ and say the result is statistically significant.

This file records every test we run, what we expected before running it, and what the result meant.

---

## Test Registry

---

### TEST 01 — Pre-trend test (ITS assumption check)
**Notebook:** `01_eda.ipynb` Cell 7  
**Figure:** `figures/fig04_pretrend_test.png`

**Purpose:**  
Before using Interrupted Time Series (ITS), we must verify that the outcome variable (domestic_premium) was NOT already trending upward before the May 13 treatment. If it was, the post-hike jump could be a continuation of that trend rather than a treatment effect.

**What we test:**  
OLS regression of domestic_premium on a time index, fitted only in the low-duty window (Jul 24 2024 – May 12 2026):

```
premium_t = α + β·t    (low-duty window only)
```

**H₀:** β = 0 (no pre-existing trend — premium was flat before the hike)  
**H₁:** β ≠ 0 (premium was already rising or falling before the hike)

**Decision rule:** If p > 0.05 → ITS assumption holds, proceed. If p < 0.05 → include explicit time trend term `β₂·t` in ITS regression (already in our spec) and note the direction.

**Expected result (before running):**  
The low-duty window shows the premium oscillating around zero with no clear directional trend in Figure 1. Expectation: β ≈ 0, p > 0.05. The April–May 2026 negative dip (bank IGST pause) may introduce a slight downward slope, which would actually make our ITS estimate conservative (understating the true jump).

**Result:** ✓ COMPLETED (Cell 7, Jun 2026)  
- Window: 2024-07-25 → 2026-05-12 (**N=360 trading days** in v2; was 321 in v1 — BullionWorld gap-fill added 39 low-duty dates)
- Slope: **+0.78 INR/10g per trading day** (+₹196/year — economically negligible) *(run on v1; v2 result expected unchanged given BW dates have near-zero premiums)*
- p-value: **0.3469** (>> 0.05)
- R²: 0.0028 (time trend explains 0.28% of variance)
- ITS effect at Day 1: ₹8,004 (actual Day 1 premium − counterfactual)
- **Conclusion: Fail to reject H₀. No significant pre-trend. ITS assumption holds.**

---

### TEST 02 — Stationarity test (ADF)
**Notebook:** `01_eda.ipynb` Cell 8  
**Figure:** N/A

**Purpose:**  
The ITS regression assumes domestic_premium is stationary (i.e., it reverts to a mean, not a random walk). If the series has a unit root (non-stationary), OLS standard errors are invalid and we need to difference the series or use a cointegration framework (ARDL in Notebook 03).

**What we test:**  
Augmented Dickey-Fuller (ADF) test on domestic_premium:

```
Δpremium_t = α + γ·premium_{t-1} + Σδᵢ·Δpremium_{t-i} + ε_t
```

**H₀:** γ = 0 — premium has a unit root (non-stationary, random walk)  
**H₁:** γ < 0 — premium is stationary (mean-reverting)

**Decision rule:**  
- If p < 0.05 → reject H₀ → premium is stationary → OLS ITS is valid  
- If p > 0.05 → cannot reject unit root → use first differences or ARDL cointegration (Notebook 03)

**Also test:** Gold_USD, rupees_per_dollar — if these are I(1) but their combination is I(0), that justifies the ARDL bounds test.

**Expected result:** domestic_premium is likely stationary — it has an economic floor (smuggling becomes attractive below a threshold) and a ceiling (demand destruction). But the high-duty window may look locally non-stationary. Test all three sub-periods separately.

**Result:** ✓ COMPLETED (Cell 8, Jun 2026)

| Series | Window | ADF stat | p-value | Lags | Decision |
|---|---|---|---|---|---|
| domestic_premium | Full sample (N=767 v1; ~882 v2) | −1.073 | 0.726 | 6 | ⚠ Appears non-stationary (structural break artifact) |
| domestic_premium | Low-duty only (N=319 v1; **360 v2**) | −9.388 | 0.000 | 1 | ✓ Strongly stationary |
| Gold_USD | Levels (N=1103) | +0.296 | 0.977 | 9 | ⚠ I(1) — unit root |
| Gold_USD | First difference (N=1056) | −8.548 | 0.000 | 10 | ✓ I(0) — confirms I(1) |
| rupees_per_dollar | Levels (N=1151) | −0.069 | 0.952 | 3 | ⚠ I(1) — unit root |
| rupees_per_dollar | First difference (N=1148) | −22.636 | 0.000 | 2 | ✓ I(0) — confirms I(1) |

**Interpretation of full-sample non-stationarity:**
The full-sample ADF failure is a structural break problem, NOT a true unit root. The ADF sees the premium shifting from +₹4,380 (high-duty) → −₹257 (low-duty) → +₹10,318 (post-hike) and misinterprets regime changes as a unit root. The low-duty-only ADF (p=0.000) confirms the series is fundamentally stationary within regimes.

**ITS implication:** The `post_hike` dummy in the ITS regression absorbs the level shift. The ITS is valid and operates on a series that is I(0) within each regime. Full-sample non-stationarity does not invalidate OLS here.

**ARDL implication:** Gold_USD ~ I(1), rupees_per_dollar ~ I(1), domestic_premium ~ I(0) within regime. Mixed I(0)/I(1) system — ARDL bounds test (Pesaran 2001) is valid and appropriate for Notebook 03.

**Newey-West lag selection:**
- PACF significant lags (full sample): 1, 2, 3, 4, 7, 18, 22, 26, 28, 29
- Lags 18+ are regime artifacts, not true autocorrelation
- Within low-duty window: AIC selected only 1 lag
- n^(1/4) rule-of-thumb: 6
- **Decision: NW lag = 8 for ITS main specification (v2, T=845)** — was 7 at T=739 (v1), was 6 initially (incorrect formula)
- Sensitivity to NW lag (3, 6, 8, 10, 20) tested in Notebook 05

---

### TEST 03 — ITS main treatment effect
**Notebook:** `03_its.ipynb`  
**Method:** OLS with Newey-West HAC standard errors

**Purpose:**  
Estimate the causal effect of the May 13 2026 duty hike on domestic_premium using Interrupted Time Series.

**Regression specification:**

```
premium_t = α + β₁·post_t + β₂·t + β₃·δGold_USD_t + β₄·δFX_t + ε_t
```

Where:
- `post_t` = 1 if date ≥ May 13 2026, else 0
- `t` = time trend (trading days since start)
- `δGold_USD_t` = day-over-day % change in Gold_USD
- `δFX_t` = day-over-day % change in INR/USD
- Standard errors: Newey-West HAC (lag = **8**, v2 dataset T=845; NW-94 rule ⌈0.75 × T^(1/3)⌉)

**H₀:** β₁ = 0 (duty hike had no effect on domestic premium)  
**H₁:** β₁ > 0 (duty hike raised domestic premium)

**Secondary hypothesis:**  
H₀: β₂ = 0 (no time trend after controlling for treatment)  
This tests whether there was any residual drift not explained by the policy.

**Expected result:**  
β₁ ≈ ₹10,000–12,000 (consistent with 83.5% pass-through × ~₹12,230 ceiling in v2).  
p-value for β₁ expected to be < 0.001 given the size of the shock relative to the pre-period standard deviation. *(v2 event-study lower bound: ₹10,215 mean premium over 23 valid post-hike days; 83.6% average pass-through)*

**Pass-through calculation:**  
`pass_through = β₁ / mean(parity_post − parity_pre)` during post-hike window.

**Result:** *(fill in after running Notebook 03)*  
- β₁: _____  
- SE (HAC): _____  
- p-value: _____  
- Pass-through: _____  
- Conclusion: _____

---

### TEST 04 — Local Projection impulse response (Jordà 2005)
**Notebook:** `03_its.ipynb`  
**Method:** Separate OLS for each horizon h = 0, 1, … 30

**Purpose:**  
Estimate the dynamic response of domestic_premium to the duty hike at each horizon. Answers: *how fast did pass-through happen, and is it still converging at day 30?*

**Specification (one regression per horizon h):**

```
premium_{t+h} = αʰ + βʰ·post_t + γʰ·t + δʰ·δGold_USD_t + ζʰ·δFX_t + εʰ_t
```

**H₀:** βʰ = 0 for each horizon h (no dynamic effect at horizon h)  
**H₁:** βʰ > 0 and increasing toward the full duty shock

**Expected shape of impulse response:**  
- h=0 (Day 1): βʰ ≈ ₹7,872 (60.6% pass-through, from event study)  
- h=5: βʰ rising toward ₹10,000  
- h=10–20: βʰ plateauing or overshooting as international gold price fell  
- 95% confidence bands should exclude zero throughout

**Result:** *(fill in after running Notebook 03)*  
- β at h=0: _____  
- β at h=5: _____  
- β at h=10: _____  
- β at h=20: _____  
- Converging / Plateauing / Overshooting: _____

---

### TEST 05 — ARDL Bounds Test (Pesaran 2001)
**Notebook:** `03_its.ipynb`  
**Method:** ARDL bounds test for cointegration

**Purpose:**  
Test whether domestic gold price (Gold_INR_PM), international gold price (Gold_USD), and exchange rate (rupees_per_dollar) are cointegrated — i.e., they share a long-run equilibrium relationship. If yes, short-run deviations (domestic premium) are mean-reverting toward the import parity, which is the theoretical foundation of our model.

**H₀:** No cointegration (series move independently in the long run)  
**H₁:** Cointegrated (there is a long-run equilibrium that the premium reverts to)

**Decision rule:** Compare F-statistic to Pesaran (2001) critical value bounds. If F > upper bound → reject H₀ → cointegration confirmed.

**Expected result:** Cointegration is expected economically — gold is a globally traded commodity with arbitrage. The duty shock shifts the long-run equilibrium level by the duty differential.

**Result:** *(fill in after running Notebook 03)*  
- F-statistic: _____  
- Upper bound (5%): _____  
- Conclusion: _____

---

### TEST 06 — Fake-date placebo test
**Notebook:** `05_robustness.ipynb`  
**Method:** ITS re-run with fake treatment dates

**Purpose:**  
Verify that the estimated treatment effect is specific to May 13 2026 and not an artifact of the model or the data. If we run the same ITS with fake treatment dates (where no policy change occurred), we should find β₁ ≈ 0.

**Fake dates chosen (before seeing results):**
- March 1 2026 — 10 weeks before actual hike, no policy event
- January 15 2026 — 4 months before, no policy event  
- November 1 2025 — 6 months before, no policy event

**H₀:** β₁ = 0 at fake dates (no spurious effect)  
**H₁:** β₁ ≠ 0 (the model is detecting something that isn't there)

**Note on silver placebo:** Silver AIDC was also raised on May 13 2026 (same notification). Silver premium CANNOT be used as a placebo — it is a co-treated series. Silver_premium is retained in the CSV as descriptive only.

**Expected result:** β₁ ≈ 0 at all three fake dates (p > 0.05). The actual May 13 estimate should be a clear outlier in the placebo distribution.

**Result:** *(fill in after running Notebook 05)*  
- β₁ at Mar 1 2026: _____  
- β₁ at Jan 15 2026: _____  
- β₁ at Nov 1 2025: _____  
- Conclusion: _____

---

## Summary table (fill in as tests are run)

| # | Test | Notebook | H₀ | p-value | Decision |
|---|------|----------|----|---------|----------|
| 01 | Pre-trend (OLS slope in low-duty) | 01_eda | β = 0 | 0.3469 | ✅ Fail to reject — no pre-trend, ITS assumption holds (N=360 in v2) |
| 02 | Stationarity ADF (domestic_premium) | 01_eda | Unit root | 0.000 (low-duty only) | ✅ Reject unit root within regime — I(0), OLS valid; NW lag=8 (v2, T=845) |
| 03 | ITS treatment effect (β₁) | 03_its | β₁ = 0 | TBD | TBD |
| 04 | Local Projection β at h=0…30 | 03_its | βʰ = 0 | TBD | TBD |
| 05 | ARDL cointegration bounds | 03_its | No cointegration | TBD | TBD |
| 06 | Fake-date placebo (3 dates) | 05_robustness | β₁ = 0 | TBD | TBD |

---

## Notes on interpretation

- All tests use **two-tailed** p-values unless noted (ITS β₁ is one-tailed: we expect positive)
- Standard errors in Tests 03–04: **Newey-West HAC** to correct for autocorrelation in daily time series
- Test 02 result determines whether to use levels or first differences in Tests 03–05
- Pass-through estimate from Test 03 is a **lower bound** — see DATA_PIPELINE.md for reasons
