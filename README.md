# What makes a UK road crash turn serious — and does the burden fall unequally?

A reproducible data‑analysis project (Warwick MSc, IB94V0) on UK road‑collision severity, joined to area deprivation (IMD) and traffic exposure, written as a first‑person *discovery* notebook.

> **Note:** large raw datasets are **not** committed (see `.gitignore`); download links are below. The notebook documents the full analytical flow end‑to‑end.

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
2. Download the datasets below and set `BASE` (top of the notebook) to the folder holding them.
3. Run all cells (`jupyter nbconvert --to notebook --execute road_safety_severity_analysis.ipynb`).

## Data sources (download separately — gitignored)
| Dataset | Source |
|---|---|
| Road Accident Dataset (primary, 2021–22) | https://www.kaggle.com/datasets/xavierberge/road-accident-dataset |
| English Indices of Deprivation 2019 (IMD) — File_7 (LSOA) & File_10 (LAD) | https://www.gov.uk/government/statistics/english-indices-of-deprivation-2019 |
| DfT local‑authority traffic (vehicle‑miles) | https://roadtraffic.dft.gov.uk/downloads |
| Official multi‑year STATS19 collisions (1979–2024) + Data Guide | https://www.gov.uk/government/statistics/road-safety-data |
| LTLA→UTLA (district→county) lookup | ONS Open Geography Portal — https://geoportal.statistics.gov.uk/ |

## Notebook structure
Setup → authenticity check → target + leakage defence → IMD join (coverage waterfall) → cleaning → EDA → feature selection + VIF → models (logistic/tree/RF/XGBoost, out‑of‑time 2021→2022, GroupKFold‑by‑LAD) → calibration + cost‑based threshold + confusion matrix → SHAP → distributional audit + cluster‑robust nested logit → **§12.6 exposure denominator** → k‑means → conclusions → sensitivity → **§17 robustness extension (multi‑year + LSOA, official STATS19)** → AI disclosure & references.

## Limitations
Observational (correlation ≠ causation); IMD is area‑level (ecological) and England‑only; vehicle‑mile exposure omits pedestrian/cyclist exposure; this is a *regional‑equity (distributional)* analysis, not an individual fairness audit. References in the notebook are marked `[VERIFY]` and must be confirmed before submission.

## AI usage
An AI assistant (Claude, Anthropic) was used to accelerate code scaffolding, join logic, and a simulated peer‑review that hardened the design; all statistical choices, claim wording and conclusions were reviewed and written by the author.
