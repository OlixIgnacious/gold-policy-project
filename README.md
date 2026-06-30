# India Gold Import Duty — Pass-Through Study

**Type:** Working paper (SSRN/arXiv → journal)  
**Status:** Phase 3 complete — EDA, causal analysis (ITS, Local Projection, ARDL cointegration), and robustness suite all done; now experimenting with ARIMA forecasting (Notebook 04)  
**Data coverage:** Jan 3, 2022 → Jun 16, 2026 (1,160 trading days × 29 columns)  
**Dataset version:** v2 (IBJA PDF: 815 rows + BullionWorld gap-fill: 105 rows — see `data/VERSIONS.md`)

---

## Research Question

> How much of India's May 13, 2026 gold import duty hike (6% → 15%) passed through to the domestic price premium, through what channels did it operate, and what structural forces — exchange rate stress, smuggling leakage, and pre-existing supply disruption — governed the degree of pass-through?

**Headline finding (causal, primary spec):** The duty hike raised the domestic gold premium by **₹9,743/10g** (Newey-West HAC SE ₹589, p=1.96e-61, N=368) — **79.4% pass-through** [95% CI: 69.9%, 88.8%]. Partial pass-through (vs. 100%) is statistically confirmed (t=−4.30). Local projection shows the jump was immediate (Day 1) and held on a flat plateau for ~14 days, not a gradual build-up. The ARDL bounds test confirms domestic and international gold prices were cointegrated pre-hike (F=9.005), meaning the hike shifted the long-run equilibrium rather than producing a temporary spike. A fake-date placebo test (3 dates) found no spurious effect, confirming the result is specific to May 13, 2026.

---

## Policy Background

| Event | Date | Detail |
|-------|------|--------|
| Duty cut | Jul 23, 2024 | 15% → 6% (Union Budget) |
| Bank IGST pause | Apr–May 2026 | 17 banks halted bullion imports; imports fell to ~15t (30-year low) |
| UAE supply disruption | Mar–May 2026 | Re-export route disrupted; negative premiums in April |
| Rupee record low | May 12, 2026 | ₹95.63/USD |
| **Duty hike** | **May 13, 2026** | **6% → 15% (10% BCD + 5% AIDC), midnight notification** |

The hike was driven by dual pressures: gold imports surged to $71.98B in FY2026 (~9% of all imports) and CAD widened to $30.1B (Apr–Dec 2025).

---

## Key Findings So Far

### Parity validation
When duty was 6% (Jul 2024 – May 2026), the mean domestic premium was −₹61 (N=360 IBJA days, v2) — essentially zero. This confirms the parity formula is correctly calibrated.

### Post-hike pass-through trajectory

| Window | N valid | Mean Pass-through | Interpretation |
|--------|---------|------------------|----------------|
| Week 1 (obs 1–5) | 5 | 73.2% | Old 6%-duty inventory in supply chain |
| Week 2 (obs 6–10) | 5 | 77.8% | Gradual convergence as inventory depletes |
| Week 3 (obs 11–15) | 5 | 78.0% | Plateau — seasonal demand weakness, smuggling |
| Week 4 (obs 16–20) | 5 | 107.2% | Overshoot — Gold_USD fell ~12%, ceiling shrank, IBJA sticky |
| Tail (obs 21–23) | 3 | 80.7% | Pull-back as Gold_USD partially recovers |
| **Overall** | **23** | **83.6%** | **Headline estimate (lower bound)** |

### Pre-trend test (TEST 01)
Slope in low-duty window: +₹0.78/day, p=0.35 (N=360, v2) — no pre-existing trend. ITS assumption holds.

### Stationarity (TEST 02)
`domestic_premium` is I(0) within regimes (ADF p=0.000 in low-duty window). Full-sample non-stationarity is a structural break artifact, not a true unit root. `Gold_USD` and `rupees_per_dollar` are I(1) → ARDL bounds test valid for Notebook 03. Newey-West HAC lag = 8 (v2, T=845 regression sample).

### ITS treatment effect (TEST 03)
Primary spec (Jul 2024+ low-duty window, N=368): β₁ = **₹9,743/10g** (HAC SE ₹589, p=1.96e-61) → **79.4% pass-through** [CI: 69.9%, 88.8%]. Full-window robustness spec: ₹10,107/10g (82.3% PT). H₀ of 100% pass-through is rejected (t=−4.30) — 20.6% is absorbed by smuggling leakage, demand destruction, and scrap supply.

### Local projection impulse response (TEST 04)
Pass-through is immediate, not gradual: β_h jumps to ₹9,743 (79.4%) on Day 1 (h=0), peaks at ₹9,995 (81.4%) on Day 2, then holds a flat plateau through h=14 (78.2%). 95% CI excludes zero throughout h=0–19. Apparent late-horizon decline (h≥19) is a sample-thinning artifact, not economic reversion.

### ARDL cointegration (TEST 05)
Domestic gold price, international gold price, and the exchange rate are cointegrated pre-hike (ARDL(4,3), F=9.005 vs. 95% upper bound 4.812, N=859). The duty hike therefore shifted the long-run equilibrium permanently rather than causing a temporary deviation that would mean-revert.

### Robustness (TEST 06, Notebook 05)
Fake-date placebo (3 pre-hike dates) found no significant effect (p > 0.10 all dates) vs. the real May 13 estimate (p < 0.001) — a 4–11× larger outlier. β₁ is stable across NW lag choices (3/6/8/10/20) and window starts (Jan 2022/Jan 2024/Jul 2024), with no evidence of anticipation effects or confounding from the Apr 2 import restriction.

---

## Project Structure

```
gold-policy-project/
├── data/
│   ├── gold_policy_clean.csv          ← PRIMARY DATASET, v2 (1160 × 29)
│   ├── gold_policy_v1.csv             ← Frozen v1 snapshot (pre-BullionWorld gap-fill)
│   ├── gold_policy_v2.csv             ← Frozen v2 snapshot (current)
│   ├── VERSIONS.md                    ← v1 → v2 changelog and how to switch versions
│   ├── versions.json                  ← Machine-readable version metadata
│   ├── ibja_raw.csv                   ← IBJA-only panel (all fields)
│   ├── ibja_missing_dates.txt         ← Dates with no IBJA PDF
│   ├── ibja_missing_dates.csv         ← Same, structured for analysis
│   ├── bullionworld_full.csv          ← Raw BullionWorld scrape (gap-fill source)
│   ├── bullionworld_recovered.csv     ← BullionWorld rows matched to archive gaps
│   ├── IBJA_Market_Calendar_133_Dates.xlsx  ← Classification of still-missing dates
│   ├── ibja_pdfs/                     ← 814 PDFs (gitignored, 862MB)
│   ├── rbi_forex_reserves_weekly.xlsx
│   ├── rbi_fx_intervention_monthly.xlsx
│   ├── rbi_foreign_trade_monthly.xlsx
│   └── gazette/                       ← 4 CBIC customs notifications (legal text)
├── notebooks/
│   ├── 00_data_pipeline.ipynb         ← ✅ DONE — fetch, merge, construct premium, save CSV
│   ├── 01_eda.ipynb                   ← ✅ DONE — distributions, event window, correlations, outliers
│   ├── 03_causal.ipynb                ← ✅ DONE — ITS + local projection + ARDL bounds test
│   ├── 04_experiments/                ← 🔬 IN PROGRESS — exploratory ARIMA forecasting
│   └── 05_robustness.ipynb            ← ✅ DONE — placebo, NW lag/window sensitivity, anticipation test
├── figures/                           ← PNG charts from EDA
├── paper/                             ← LaTeX draft (empty)
├── OUTLINE.md                         ← Full methodology and paper outline
├── DATA_PIPELINE.md                   ← Data sources, column dictionary, QC notes
├── findings_of_eda.md                 ← Cell-by-cell EDA findings
└── 02_hypothesis.md                   ← Pre-registered hypotheses (written before model runs)
```

---

## Data

**Primary source:** IBJA daily PDF reports (815 files) — gold/silver INR benchmark rates, MCX OHLCV, London Fix  
**Gap-fill source:** BullionWorld.in (IBJA data republisher) — 105 rows recovered for archive-gap dates (v2)  
**Secondary sources:** Yahoo Finance (Gold_USD, INR/USD, Nifty50, GOLDBEES), RBI DBIE (forex reserves, FX intervention)  
**Gazette:** Four CBIC customs notifications (No. 15–18/2026-Customs, dated May 12, 2026)

The `ibja_source` column tracks per-row provenance: `pdf` (815) | `bullionworld` (105) | NaN (240, holidays + unrecovered gaps). See `data/VERSIONS.md` for the full v1 → v2 changelog and 133 still-missing dates pending IBJA's response.

### Duty structure

| Period | BCD | AIDC | **Total** |
|--------|-----|------|-----------|
| Jan 2022 – Jul 23, 2024 | 10% | 5% | **15%** |
| Jul 24, 2024 – May 12, 2026 | 5% | 1% | **6%** |
| May 13, 2026+ | 10% | 5% | **15%** |

### Parity construction
```
parity_pre  = Gold_USD × (10/31.1035) × INR/USD × 1.06   # ITS baseline (fixed)
parity_post = Gold_USD × (10/31.1035) × INR/USD × 1.15   # Theoretical ceiling
domestic_premium = Gold_INR_PM − parity_pre                # Primary outcome
```

---

## Empirical Strategy

**Model A — ITS (point estimate)** ✅
```
premium_t = α + β₁·post_t + β₂·t + β₃·ΔGold_USD_t + β₄·ΔFXR_t + ε_t
```
Newey-West HAC SE (lag=6, primary spec). β₁ = ₹9,743/10g (79.4% pass-through) is the headline result.

**Model B — Local Projection (Jordà 2005) — time path** ✅  
Separate OLS at each horizon h=0…30. Confirms pass-through was immediate (Day 1) and plateaued, not gradual or overshooting.

**Model C — ARDL Bounds Test (Pesaran 2001) — permanent vs temporary** ✅  
Tests cointegration between domestic gold price, international gold, and exchange rate. Mixed I(0)/I(1) system. F=9.005 confirms cointegration — the shift is permanent.

**Robustness (Notebook 05)** ✅: Fake-date placebo (3 dates), window sensitivity, HAC lag sensitivity, Apr 2 restriction control, anticipation test — all confirm the primary estimate is stable and not spurious.

**Experiments (Notebook 04, in progress):** Exploratory ARIMA/SARIMAX forecasting of `domestic_premium` with exogenous regressors (ΔGold_USD, ΔFX), order-selected via `pmdarima.auto_arima`.

---

## Setup

```bash
uv sync          # install dependencies from pyproject.toml
```

Run notebooks in order: `00_data_pipeline.ipynb` → `01_eda.ipynb` → `03_causal.ipynb` → `05_robustness.ipynb` → `04_experiments/` (exploratory, ARIMA forecasting)

> **Note:** `data/ibja_pdfs/` (814 PDFs, 862MB) is gitignored. Run Notebook 00, Cell 3c to re-download if needed. The parsed `ibja_raw.csv` and `gold_policy_clean.csv` are committed and sufficient to reproduce all analysis without the PDFs.

---

## References

- Pesaran, Shin & Smith (2001) — ARDL bounds test
- Jordà (2005) — Local projection method
- Fajgelbaum et al. (2019, NBER w25638) — Return to protectionism; DiD tariff pass-through
- JRFM (2024) — Gold smuggling in India and bullion market effects
- Federal Reserve FEDS Notes (Apr/May 2026) — Detecting tariff effects via local projection
- WGC India Gold Market Updates (Jul 2024, May 2026)
