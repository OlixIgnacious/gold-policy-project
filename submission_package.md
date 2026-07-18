# Submission Package
## Import Duty Pass-Through to Domestic Gold Prices: Quasi-Experimental Evidence from India's Gold Market, 2024–2026
*Ashwini Sharma — July 2026*

---

## 1. arXiv Submission

### Step-by-step checklist

- [ ] Go to https://arxiv.org/submit
- [ ] Select **New Submission**
- [ ] Primary category: **econ.GN** (General Economics)
- [ ] Cross-list: **econ.EM** (Econometrics and Theory)
- [ ] Upload `arxiv_submission.zip` (519 KB, 7 files — see below)
- [ ] Paste the title below into the Title field
- [ ] Paste the plain-text abstract below into the Abstract field
- [ ] Add authors: Ashwini Sharma
- [ ] License: **CC BY 4.0** (recommended for maximum reuse) or CC BY-NC 4.0
- [ ] No comments needed (leave Comments field blank or add "19 pages, 4 figures")
- [ ] Submit and note the arXiv ID (format: 2607.XXXXX)

### arXiv zip contents (arxiv_submission.zip)

| File | Role |
|------|------|
| main.tex | Main LaTeX source (graphicspath patched to ./) |
| abstract.tex | Abstract input file |
| main.bbl | Pre-compiled bibliography (arXiv won't run bibtex) |
| fig01_premium_timeseries.png | Figure 1 — Premium time series |
| fig_lp_impulse_response.png | Figure 2 — Local projection IRF |
| fig15_placebo_test.png | Figure 3 — Placebo test |
| fig_garch_volatility.png | Figure 4 — GARCH volatility |

### arXiv Title (copy-paste exactly)

```
Import Duty Pass-Through to Domestic Gold Prices: Quasi-Experimental Evidence from India's Gold Market, 2024--2026
```

### arXiv Plain-Text Abstract (LaTeX stripped — copy-paste into abstract field)

```
This paper estimates how much of India's May 2026 gold import duty hike, from 6 to 15%, passed through to domestic gold prices. Exploiting the sharp and unanticipated policy discontinuity as a quasi-experiment, this paper applies an interrupted time series (ITS) design with Newey-West HAC standard errors to a primary sample of 375 daily observations from the India Bullion and Jewellers Association benchmark price series (July 2024 -- June 2026), with 346 pre-hike and 29 post-hike trading days. The full dataset comprises 929 daily price observations (824 IBJA PDF circulars and 105 BullionWorld.in gap-fills) spanning January 2022 through June 2026, within a 1,170-row weekday date spine.

The duty hike raised the domestic gold premium by Rs. 9,841 per 10 grams, representing 81.7% of the theoretical ceiling, with a 95% confidence interval of 73.3 to 90.1%. Full pass-through is formally rejected (t = -4.26, p < 0.001). Local projections confirm the adjustment was immediate on the first trading day and held on a stable plateau through Day 14 (78.2%), with no evidence of anticipatory pricing. An ARDL bounds test confirms cointegration between domestic and international gold prices in the pre-hike period; the stable local-projection plateau through Day 14 is consistent with the shift persisting rather than mean-reverting within the observation window. Placebo tests and counterfactual methods confirm the result is specific to May 13, 2026.

This paper also documents asymmetric price transmission: the duty hike passed through at 81.7% by the first trading day, while the July 2024 cut reached only 65.6% at an equivalent horizon (clean sample, July 2022--May 2026, N = 746). The hike therefore transmitted more completely than the cut, consistent with downward rigidity in duty pass-through. The incomplete pass-through is consistent with supply-side dampening from front-loaded inventory imported at the prior duty rate, seasonal demand weakness, and historical smuggling elasticity.
```

### arXiv subject classification

| Field | Value |
|-------|-------|
| Primary | **econ.GN** — General Economics |
| Cross-list | **econ.EM** — Econometrics and Theory |

*Rationale: econ.GN captures the applied policy focus (commodity taxation, trade policy); econ.EM captures the ITS/LP/ARDL methodology. q-fin.GN is a weaker fit — this is not a finance paper.*

---

## 2. SSRN Submission

### Step-by-step checklist

- [ ] Go to https://www.ssrn.com/index.cfm/en/
- [ ] Click **Submit a Paper** → **Submit Your Paper**
- [ ] Network: **ERN (Economics Research Network)** → Sub-network: **Development Economics** or **International Trade**
- [ ] Upload `paper/main.pdf` (the locked PDF, ~675 KB)
- [ ] Paste the metadata block below
- [ ] Approval status: **Approved** (SSRN does not review content; posts within 24–48 hours)
- [ ] Note the SSRN paper ID after submission

### SSRN Metadata Block (copy-paste field by field)

**Title**
```
Import Duty Pass-Through to Domestic Gold Prices: Quasi-Experimental Evidence from India's Gold Market, 2024–2026
```

**Abstract**
*(Same plain-text abstract as arXiv above — paste the three-paragraph version)*

**Author**
```
Name: Ashwini Sharma
Email: ashwini.sharma0807@gmail.com
Affiliation: Independent
```

**Date**
```
July 2026
```

**JEL Classification Codes**
```
E31, F13, H22, Q02
```

**Keywords**
```
import duty pass-through; gold market; India; interrupted time series; quasi-experiment; commodity taxation
```

**Number of Pages**
```
19
```

**Accepted for publication?**
```
No (working paper / preprint)
```

---

## 3. Journal Targets (for future submission)

Both platforms are preprint servers — there is no editorial cover letter needed. When you submit to a journal, use the following:

### Tier 1 targets (high reach, policy-oriented)
| Journal | Notes |
|---------|-------|
| Journal of Development Economics | Strong fit: commodity markets, India, policy evaluation |
| Journal of International Economics | Exchange rate / trade policy angle |
| American Economic Journal: Economic Policy | ITS quasi-experimental design matches AEJ-Policy style |

### Tier 2 targets (more accessible, faster turnaround)
| Journal | Notes |
|---------|-------|
| Journal of Applied Econometrics | ARDL/ITS methods fit well |
| Resources Policy | Gold and commodity markets specialty |
| Review of Development Economics | Emerging market policy focus |

### Cover letter template (for journal submissions)

```
Dear Editors,

We submit for your consideration "Import Duty Pass-Through to Domestic Gold Prices: Quasi-Experimental Evidence from India's Gold Market, 2024–2026."

This paper estimates pass-through from India's May 2026 gold import duty hike (6% to 15%) using an interrupted time series design with Newey-West HAC standard errors. The headline finding — 81.7% pass-through within the first trading day, with formal rejection of full pass-through — rests on a novel daily price dataset (929 observations, January 2022–June 2026) compiled from IBJA PDF circulars and BullionWorld.in. We document asymmetric transmission (hike: 81.7% vs. cut: 65.6%), consistent with downward rigidity, and validate the result using local projections, ARDL bounds tests, GARCH volatility analysis, and XGBoost/ARIMAX/Prophet counterfactuals.

The paper has been posted as a preprint on arXiv (econ.GN/econ.EM) and SSRN. It is not under review elsewhere.

We have no conflicts of interest to declare. Data and replication code are available at https://github.com/OlixIgnacious/gold-policy-project.

Thank you for your consideration.

Ashwini Sharma
ashwini.sharma0807@gmail.com
```

---

## 4. Post-submission to-do

- [ ] After arXiv posts, add arXiv ID to SSRN "Related Links" field
- [ ] After SSRN posts, update GitHub README with both links
- [ ] Share arXiv/SSRN link on relevant forums (r/economics, Twitter/X econ community, India finance policy circles)
- [ ] Monitor for arXiv "put on hold" notice (rare; usually clears within 24 h)
- [ ] File v2 when July 2026 data is available (noted in paper: "July 2026 observations will be incorporated in a follow-up revision")
