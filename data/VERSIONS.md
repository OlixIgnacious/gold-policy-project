# Dataset Versions — India Gold Import Duty Study

| Version | File | IBJA rows | Source | Created | Key change |
|---------|------|-----------|--------|---------|------------|
| v1 | gold_policy_v1.csv | 810 | IBJA PDF archive | 2025 | Baseline |
| v2 | gold_policy_v2.csv | 915 | v1 + BullionWorld gap-fill | 2026-06-17 | +105 archive-gap dates recovered |

## How to switch versions

In `01_eda.ipynb` Cell 1:
```python
DATASET_VERSION = 'v1'   # reproduces pre-BullionWorld findings
DATASET_VERSION = 'v2'   # current (BullionWorld-patched)
```

In `03_its_regression.ipynb` Cell 1 (once created):
```python
DATASET_VERSION = 'v2'   # always run regression on latest
```

## Impact of v1 → v2

| Metric | v1 | v2 | Change |
|--------|----|----|--------|
| Premium observations | 774 | 879 | +105 |
| Pre-hike mean premium (INR/10g) | 2,406 | 2,397 | −9 (negligible) |
| Post-hike mean premium (INR/10g) | 10,318 | 10,318 | 0 |
| Raw mean shift | 7,913 | 7,921 | +8 (<0.1%) |
| ITS β₁ (to be re-checked) | ~₹10,520 | TBD | — |

## What ibja_source column means
- `pdf` — price extracted directly from IBJA daily PDF
- `bullionworld` — price sourced from BullionWorld.in (IBJA data republisher)
- `NaN` — date has no IBJA data (holiday, US market closure, or uncovered archive gap)

## Still-missing (133 gap-cluster dates after v2)
See `data/IBJA_Market_Calendar_133_Dates.xlsx` for full classification.
110 are genuine archive gaps where IBJA published but PDF is not online.
Pending IBJA email response — will become v3 if recovered.
