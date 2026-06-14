# India Gold Import Duty — Causal Analysis of Price Pass-Through

**Project:** gold-policy-project
**Type:** SSRN/arXiv Working Paper → Journal submission
**Status:** Planning
**Last updated:** June 2026

---

## Research Question

> How much of India's May 13, 2026 gold import duty hike (6% → 15%) passed through to the domestic price premium, through what channels did it operate, and what structural forces — exchange rate stress, smuggling leakage, and pre-existing supply disruption — governed the degree of pass-through?

---

## Policy Context

| Event | Date | Detail |
|---|---|---|
| Duty cut | Jul 23, 2024 | 15% → 6% (Union Budget) |
| Iran conflict escalation | ~Apr 2025 | Gold demand surge, INR pressure begins |
| Import restriction | Apr 2, 2026 | New customs/IGST rules — imports fell to ~15t (30-year low) |
| Rupee record low | May 12, 2026 | ₹95.63/USD |
| **Duty hike** | **May 13, 2026** | **6% → 15% (10% BCD + 5% AIDC), midnight notification** |

**Why the hike happened (dual rationale):**
1. Forex reserve protection — gold imports surged 24% to $71.98B in FY2026, consuming ~9% of all India imports; CAD widened to $30.1B (Apr–Dec 2025)
2. Demand compression — reduce gold consumption to narrow the trade deficit and stabilise the rupee

**The partial pass-through puzzle:**
Theoretical full pass-through at 9pp duty increase ≈ ₹8,500–9,000/10g. Observed in data ≈ ₹7,313. The gap is explained by three absorption mechanisms: smuggling leakage (estimated >100t in 2026, historical duty-smuggling correlation = +0.52), demand destruction (price elasticity −0.69 to −1.01), and scrap/recycling supply increase. This puzzle is the core contribution of the paper.

---

## Two-Channel Framework

| Channel | Mechanism | Identified by | Data status |
|---|---|---|---|
| Direct duty | Hike raises import parity → premium rises mechanically | β₁ in ITS (ΔFXR controlled out) | ✅ Have |
| FX stabilisation | Duty → fewer imports → less CAD → rupee stabilises | Event study on INR/USD volatility pre/post May 13 | ✅ Have |

Because the ITS model controls for ΔFXR_t, β₁ isolates the direct duty effect net of any FX movement. The FX channel is then separately identified via the event study.

---

## Project Structure

```
gold-policy-project/
├── data/                          ← raw + processed CSVs
├── notebooks/
│   ├── 00_data_pipeline.ipynb     ← fetch all series, construct premium, save CSV
│   ├── 01_eda.ipynb               ← visual evidence, distributions, event window
│   ├── 02_hypothesis.md           ← precommit: written BEFORE any model runs
│   ├── 03_causal.ipynb            ← ITS + local projection + ARDL cointegration
│   ├── 04_experiments/
│   │   ├── arima.ipynb            ← counterfactual: what would gold have done?
│   │   ├── xgboost.ipynb          ← feature importance, premium drivers
│   │   ├── prophet.ipynb          ← trend decomposition, structural break
│   │   ├── finbert.ipynb          ← news sentiment, anticipation test
│   │   └── garch.ipynb            ← volatility regime pre/post hike
│   └── 05_robustness.ipynb        ← placebo, sensitivity, SGB check, leakage test
├── charts/                        ← publication-ready figures
├── paper/                         ← LaTeX draft
└── OUTLINE.md                     ← this file
```

---

## Data Plan

### Series to fetch in notebook 00

| Series | Source | Ticker / URL | Frequency | Status | Notes |
|---|---|---|---|---|---|
| Gold spot (USD/oz) | Yahoo Finance | `GC=F` | Daily | ✅ Have | |
| INR/USD exchange rate | Yahoo Finance | `INR=X` | Daily | ✅ Have | |
| IBJA benchmark (₹/10g) | GoodReturns scrape | goodreturns.in | Daily | ✅ Have | 16 NaN days (IBJA closed) — do NOT ffill |
| Crude oil Brent (USD/bbl) | Yahoo Finance | `BZ=F` | Daily | ✅ Have | |
| RBI forex reserves ($B) | RBI weekly release | rbi.org.in | Weekly → ffill | ✅ Have | |
| Kalyan Jewellers stock | Yahoo Finance | `KALYANKJIL.NS` | Daily | ✅ Have | |
| **Silver spot (USD/oz)** | Yahoo Finance | `SI=F` | Daily | ❌ Fetch | Same 6%→15% hike — natural comparison |
| **MCX gold futures (near-month)** | Yahoo Finance / NSE | `GOLDM.MCX` | Daily | ❌ Fetch | Basis / forward expectations |
| FPI equity flows | NSDL / SEBI | sebi.gov.in | Daily | ❌ Lower priority | Capital account pressure |
| Gold imports monthly ($B, t) | DGCI&S | commerce.gov.in | Monthly | ⚠️ Apr 2026 latest | May data ~6wk lag; use for narrative only |

### Constructed variables

| Variable | Formula | Purpose |
|---|---|---|
| `parity_6pct` | Gold_USD × (10/31.1035) × FX × 1.06 | Pre-policy counterfactual |
| `parity_15pct` | Gold_USD × (10/31.1035) × FX × 1.15 | Post-policy landed cost |
| `domestic_premium` | IBJA − parity_6pct | **Main outcome variable** |
| `premium_pct` | domestic_premium / parity_6pct | Normalised outcome |
| `post_hike` | 1 if date ≥ 2026-05-13 | Treatment dummy |
| `days_since_hike` | Trading days since May 13 | Post-period time trend |
| `δGold_USD` | Day-over-day % change | Global gold control |
| `δFX` | Day-over-day change in INR/USD | FX control |
| `δOil` | Day-over-day change in Brent | Oil/CAD control |
| `silver_premium` | Silver_INR_actual − silver_parity_6pct | Comparison outcome |

### ⚠️ Data window — critical note

For ARDL cointegration tests, minimum 3–5 years of daily data is recommended. Current CSV covers ~1.5 years (Jan 2025 – Jun 2026), which is borderline. **Notebook 00 should fetch from January 2022** for all available series. This extends the pre-treatment window and makes cointegration tests meaningful.

---

## Notebook Guide

### 00 — Data Pipeline

**Purpose:** Single source of truth. No charts, no models, no analysis.
**Output:** `data/gold_policy_clean.csv` + inline data dictionary

Steps:
1. Fetch all series from Jan 2022 to present (where available)
2. Align to common trading calendar (union of NSE + IBJA open days)
3. Handle IBJA-closed days — set to NaN, do NOT forward-fill
4. Construct all premium and return variables
5. Add silver series alongside gold
6. Document every column in a markdown data dictionary cell at top
7. Print: shape, date range, NaN counts per column, basic describe()

---

### 01 — EDA

**Purpose:** Look before you model. Produce 5–7 charts for the paper. Find every anomaly.

Sequence:
1. Univariate — distributions of premium, returns, FX; check for outliers
2. Temporal — full time series of all key variables; mark key dates (Jul 2024 cut, Apr 2 restriction, May 13 hike)
3. Event window — zoom ±30 days around May 13; was the jump immediate or gradual?
4. Silver comparison — does silver premium mirror gold premium post-hike?
5. Anticipation check — did premium start moving before May 13? (3–5 day pre-window)
6. INR/USD post-hike — did rupee stabilise or keep falling? Visual FX channel test
7. Correlation matrix — pre-hike vs post-hike; did the premium-FX relationship change?

---

### 02 — Hypothesis (precommit document)

**Purpose:** Written record of exact hypotheses before any regression. Non-negotiable discipline.

Must include:
- H₀ and H₁ for each test (ITS, local projection, ARDL, GARCH, silver)
- Expected sign and approximate magnitude of each coefficient
- Expected shape of local projection path — immediate jump and flat? gradual build? partial reversion?
- Expected ARDL result — cointegrated pre-hike, broken post-hike?
- What a null result would look like and what it would mean for the paper

---

### 03 — Causal Analysis

**Purpose:** Main empirical result. Three complementary models.

**Model A — ITS Regression (point estimate)**

```
premium_t = α + β₁·post_t + β₂·t + β₃·δGold_USD_t + β₄·δFX_t + ε_t
```

- SE: Newey-West HAC (lag=5)
- Run 3 specs: baseline OLS → + FX control → + time trend (3-column table)
- β₁ is the headline result

**Model B — Local Projection (time path)**

For each horizon h = 1, 2, ..., 30 trading days post-hike:
```
premium_{t+h} − premium_t = α_h + β_h·post_t + controls + ε_{t+h}
```

Plot β_h over h. Shows whether pass-through was immediate, gradual, or partial and reverting. This is what Federal Reserve uses for tariff analysis. More informative than a single ITS dummy.

**Model C — ARDL Cointegration (permanent vs temporary)**

- Bounds test: are domestic_premium and parity_6pct cointegrated pre-hike?
- If yes: estimate error correction coefficient (speed of mean reversion)
- Test structural break: does the cointegrating relationship hold post-hike?
- Interpretation: cointegration breaks → permanent regime change; holds → temporary shock being absorbed

**Silver comparison:** Run Model A on silver premium. Compare β₁ to gold. Same policy treatment, independent series — natural robustness check.

---

### 04 — Experiments

**MLflow setup (first cell in every experiment notebook):**

```python
import mlflow
mlflow.set_tracking_uri("../mlruns")
mlflow.set_experiment("gold-policy-paper1")
# then: mlflow.start_run(), log_params(), log_metrics(), log_artifact()
```

**arima.ipynb — Counterfactual**
Train ARIMA on pre-hike window → forecast into post-hike → gap between forecast and actual = counterfactual estimate of hike effect. Compare with ITS β₁ as cross-check.

**xgboost.ipynb — Feature Importance**
Train XGBoost on pre-hike premium drivers → test out-of-distribution on post-hike data → if R² collapses post-hike, structural break confirmed. SHAP values identify which variables matter most for premium dynamics.

**prophet.ipynb — Trend Decomposition**
Fit Prophet with changepoint prior at May 13. Decompose trend vs seasonality vs residual. Compare trend shift magnitude with ITS β₁.

**finbert.ipynb — News Sentiment**
Fetch 6+ months of gold/India policy news headlines (pre and post hike). Run FinBERT sentiment classification. Test:
- Did sentiment shift sharply around May 13 or gradually before?
- Does sentiment lead the premium (anticipation) or lag it (reaction)?
- Is the Iran war period already driving a negative-sentiment baseline?
A null result (sentiment not predictive) strengthens the causal claim — the premium jump is policy-driven, not sentiment-driven.

**garch.ipynb — Volatility Regime**
Fit GARCH(1,1) on premium returns. Test whether conditional volatility increased or decreased post-hike.
- Higher volatility post-hike → market still discovering new equilibrium
- Lower volatility post-hike → new premium regime accepted and stable

---

### 05 — Robustness

**Purpose:** Try to falsify the main result from 03_causal.

Tests:
1. **Placebo** — substitute Mar 1, Jan 15, Nov 1 as fake treatment dates; β₁ should be near zero
2. **NaN sensitivity** — include vs exclude the 16 IBJA-closed days
3. **Window sensitivity** — vary pre-period start (Jan 2022 vs Jan 2025 vs post-war only)
4. **Alternative SE** — plain OLS, clustered, HAC lag=3, HAC lag=10
5. **Anticipation test** — drop 5 trading days before May 13 from the pre-period
6. **Apr 2 restriction** — add a dummy for the pre-hike supply squeeze; does β₁ change?
7. **SGB check** — did Sovereign Gold Bond secondary spreads widen post-hike? (investment demand shift)
8. **Leakage test** — did INR start recovering before or after May 13? (direction of FX channel)
9. **Price elasticity cross-check** — implied demand destruction from β₁ vs elasticity literature (−0.69 to −1.01)

---

## Methodology Reference

**ITS Regression**
Standard approach for sharp policy discontinuities. HAC standard errors (Newey-West) correct for autocorrelation and heteroskedasticity in daily financial time series.

**Local Projection (Jordà 2005)**
Estimates the impulse response of the premium to the duty shock at each horizon h separately. More flexible than VAR, robust to model misspecification. Now standard for tariff pass-through analysis (see Federal Reserve FEDS Notes 2025/2026).

**ARDL Bounds Test (Pesaran, Shin & Smith 2001)**
Tests for cointegration without requiring series to be the same order of integration. Appropriate for mixed I(0)/I(1) financial series. Identifies whether the premium shift is permanent (no cointegration post-hike) or mean-reverting (still cointegrated).

**GARCH(1,1)**
Models time-varying conditional volatility. Identifies whether the duty shock created a volatility regime change in the premium series.

**MLflow**
Open-source experiment tracking. Logs parameters, metrics, and model artifacts for every run in 04_experiments. Enables clean comparison across specifications without losing track of versions.

---

## Paper Outline

1. **Introduction** — Dual policy motivation (forex + demand), the partial pass-through puzzle, research gap (no empirical smuggling-premium study exists), result preview
2. **Policy Background** — Jul 2024 duty cut, FY26 import surge ($71.98B), rupee at ₹95.80, Apr 2 supply restriction, May 13 midnight notification
3. **Data** — IBJA prices, premium construction, silver comparison, trading calendar, summary statistics
4. **Empirical Strategy** — ITS design, two-channel framework, local projection extension, ARDL test, HAC SE rationale, identifying assumptions
5. **Results**
   - 5a. Direct duty channel — ITS β₁ (3-model table), headline estimate
   - 5b. Time path — local projection impulse response (30-day window)
   - 5c. FX stabilisation — event study on INR/USD volatility around May 13
   - 5d. Silver comparison — same ITS design on silver premium
6. **Robustness** — Placebo, window sensitivity, anticipation test, Apr 2 control, SGB check
7. **Discussion** — Partial pass-through puzzle decomposed (smuggling ~₹X, demand destruction ~₹Y, scrap supply ~₹Z), policy implications, historical comparison (2013 hike episode)
8. **Conclusion** — Summary, limitations, Paper 2 preview (market efficiency, MCX futures basis)

**Target:** 25–30 pages | SSRN first → arXiv → Journal of Development Economics / Economics Letters / India Policy Forum

---

## Key References

- Pesaran, Shin & Smith (2001) — ARDL bounds test methodology
- Jordà (2005) — Local projection method
- Fajgelbaum et al. (2019, NBER w25638) — Return to protectionism; DiD tariff pass-through
- JRFM (2024) — Gold smuggling in India and bullion market effects [doi:10.3390/jrfm17030122]
- ScienceDirect (2016) — Elasticity of gold import demand in India (price elasticity: −0.69 to −1.01)
- Federal Reserve FEDS Notes (Apr/May 2026) — Detecting tariff effects in real time via local projection
- WGC India Gold Market Updates (Jul 2024, May 2026)
- CNBC (May 13, 2026) — India hikes bullion import duties to arrest rupee slide
- Business Standard (May 14, 2026) — Why higher gold import duty alone may not ease CAD pressure

---

## Known Gaps & Open Questions

| Gap | Impact on paper | Mitigation |
|---|---|---|
| DGCI&S May/Jun 2026 import volumes not yet published | Can't directly measure post-hike demand destruction | Back-calculate using price elasticity literature |
| Smuggling — no clean daily series | Can't directly control for leakage in regression | Narrative explanation; cite JRFM 2024 (corr=0.52) |
| ARDL needs 3–5yr history | Cointegration test may be underpowered with 1.5yr | Fetch from Jan 2022 in notebook 00 |
| MCX futures data availability | Can't analyse forward curve / basis | Try Yahoo Finance GOLDM.MCX; note if unavailable |
| FPI daily flows | Can't fully control capital account pressure | Use monthly SEBI data as narrative; lower priority |
| Apr 2 supply restriction (pre-treatment) | Could confound May 13 treatment | Add Apr 2 dummy in robustness; check pre-trend |

---

*Last updated: June 2026*
