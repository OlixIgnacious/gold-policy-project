# Reference Audit — Gold Policy Paper
**Date:** July 7, 2026  
**Verified against:** AEA, NBER, MDPI, and local WGC PDFs

---

## Summary of all 14 .bib entries

### ✅ Verified correct

| Key | Authors | Title | Venue | Status |
|-----|---------|-------|-------|--------|
| Jorda2005 | Òscar Jordà | Estimation and Inference of Impulse Responses by Local Projections | AER 95(1): 161–182, 2005 | ✅ AEA confirmed |
| Pesaran2001 | Pesaran, Shin, Smith | Bounds Testing Approaches to the Analysis of Level Relationships | JAE 16(3): 289–326, 2001 | ✅ Well-known; confirmed |
| Fajgelbaum2020 | Fajgelbaum, Goldberg, Kennedy, Khandelwal | The Return to Protectionism | QJE 135(1): 1–55, 2020 | ✅ NBER w25638 confirmed |
| Newey1987 | Newey, West | A Simple, Positive Semi-Definite, HAC Covariance Matrix | Econometrica 55(3): 703–708, 1987 | ✅ Well-known; confirmed |
| Campa2002 | Campa, Goldberg | Exchange Rate Pass-Through into Import Prices | ReStat 87(4): 679–690, 2005 | ✅ NBER confirmed (see note) |
| Nakamura2018 | Nakamura, Steinsson | Identification in Macroeconomics | JEP 32(3): 59–86, 2018 | ✅ AEA confirmed (open access) |
| Menon1995 | Menon | Exchange Rate Pass-Through | JES 9(2): 197–231, 1995 | ✅ Confirmed from knowledge |
| Gopinath2010 | Gopinath, Itskhoki, Rigobon | Currency Choice and Exchange Rate Pass-Through | AER 100(1): 304–336, 2010 | ✅ AEA confirmed |
| GoldSmuggling2024 | Maria Immanuvel Susai, Lazar Daniel | Gold Smuggling in India and Its Effect on the Bullion Industry | JRFM 17(3): 122, 2024 | ✅ MDPI confirmed (open access) |
| GoldElasticity2016 | Baig, Ahmad | An Estimation of Gold Import Demand Function in India | IJABER 14(12): 8707–8718, 2016 | ✅ Confirmed by user |
| WGC2026May | World Gold Council | India Gold Market Update: Import Tightening | WGC, May 2026 | ✅ PDF on disk |
| WGC2026Apr | World Gold Council | India Gold Market Update: Mixed Reading | WGC, Apr 2026 | ✅ PDF on disk |
| WGC2024Jul | World Gold Council | India Gold Market Update: Import Duties Reduced to Decade Low | WGC, Jul 2024 | ✅ Confirmed by WGC2026May Table 1 |
| WGC2024Aug | World Gold Council | India Gold Market Update: Import Duty Reduction, Catalyst for Demand | WGC, Aug 2024 | ✅ Plausible; cited for 50t estimate |

---

## Notes on individual entries

### Campa2002
- BibTeX key is `Campa2002` (NBER WP date), but year=2005 (published).  
- Published title per NBER: "Exchange Rate Pass Through into Import Prices" (no subtitle, no hyphen in "Pass Through").  
- Our .bib has the WP title "Exchange Rate Pass-Through into Import Prices: A Macro or Micro Phenomenon?" — acceptable since it's the same paper; subtitle may be retained.

### GoldSmuggling2024
- **Published authors (JRFM):** Maria Immanuvel Susai and Lazar Daniel (2 authors).  
- **Working paper (IIMA):** Also lists Rakshambiga VN as third author — not included in the published version.  
- **BibTeX updated to:** `{Maria Immanuvel Susai} and {Lazar Daniel}` ✓  
- **Important:** The +0.52 correlation between duty and smuggling does NOT come from this paper. The paper establishes Granger causality (customs duty → smuggled gold, F=2.49, p=0.09) but does not report a +0.52 figure.  
- The +0.52 comes from **WGC2026May** ("Import duties and smuggling" section, p. 4).

### WGC2026May — critical source
- Contains: "Excluding the COVID years of 2020–21, the correlation between import duty and unofficial imports is positive at **0.52**"  
- This is now correctly attributed to WGC2026May in main.tex (not GoldSmuggling2024).

---

## PDF files on disk

| File | Corresponds to | Notes |
|------|----------------|-------|
| `India_gold_market_update_Import_tightening.pdf` | WGC2026May | ✅ Confirmed. Contains the 0.52 figure |
| `India_gold_market_update_apr26_final.pdf` | WGC2026Apr | ✅ Confirmed. "Mixed reading" subtitle matches |
| `India_gold_market_update_Volatility_softens_demand .pdf` | **February 2026** (NOT in .bib) | ⚠️ This is the Feb 2026 WGC update — not cited in the paper |
| `GoldSmuggling2024_Immanuvel_Daniel.pdf` | GoldSmuggling2024 | Available from MDPI: https://mdpi-res.com/d_attachment/jrfm/jrfm-17-00122/article_deploy/jrfm-17-00122.pdf |

---

## PDFs still needed for .bib entries

The shell sandbox does not have outbound internet for binary downloads. Download these manually:

| Key | Direct PDF URL |
|-----|----------------|
| Jorda2005 | https://www.aeaweb.org/articles/pdf/doi/10.1257/0002828053828518 |
| Pesaran2001 | https://onlinelibrary.wiley.com/doi/epdf/10.1002/jae.616 (subscription) |
| Fajgelbaum2020 | https://www.nber.org/system/files/working_papers/w25638/w25638.pdf |
| Newey1987 | https://www.econometricsociety.org/publications/econometrica/1987/05/01/... (subscription) |
| Campa2002 | https://www.nber.org/system/files/working_papers/w8934/w8934.pdf (WP version) |
| Nakamura2018 | https://www.aeaweb.org/articles/pdf/doi/10.1257/jep.32.3.59 (open access) |
| Menon1995 | Subscription only (Journal of Economic Surveys) |
| Gopinath2010 | https://www.aeaweb.org/articles/pdf/doi/10.1257/aer.100.1.304 (subscription) |
| GoldSmuggling2024 | https://mdpi-res.com/d_attachment/jrfm/jrfm-17-00122/article_deploy/jrfm-17-00122.pdf |
| GoldElasticity2016 | IJABER subscription only |
| WGC2024Jul | https://www.gold.org/goldhub/research/india-gold-market-update-import-duties-reduced-decade-low |
| WGC2024Aug | https://www.gold.org/goldhub/research/india-gold-market-update-import-duty-reduction-catalyst-demand |

---

## Issues fixed in this audit session

1. **GoldSmuggling2024 authors**: `Veeramani, C. and Dhir, Garima` → `{Maria Immanuvel Susai} and {Lazar Daniel}` (confirmed from both IIMA working paper and MDPI published version).

2. **+0.52 correlation**: Originally attributed to GoldSmuggling2024 (wrong). This figure appears in WGC2026May p. 4 ("Import duties and smuggling"). Now correctly attributed to WGC2026May in main.tex.

3. **"Nepal and Myanmar corridors"**: Originally attributed to GoldSmuggling2024. Not in that paper (routes mentioned are Dubai/Gulf → Kerala, Chennai, Mumbai airports). Removed from main.tex.

4. **GoldSmuggling2024 title**: Updated from fabricated "Gold Smuggling in India: Duty Structure and Bullion Market Effects" → actual title "Gold Smuggling in India and Its Effect on the Bullion Industry".

5. **DOI added**: `10.3390/jrfm17030122`

6. **Fajgelbaum key**: Renamed from `Fajgelbaum2019` → `Fajgelbaum2020` (published 2020, QJE).

7. **Gopinath key**: Renamed from `Gopinath2020` → `Gopinath2010` (published 2010, AER).

8. **Campa2002**: Updated from NBER WP entry → published ReStat 2005 details.

9. **WGC year format**: Fixed `{2026, May}` → `year={2026}, month=may` style throughout.

10. **7 uncited entries removed**: Arize2008, Ashenfelter1978, Bollerslev1986, Engle1982, Goldberg2010, Hausman1996, Schmitt2018.
