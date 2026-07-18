# India Gold Import Duty — Pass-Through Study

**Type:** Working paper (arXiv/SSRN → journal)
**Status:** Paper complete — LaTeX draft compiled ([paper/main.pdf](paper/main.pdf)); arXiv/SSRN submission package prepared (see `submission_package.md` and `arxiv_submission.zip`)
**Data coverage:** Jan 3, 2022 → Jun 30, 2026 (1,170-row weekday spine × 30 columns, v4)
**Dataset version:** v4 — 929 daily price observations (824 IBJA PDF circulars + 105 BullionWorld.in gap-fills; see `data/VERSIONS.md`)

---

## Research Question

> How much of India's May 13, 2026 gold import duty hike (6% → 15%) passed through to the domestic price premium, through what channels did it operate, and what structural forces — exchange rate stress, smuggling leakage, and pre-existing supply disruption — governed the degree of pass-through?

**Headline finding (causal, primary spec):** The duty hike raised the domestic gold premium by **₹9,841/10g** (Newey-West HAC SE ₹516, p<0.001, N=375, Pre=346, Post=29) against a theoretical ceiling of ₹12,039 — **81.7% pass-through** [95% CI: 73.3%, 90.1%]. Full pass-through is formally rejected (t=−4.26, p<0.001). Local projections show the adjustment was immediate on the first trading day and held on a stable plateau through Day 14 (78.2%), with no anticipatory pricing. The ARDL bounds test (ARDL(4,3), F=9.005, N=859) confirms domestic and international gold prices were cointegrated pre-hike. Placebo tests (three fake dates, all non-significant; cleanly null on truncated samples) and ML counterfactuals (ARIMAX/XGBoost within ~5% of the ITS estimate) confirm the result is specific to May 13, 2026.

**Asymmetry finding:** The hike passed through more completely (81.7%) than the July 2024 duty cut did (65.6%, 95% CI: 53.1–78.1%), with both adjusting at Day 0 — consistent with downward rigidity ("rockets and feathers") in duty pass-through.

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

## Key Findings

### ITS treatment effect (primary spec)
Jul 2024+ window, N=375 (Pre=346, Post=29): β₁ = **₹9,841/10g** (HAC SE ₹516, NW lag=6) → **81.7% pass-through** [CI: 73.3%, 90.1%] against a mean ceiling of ₹12,039. H₀ of 100% pass-through is rejected (t=−4.26) — the remaining ~18% is absorbed by front-loaded low-duty inventory, seasonal demand weakness, and smuggling leakage.

### Pre-trend test
Pre-treatment trend: +₹1.40/day, p=0.12 — no evidence of pre-existing drift. ITS assumption holds.

### Local projection impulse response (Jordà 2005)
Pass-through is immediate, not gradual: the premium reaches 81.7% of the ceiling on Day 0 and holds a stable plateau through Day 14 (78.2%). No anticipatory pricing in the pre-hike window.

### ARDL cointegration (Pesaran 2001)
Domestic and international gold prices were cointegrated pre-hike (ARDL(4,3), F=9.005, N=859). The stable local-projection plateau is consistent with the duty hike shifting the long-run equilibrium rather than producing a temporary spike.

### Robustness
- **Fake-date placebos:** all three non-significant — Nov 1, 2025 (β₁=−₹932, p=0.38), Jan 15, 2026 (₹2,004, p=0.11), Mar 1, 2026 (₹2,909, p=0.06); the largest is <30% of the true estimate. On truncated samples ending May 12 (no post-hike contamination), all placebos are cleanly null.
- **Apr 2 import restriction control:** adding it changes β₁ by −0.8% (₹9,758); the restriction coefficient itself is indistinguishable from zero.
- **Counterfactual ML checks:** ARIMAX and XGBoost counterfactual gaps (~₹10,250) converge within ~5% of the ITS estimate.
- β₁ is stable across NW lag choices and window starts.

### Asymmetric transmission (hike vs. cut)
Symmetric ITS on the Jul 2024 cut: β₁ = ₹5,844 (HAC SE ₹568) against a ceiling of ₹8,911 → 65.6% pass-through, vs. 81.7% for the hike (CIs overlap, so the difference is suggestive rather than statistically significant). The cut's dynamic path also erodes gradually (to ~55% by Day 20) while the hike plateaus immediately — dealers holding high-duty inventory resisted passing the cut through.

---

## Project Structure

```
gold-policy-project/
├── data/
│   ├── gold_policy_clean.csv          ← PRIMARY DATASET, v4 (1170 × 30)
│   ├── gold_policy_v1.csv … v4.csv    ← Frozen version snapshots
│   ├── VERSIONS.md                    ← Version changelog and how to switch versions
│   ├── versions.json                  ← Machine-readable version metadata
│   ├── its_results.json               ← Headline ITS estimates (machine-readable)
│   ├── ibja_raw.csv                   ← IBJA-only panel (all fields)
│   ├── bullionworld_full.csv          ← Raw BullionWorld scrape (gap-fill source)
│   ├── bullionworld_recovered.csv     ← BullionWorld rows matched to archive gaps
│   ├── ibja_pdfs/                     ← 814 PDFs (gitignored, 862MB)
│   ├── rbi_*.xlsx                     ← RBI forex reserves / FX intervention / trade
│   └── gazette/                       ← 4 CBIC customs notifications (legal text)
├── notebooks/
│   ├── 00_data_pipeline.ipynb         ← ✅ fetch, merge, construct premium, save CSV
│   ├── 01_eda.ipynb                   ← ✅ distributions, event window, correlations, outliers
│   ├── 03_causal.ipynb                ← ✅ ITS + local projection + ARDL bounds test
│   ├── 04_experiments/                ← ✅ ARIMAX, XGBoost, Prophet, FinBERT, GARCH
│   ├── 05_robustness.ipynb            ← ✅ placebo, NW lag/window sensitivity, anticipation test
│   └── 06_methodology_robustness.ipynb ← ✅ truncated placebos, asymmetry (hike vs. cut), extras
├── paper/
│   ├── main.tex                       ← LaTeX source (19 pages, 4 figures)
│   ├── main.pdf                       ← Compiled paper
│   ├── IndiaGoldQuasiExp.pdf          ← Distribution copy of the compiled paper
│   ├── abstract.tex, references.bib   ← Abstract and bibliography
│   └── fig_*.png                      ← Paper-only figures
├── figures/                           ← PNG charts from EDA
├── arxiv_submission.zip               ← Ready-to-upload arXiv bundle (tex + bbl + 4 figs)
├── submission_package.md              ← arXiv/SSRN checklists, metadata, journal targets
├── OUTLINE.md                         ← Full methodology and paper outline
├── DATA_PIPELINE.md                   ← Data sources, column dictionary, QC notes
├── findings_of_eda.md                 ← Cell-by-cell EDA findings
└── 02_hypothesis.md                   ← Pre-registered hypotheses (written before model runs)
```

---

## Data

**Primary source:** IBJA daily PDF reports (824 files with usable PM gold rates) — gold/silver INR benchmark rates, MCX OHLCV, London Fix
**Gap-fill source:** BullionWorld.in (IBJA data republisher) — 105 archive-gap dates recovered and validated (v4)
**Secondary sources:** Yahoo Finance (Gold_USD, INR/USD, Nifty50, GOLDBEES), RBI DBIE (forex reserves, FX intervention)
**Gazette:** Four CBIC customs notifications (No. 15–18/2026-Customs, dated May 12, 2026)

The `ibja_source` column tracks per-row provenance: `pdf` (824) | `bullionworld` (105) | NaN (241, market holidays + unrecovered archive gaps). See `data/VERSIONS.md` for the full version changelog.

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
Newey-West HAC SE (lag=6, primary spec T=375). β₁ = ₹9,841/10g (81.7% pass-through) is the headline result.

**Model B — Local Projection (Jordà 2005) — time path** ✅
Separate OLS at each horizon h=0…30. Confirms pass-through was immediate (Day 0) and plateaued, not gradual or overshooting.

**Model C — ARDL Bounds Test (Pesaran 2001) — permanent vs temporary** ✅
Tests cointegration between domestic gold price, international gold, and exchange rate. Mixed I(0)/I(1) system. F=9.005 (ARDL(4,3), N=859) confirms pre-hike cointegration.

**Robustness (Notebooks 05–06)** ✅: Fake-date placebos (full-sample + truncated), window/HAC-lag sensitivity, Apr 2 restriction control, anticipation test, hike-vs-cut asymmetry, GARCH volatility, and ARIMAX/XGBoost/Prophet counterfactuals — all confirm the primary estimate is stable and not spurious.

---

## Paper & Submission

- **Compiled paper:** [paper/main.pdf](paper/main.pdf) (19 pages, 4 figures) — also saved as [paper/IndiaGoldQuasiExp.pdf](paper/IndiaGoldQuasiExp.pdf)
- **arXiv bundle:** `arxiv_submission.zip` (main.tex + abstract.tex + main.bbl + 4 figures) — target categories econ.GN (primary), econ.EM (cross-list)
- **Submission guide:** `submission_package.md` — step-by-step arXiv/SSRN checklists, copy-paste metadata (title, abstract, JEL codes E31/F13/H22/Q02, keywords), and journal targets with a cover-letter template

---

## Setup

```bash
uv sync          # install dependencies from pyproject.toml
```

Run notebooks in order: `00_data_pipeline.ipynb` → `01_eda.ipynb` → `03_causal.ipynb` → `05_robustness.ipynb` → `06_methodology_robustness.ipynb` → `04_experiments/`

> **Note:** `data/ibja_pdfs/` (814 PDFs, 862MB) is gitignored. Run Notebook 00, Cell 3c to re-download if needed. The parsed `ibja_raw.csv` and `gold_policy_clean.csv` are committed and sufficient to reproduce all analysis without the PDFs.

---

## References

- Pesaran, Shin & Smith (2001) — ARDL bounds test
- Jordà (2005) — Local projection method
- Newey & West (1987) — HAC standard errors
- Nakamura & Steinsson (2018) — Identification via policy discontinuities
- Bernal, Cummins & Gasparrini (2017) — Interrupted time series for public policy
- Fajgelbaum et al. (2019, NBER w25638) — Return to protectionism; DiD tariff pass-through
- JRFM (2024) — Gold smuggling in India and bullion market effects
- Federal Reserve FEDS Notes (Apr/May 2026) — Detecting tariff effects via local projection
- WGC India Gold Market Updates (Jul 2024, May 2026)
