# What makes a UK road crash turn serious — and does the burden fall unequally?

A reproducible data‑analysis project (Warwick MSc, IB94V0) on UK road‑collision severity, joined to area deprivation (IMD) and traffic exposure, written as a first‑person *discovery* notebook.

> **Self‑contained & reproducible:** the datasets used are bundled under [`data/`](data/) and the notebook reads them via a relative path (`BASE = ./data`), so it runs after a clone with no extra downloads. The one exception is the official STATS19 microdata — the full file is 1.4 GB (over GitHub's 100 MB limit), so `data/stats19_collisions_2015_2024_subset.csv` is a **derived subset** (the 4 columns + 2015–2024 rows the §17 robustness extension uses); regenerate from the full file via the link below.

## Business framing
- **Decision‑maker:** a regional road‑safety / public‑health team deciding where to target finite **KSI** (Killed‑or‑Seriously‑Injured, the official DfT outcome) prevention resources.
- **Questions:** (1) can pre‑crash road context predict severity? (2) what drives severity? (3) does the burden fall unequally across rich vs poor areas?

## Headline findings (honest, two‑channel)
- **Prediction is only modestly possible** (out‑of‑time XGBoost ROC‑AUC ≈ 0.61, stable; RandomForest over‑fits) — severity is largely intrinsic to the crash. The model is a **risk‑factor / triage** tool, not an oracle.
- **The road environment drives severity** — high speed limits, rural roads, poor light (SHAP + k‑means agree: a "fast‑rural" cluster has the highest KSI rate).
- **Two opposing equity channels:** crash *severity given a crash* is mildly **anti‑regressive** (fast rural roads sit in least‑deprived areas), but crash *frequency per vehicle‑mile* is strongly **regressive**; once traffic exposure is added, **net KSI per vehicle‑mile is ~1.5× higher in the most‑deprived areas** → the burden **is** regressive.
- **Robustness:** the regressive frequency burden holds over **2015–2024** (not a COVID artefact); the anti‑regressive severity gradient holds at **LSOA** level (not an ecological‑fallacy artefact). The rising KSI‑*share* time trend is a **reporting‑system artefact** (CRASH/COPA, ~2016–19), not a real worsening — so the cross‑sectional gradient is used, not the trend.

## How to run
1. `pip install -r requirements.txt`
2. Just run all cells — data is bundled in `data/` and read relatively (`jupyter nbconvert --to notebook --execute road_safety_severity_analysis.ipynb`).

## Data (bundled in `data/`; original sources below)
| Dataset | In repo | Source |
|---|---|---|
| Road Accident Dataset (primary, 2021–22) | `data/Road Accident Data.csv` | https://www.kaggle.com/datasets/xavierberge/road-accident-dataset |
| English Indices of Deprivation 2019 (IMD) — File_7 (LSOA) & File_10 (LAD) | `data/File_7_*.csv`, `data/File_10_*.xlsx` | https://www.gov.uk/government/statistics/english-indices-of-deprivation-2019 |
| DfT local‑authority traffic (vehicle‑miles) | `data/local_authority_traffic.csv` | https://roadtraffic.dft.gov.uk/downloads |
| Official multi‑year STATS19 collisions | `data/stats19_collisions_2015_2024_subset.csv` (derived subset) | https://www.gov.uk/government/statistics/road-safety-data |
| LTLA→UTLA (district→county) lookup | `data/LAD17_CTYUA17_EW_LU.csv` | https://geoportal.statistics.gov.uk/ |

### Other this‑case files included
- `data/_exploration_extra/ras-all-tables-excel/` — DfT *Reported road casualty statistics* (RAS) published summary tables; used for the official KSI scale/trend and as a citation source.
- `docs/Road_safety_statistics_guidance_GOV.UK.pdf` — DfT STATS19 definitions/methodology guidance (source for the KSI definition).

## Two large datasets deliberately NOT committed — why, and how I reduced the data
Both are this‑case datasets, but left out of the repo for the reasons below. Each has a download link, and the reduction is reproducible, so the analysis stays fully reproducible.

### 1. Full STATS19 collision microdata — 1.4 GB (blocked by GitHub's file‑size limit)
- **Why not committed:** the official file `dft-road-casualty-statistics-collision-1979-latest-published-year.csv` is **9,015,100 rows × 44 columns (1979–2024) ≈ 1.4 GB** — far over GitHub's **100 MB per‑file hard limit** (free Git LFS quota is also < 1.4 GB).
- **How I cleaned it down to only what I need** → committed as `data/stats19_collisions_2015_2024_subset.csv` (≈ 29 MB). The §17 robustness extension uses only **4 of the 44 columns** and only the **2015–2024** rows (the analysis window), so I dropped the rest:
  ```python
  import pandas as pd
  df = pd.read_csv("dft-road-casualty-statistics-collision-1979-latest-published-year.csv",
                   usecols=["collision_year", "collision_severity",
                            "local_authority_ons_district", "lsoa_of_accident_location"],
                   low_memory=False)
  df = df[(df.collision_year >= 2015) & (df.collision_year <= 2024)]
  df.to_csv("data/stats19_collisions_2015_2024_subset.csv", index=False)
  # 9,015,100 -> 1,150,305 rows | 44 -> 4 columns | 1.4 GB -> 29 MB (identical results)
  ```
- **Download the full file:** https://www.gov.uk/government/statistics/road-safety-data (file *"Road Safety Data — Collisions 1979 – Latest Published Year"*).

### 2. ONS NSPCL postcode lookup — ~0.5 GB (left out as unused)
- **Why not committed:** a **generic national postcode→geography lookup** (124 CSVs, ≈ 496 MB) downloaded during an exploratory detour to map crash districts to counties. It turned out **not to carry an upper‑tier county field**, so the analysis never used it — the district→county mapping comes from the small `data/LAD17_CTYUA17_EW_LU.csv` instead. Committing ~0.5 GB of unused data would only bloat the repo with no analytical value.
- **Download if needed:** ONS Open Geography Portal — https://geoportal.statistics.gov.uk/ (search *"NSPCL"* / National Statistics Postcode Lookup).

> The data kept in `data/` is further cleaned inside the notebook: **§6 documents every missing‑value / sentinel‑recode / de‑duplication / England‑scope decision with its reasoning** (e.g. dropping `Carriageway_Hazards` at 98% missing, recoding "unknown" sentinels to NaN, de‑duplicating on whole rows rather than the Excel‑corrupted `Accident_Index`).

## Notebook structure
Setup → authenticity check → target + leakage defence → IMD join (coverage waterfall) → cleaning → EDA → feature selection + VIF → models (logistic/tree/RF/XGBoost, out‑of‑time 2021→2022, GroupKFold‑by‑LAD) → calibration + cost‑based threshold + confusion matrix → SHAP → distributional audit + cluster‑robust nested logit → **§12.6 exposure denominator** → k‑means → conclusions → sensitivity → **§17 robustness extension (multi‑year + LSOA, official STATS19)** → AI disclosure & references.

## Limitations
Observational (correlation ≠ causation); IMD is area‑level (ecological) and England‑only; vehicle‑mile exposure omits pedestrian/cyclist exposure; this is a *regional‑equity (distributional)* analysis, not an individual fairness audit. References in the notebook are marked `[VERIFY]` and must be confirmed before submission.

## AI usage
An AI assistant (Claude, Anthropic) was used to accelerate code scaffolding, join logic, and a simulated peer‑review that hardened the design; all statistical choices, claim wording and conclusions were reviewed and written by the author.
