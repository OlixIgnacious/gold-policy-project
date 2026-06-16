# Data Pipeline Documentation
## Gold Import Duty Pass-Through Study — India 2022–2026

**Notebook:** `notebooks/00_data_pipeline.ipynb`
**Output:** `data/gold_policy_clean.csv`
**Last rebuilt:** June 2026

---

## 1. Overview

The pipeline assembles a daily panel of gold price, macroeconomic, and policy variables spanning January 3, 2022 through June 12, 2026. The treatment event is India's gold import duty hike from 6% to 15%, effective **May 13, 2026**, enacted via four CBIC customs notifications (No. 15–18/2026-Customs, dated 12 May 2026).

**Final dataset:** 1,158 trading days × 25 columns

---

## 2. Sources

| Source | Series | Frequency | Coverage |
|--------|---------|-----------|----------|
| IBJA daily PDF reports | Gold/Silver INR price (AM + PM), MCX OHLCV, London Fix, COMEX, CFTC, DXY, NSE FX, India/US 10Y bonds, SPDR/iShares ETF holdings | Daily (trading days) | May 2022 – Jun 2026 (gaps documented below) |
| Yahoo Finance (`yfinance`) | Gold_USD (GC=F), Silver_USD (SI=F), Oil_USD (BZ=F), INR/USD (INR=X), Kalyan Jewellers (KALYANKJIL.NS), Nifty50 (^NSEI), US 10Y yield (^TNX), GOLDBEES ETF (GOLDBEES.NS) | Daily | Jan 2022 – Jun 2026 |
| RBI DBIE Table 33 | Total forex reserves (USD bn, weekly) | Weekly → forward-filled daily | Jan 2022 – May 2026 |
| RBI DBIE Table 04 | Net USD purchase/sale by RBI (USD mn, monthly) | Monthly | Jun 1995 – Mar 2026 |
| CBIC Gazette (via TaxGuru) | Four duty notifications — legal text and effective dates | One-time | May 12, 2026 |

---

## 3. Duty Structure (Gazette-Verified)

Verified directly from four customs notifications dated 12 May 2026, effective 13 May 2026.

| Period | BCD | AIDC | SWS | **Total** | DUTY_MULT |
|--------|-----|------|-----|-----------|-----------|
| Jan 2022 – Jul 23, 2024 | 10% | 5% | 0% | **15%** | 1.15 |
| Jul 24, 2024 – May 12, 2026 | 5% | 1% | 0% | **6%** | 1.06 |
| May 13, 2026+ | 10% | 5% | 0% | **15%** | 1.15 |

**Legal citations:**
- Notification 15/2026-Customs (G.S.R. 358(E)) — BCD raised 5%→10% on gold/silver (S.Nos 192–197, 203; amends No. 45/2025-Customs)
- Notification 16/2026-Customs (G.S.R. 359(E)) — AIDC (Sl. 15D) raised 1%→5%; SWS on heading 7108 confirmed zero and exemption extended to headings 7107, 7109, 7111, 7112 (amends No. 11/2018 and 11/2021-Customs)
- Notification 17/2026-Customs (G.S.R. 356(E)) — Nominated agency concessional BCD raised 4.35%→10% (amends No. 57/2000-Customs)
- Notification 18/2026-Customs (G.S.R. 357(E)) — India-UAE CEPA preferential rates tightened; TRQ set to 10 MT at 4% (amends No. 22/2022-Customs)

**Important nuance — Notification 17:** Most physical gold in India enters through nominated agencies (MMTC, SBI, designated banks) under No. 57/2000-Customs. Their concessional BCD during the low-duty period was **4.35%** (not 5%), giving an effective duty of **5.35%** rather than 6%. The standard route (No. 45/2025) applied 5% BCD. Post-hike, both routes converge to 15%. This explains the slightly negative mean domestic premium (−257 INR/10g) during the low-duty period when `parity_pre` is fixed at 6%.

**Notification 18 — UAE loophole:** The government simultaneously tightened UAE-origin gold imports under the India-UAE CEPA, which had been used as a re-export arbitrage route. This is a structural policy shift relevant to the mechanism section of the paper.

---

## 4. IBJA PDF Parsing

### Structure
Each IBJA daily report is a 6-page PDF (~1 MB). The pipeline extracts:

| Page | Field | Column name |
|------|-------|-------------|
| 0 | Gold 999 PM rate (INR/10g) | `Gold_INR_PM` ← **primary series** |
| 0 | Gold 999 AM rate (INR/10g) | `Gold_INR_AM` (in ibja_raw.csv, not in clean CSV) |
| 0 | Silver 999 PM rate (INR/kg) | `Silver_INR_PM` |
| 0 | London PM Fix (USD/oz) | `Gold_London_PM_Fix_USD` |
| 0 | COMEX Gold close (USD/oz) | `Gold_COMEX_close_USD` |
| 0 | CFTC Net positions (contracts) | `Gold_CFTC_Net` |
| 0 | SPDR Gold ETF holdings (tonnes) | `SPDR_Gold_tonnes` |
| 2 | DXY close | `DXY_close` |
| 2 | US 10Y / India 10Y yields | `US_10Y`, `India_10Y` |
| 2 | NSE USDINR spot | `USDINR_ibja` |
| 3 | MCX Gold OHLCV + OI | `MCX_Gold_close`, etc. |
| 4 | USDINR futures close | `USDINR_Futures_close` |

### Why PM rates (not AM)
IBJA's own documentation states: *"The above rates are IBJA PM Rates."* PM is the official benchmark. PM also has marginally better coverage (812 vs 811 dates) and no zero-value parse failures (AM had 1 failure on Oct 26, 2022).

### Date offset correction
Each PDF named `DD-MM-YYYY.pdf` contains rates **as of the previous trading day**. A 1-business-day backward shift (`BDay(1)`) is applied in Cell 5 to align PDF dates with rate dates. Verified:
- PDF `14-05-2026.pdf` → rate date May 13, 2026 (first post-hike rate = 160,977 INR/10g) ✓
- PDF `13-05-2026.pdf` → rate date May 12, 2026 (last pre-hike rate = 151,632 INR/10g) ✓

### Coverage and gaps

**Total PDFs downloaded:** 814 (stored in `data/ibja_pdfs/`)
**Total weekdays checked:** 1,161
**Gold_INR_PM found:** 812 / 1,161 (70%)
**Missing dates logged:** `data/ibja_missing_dates.txt`

| Gap | Trading days | Classification |
|-----|-------------|----------------|
| 2022-01-03 → 2022-05-06 | 90 | Structural — IBJA had not begun publishing daily PDFs |
| 2023-06-28 → 2023-09-29 | 68 | Server outage — genuine 404s, confirmed in missing_dates.txt |
| 2024-07-16 → 2024-10-31 | 78 | Server outage — genuine 404s, confirmed in missing_dates.txt |
| 2025-09 → 2025-11-05 | ~29 | Server outage — 29 dates confirmed missing in missing_dates.txt |
| Scattered 1–16 day gaps | ~83 | Indian public holidays (NSE closed) and minor server gaps |

All gaps are **Missing Completely At Random (MCAR)** with respect to the treatment. All structural gaps pre-date the May 13, 2026 event. The post-treatment period (May 13 – Jun 12, 2026) has complete IBJA coverage.

---

## 5. Parity Construction

```
TROY_OZ_TO_10G = 10 / 31.1035 = 0.321507   (converts USD/troy oz → USD/10g)

gold_base = Gold_USD × TROY_OZ_TO_10G × rupees_per_dollar

parity_pre    = gold_base × 1.06   # 6% duty, fixed throughout — ITS baseline
parity_post   = gold_base × 1.15   # 15% duty, fixed throughout — theoretical ceiling
parity_actual = gold_base × duty_mult_ts  # time-varying actual duty (for EDA/plots)
```

**Design choice:** `parity_pre` is fixed at 6% for all dates. `domestic_premium` measures how much IBJA PM exceeds *what the price would be if duty had stayed at 6%*. Pre-hike this should be near zero; post-hike the jump equals the pass-through of the 9-percentage-point duty increase.

**Theoretical ceiling** (parity_post − parity_pre): approximately **13,000 INR/10g** on the day of the hike (varies daily with Gold_USD and FX).

**IBJA rates are ex-GST** (bullion-to-bullion wholesale). GST (3% on gold) is excluded from parity throughout, since IBJA prices and import parity both exclude GST at this level.

---

## 6. Dataset: `gold_policy_clean.csv`

**Shape:** 1,158 rows × 25 columns
**Date range:** 2022-01-03 → 2026-06-12
**Calendar:** Yahoo Finance trading days (master), minus 2 Yahoo Finance weekend artifacts dropped (2025-02-01 Saturday, 2026-06-14 Saturday)

### Column dictionary

| Column | Type | Description | NaN count |
|--------|------|-------------|-----------|
| `Gold_USD` | float | Gold spot (USD/oz), GC=F close, auto-adjusted | 45 (3.9%) |
| `Silver_USD` | float | Silver spot (USD/oz), SI=F close | 41 (3.5%) |
| `Oil_USD` | float | Brent crude (USD/bbl), BZ=F close | 40 (3.5%) |
| `rupees_per_dollar` | float | INR/USD spot, INR=X | 3 (0.3%) |
| `Kalyan` | float | Kalyan Jewellers close (INR), KALYANKJIL.NS | 60 (5.2%) |
| `Nifty50` | float | NSE Nifty 50 index close | 63 (5.4%) |
| `US10Y_yield` | float | US 10-year Treasury yield (%), ^TNX | 43 (3.7%) |
| `MCX_Gold` | float | MCX gold futures close — **100% NaN** (not on Yahoo Finance) | 1158 |
| `GOLDBEES` | float | Nippon India GOLDBEES ETF close (INR) — gold placebo candidate | 61 (5.3%) |
| `Gold_INR_PM` | float | IBJA benchmark gold rate (INR/10g, 999 purity, PM fix) | 348 (30.1%) |
| `Silver_INR_PM` | float | IBJA benchmark silver rate (INR/kg, 999 purity, PM fix) | 348 (30.1%) |
| `ibja_source` | str | `'pdf'` if parsed from local PDF; NaN if archive gap or holiday | 348 (30.1%) |
| `forex_reserves_usd_bn` | float | RBI total forex reserves (USD bn), weekly, forward-filled | 4 (0.3%) |
| `parity_pre` | float | Import parity at 6% duty throughout (ITS baseline) | 46 (4.0%) |
| `parity_post` | float | Import parity at 15% duty throughout (theoretical ceiling) | 46 (4.0%) |
| `parity_actual` | float | Import parity using time-varying actual duty (for EDA plots) | 42 (3.6%) |
| `domestic_premium` | float | Gold_INR_PM − parity_pre (INR/10g). **Primary outcome variable.** | 384 (33.2%) |
| `premium_pct` | float | domestic_premium / parity_pre × 100 (%) | 384 (33.2%) |
| `silver_parity_pre` | float | Silver import parity at 6% duty (INR/kg) | 46 (4.0%) |
| `silver_premium` | float | Silver_INR_PM − silver_parity_pre (descriptive, NOT a valid placebo — silver duty also raised May 13) | 384 (33.2%) |
| `post_hike` | int | 1 if date ≥ 2026-05-13, else 0 | 0 |
| `days_since_hike` | float | Trading days elapsed since May 13, 2026. NaN pre-hike. | 1135 (98.0%) |
| `delta_Gold_USD` | float | Day-over-day % change in Gold_USD | 88 (7.6%) |
| `delta_FX` | float | Day-over-day % change in rupees_per_dollar | 8 (0.7%) |
| `delta_Oil` | float | Day-over-day % change in Oil_USD | 82 (7.1%) |

---

## 7. Key Statistics

### Sub-period domestic premium (Gold_INR_PM − 6% parity)

| Period | n | Mean (INR/10g) | Std | Min | Max |
|--------|---|----------------|-----|-----|-----|
| High-duty (Jan 2022 – Jul 23, 2024) | 433 | +4,380 | 940 | +1,035 | +6,749 |
| Low-duty (Jul 24, 2024 – May 12, 2026) | 321 | −257 | 1,374 | −6,115 | +8,516 |
| **Post-hike (May 13, 2026+)** | **20** | **+10,318** | **1,744** | **+7,184** | **+13,635** |

The low-duty mean of −257 (≈ 0) validates the parity design: when effective duty ≈ 6% = parity baseline, the market trades at parity. The slight negative is consistent with the nominated agency effective rate of 5.35% (Notification 17) rather than 6%.

### May 13, 2026 — Day of hike

| Metric | Value |
|--------|-------|
| Gold_INR_PM (May 12, last pre-hike) | 151,632 INR/10g |
| Gold_INR_PM (May 13, first post-hike) | 160,977 INR/10g |
| IBJA overnight jump | +9,345 INR/10g |
| parity_pre on May 13 | 153,105 INR/10g |
| parity_post on May 13 | 166,105 INR/10g |
| Theoretical ceiling (parity_post − parity_pre) | 12,999 INR/10g |
| domestic_premium on May 13 | +7,872 INR/10g (5.1%) |
| **Day-1 pass-through vs ceiling** | **60.6%** |

### Post-hike premium trajectory (all 20 trading days)

The premium is increasing over time — consistent with progressive price discovery and inventory depletion:

- Days 1–5 (May 13–15): +7,872 → +9,610
- Days 6–15 (May 18 – Jun 2): +9,314 → +10,112
- Days 16–20 (Jun 3–11): +10,997 → +11,457

All 20 post-hike observations show positive premium (minimum: +7,184 INR/10g). No reversion to parity observed.

---

## 8. Data Quality and Outlier Treatment

### Fix A — Zero IBJA rates (Cell 7)
Zero values in `Gold_INR_PM` are PDF parse failures. None found in the final clean dataset (the one Oct 2022 zero-AM case is resolved by using PM rates).

### Fix B — Gold_USD futures-roll spikes (Cell 7)
Four dates with >5% single-day moves in GC=F were identified and nulled in `Gold_USD` and all Gold_USD-derived columns:

| Date | Context |
|------|---------|
| 2025-10-21 | Gold correction from $4,336 → $4,044 (−6.7% across 2 days) |
| 2026-01-30 | Sharp reversal after Jan 29 all-time high of $5,318/oz |
| 2026-02-03 | Bounce from post-ATH correction ($4,622 → $4,920) |
| 2026-03-19 | Drop from $4,889 → $4,570 (−6.5%) |

These are genuine market events (not roll artifacts), occurring during gold's major 2025–26 bull run. They are nulled to prevent extreme leverage-adjusted parity values contaminating the regression. IBJA data is retained on these dates (Gold_INR_PM is non-null for Jan 30, Feb 3, Mar 19).

### Weekend artifact drop
Two Saturday dates included in Yahoo Finance data were dropped: 2025-02-01 (global futures data; NSE not open) and 2026-06-14 (same-day fetch artifact).

---

## 9. Placebo Notes

- **Silver** — **NOT a valid placebo.** Notification 16/2026 raised the AIDC on silver (heading 7106) simultaneously with gold on May 13, 2026. `silver_premium` is retained in the CSV as a descriptive series only.
- **GOLDBEES ETF** — Best remaining placebo candidate. Tracks gold price but is a domestically traded instrument not directly subject to import duty mechanics. Available in the clean CSV as `GOLDBEES` for Notebook 01 analysis.

---

## 10. RBI Supplementary Data (not in clean CSV)

Two RBI series are loaded in Notebook 00 but not merged into the daily regression dataset:

**`data/rbi_forex_reserves_weekly.xlsx`** — Forex reserves *are* merged (forward-filled) into the clean CSV as `forex_reserves_usd_bn`.

**`data/rbi_fx_intervention_monthly.xlsx`** — Monthly net USD purchase/sale by RBI. Available for Section 2 narrative. Key pattern: RBI was a net USD seller (defending rupee) from Oct 2024 through Dec 2025 (−$9bn to −$20bn/month), then net buyer Feb–Mar 2026, before returning to selling in Mar 2026. Not merged into daily CSV (monthly frequency).

**`data/rbi_foreign_trade_monthly.xlsx`** — Total trade balance data (not gold-specific). Latest month: March 2026 (pre-hike; April/May 2026 not yet published). Not used in regression. Available for narrative context.

---

## 11. File Inventory

```
data/
├── gold_policy_clean.csv         ← PRIMARY OUTPUT (1158 × 25)
├── ibja_raw.csv                  ← IBJA-only panel (1161 × 33, all fields)
├── ibja_missing_dates.txt        ← 347 dates with no IBJA PDF (404s)
├── ibja_pdfs/                    ← 814 PDFs (DD-MM-YYYY.pdf format)
├── rbi_forex_reserves_weekly.xlsx
├── rbi_fx_intervention_monthly.xlsx
├── rbi_foreign_trade_monthly.xlsx
└── gazette/
    ├── README.txt                ← Duty structure summary + legal citations
    ├── notification_15_2026_customs.txt   ← BCD 5%→10%
    ├── notification_16_2026_customs.txt   ← AIDC 1%→5%, SWS confirmed 0%
    ├── notification_17_2026_customs.txt   ← Nominated agency BCD 4.35%→10%
    └── notification_18_2026_customs.txt   ← UAE CEPA tightening
```

---

## 12. Reproducing the Pipeline

Run cells in order. Only cells that need to be re-run from scratch are noted.

| Cell | Purpose | Re-run needed? |
|------|---------|----------------|
| 1 | Imports + constants | Always |
| 2 | Yahoo Finance fetch | Only if extending date range |
| 3 | IBJA parser functions + test | Always (defines functions) |
| 3b | IBJA network scrape loop | Only if PDFs unavailable |
| 3c | Download IBJA PDFs to disk | Already done (814 PDFs on disk) |
| 3d | Parse ibja_raw.csv from local PDFs | Only if PDFs updated |
| 4 | RBI forex reserves | Only if updating |
| 4b | RBI FX intervention | Only if updating |
| 5 | Alignment + merge | Always |
| 6 | Construct premium variables | Always |
| 7 | QC checks + save clean CSV | Always |

**Typical re-run path (extending data):** Cell 1 → 2 → 3 → (3c to fetch new PDFs) → 3d → 5 → 6 → 7
