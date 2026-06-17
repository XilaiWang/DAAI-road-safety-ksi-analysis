UK ROAD-CRASH SEVERITY & DEPRIVATION — A RESOURCE-PRIORITISATION ANALYSIS
Warwick WBS · IB94V0 Data Analytics & Artificial Intelligence · individual project
================================================================================

WHAT THIS IS
------------
A reproducible Jupyter notebook that helps a regional road-safety / public-health
team prioritise limited KSI-prevention resources: it identifies the road CONTEXTS
and AREAS where collisions are more likely to become serious or fatal (KSI), and
where the absolute KSI burden per vehicle-mile is highest. It is a retrospective
risk-context / prioritisation analysis (a triage/ranking signal) — NOT a real-time
individual-crash predictor or an automated enforcement system.


HOW TO RUN
----------
1. Python >= 3.10.
2. Install dependencies:
       pip install -r requirements.txt
3. Put the data files in a "data/" folder next to the notebook (see layout below).
4. Open the notebook and run top-to-bottom:
       jupyter notebook road_safety_severity_analysis.ipynb
   then Kernel -> Restart & Run All.
   Seeds are fixed (RANDOM_STATE=42) and all paths are relative, so results
   reproduce on any machine. Section 3 fails fast with a clear message if any
   required data file is missing. Charts are written to "outputs/" (auto-created).


EXPECTED FOLDER LAYOUT
----------------------
project_folder/
  road_safety_severity_analysis.ipynb
  requirements.txt
  README.txt
  data/
    Road Accident Data.csv
    File_10_-_IoD2019_Local_Authority_District_Summaries__lower-tier__.xlsx
    local_authority_traffic.csv
    LAD17_CTYUA17_EW_LU.csv
    File_7_-_All_IoD2019_Scores__Ranks__Deciles_and_Population_Denominators_3.csv
    stats19_collisions_2015_2024_subset.csv      (bundled; see STATS19 note)
    # dft-road-casualty-statistics-collision-1979-latest-published-year.csv  (optional full file)
  outputs/        (created automatically; regenerable figures)


DATA SOURCES (download links)
-----------------------------
- Road Accident Data.csv — Kaggle "xavierberge/road-accident-dataset"
    https://www.kaggle.com/datasets/xavierberge/road-accident-dataset
- English Indices of Deprivation 2019 (File_10 LAD-level; File_7 LSOA-level)
    https://www.gov.uk/government/statistics/english-indices-of-deprivation-2019
- DfT local-authority road traffic / vehicle-miles (local_authority_traffic.csv)
    https://roadtraffic.dft.gov.uk/downloads
- ONS LAD17 -> UTLA/county lookup (LAD17_CTYUA17_EW_LU.csv)
    https://geoportal.statistics.gov.uk/ (Local Authority District to County/UA lookups)
- Official DfT STATS19 collisions 1979-latest (robustness, 2015-2024)
    https://www.gov.uk/government/statistics/road-safety-data


STATS19 NOTE (full vs subset)
-----------------------------
The official STATS19 collision file
("dft-road-casualty-statistics-collision-1979-latest-published-year.csv") is ~1.4 GB
and exceeds GitHub's 100 MB limit, so it is NOT bundled. Instead a derived subset
("stats19_collisions_2015_2024_subset.csv": England 2015-2024, only the 4 columns
the robustness section uses — collision_year, collision_severity,
local_authority_ons_district, lsoa_of_accident_location) IS bundled and reproduces
the same 2015-2024 results. The notebook uses the full file automatically if present,
otherwise the subset. To regenerate the subset from the full file:

    import pandas as pd
    df = pd.read_csv("dft-road-casualty-statistics-collision-1979-latest-published-year.csv",
                     usecols=["collision_year","collision_severity",
                              "local_authority_ons_district","lsoa_of_accident_location"],
                     low_memory=False)
    df = df[(df.collision_year>=2015) & (df.collision_year<=2024)]
    df.to_csv("stats19_collisions_2015_2024_subset.csv", index=False)


NOTEBOOK STRUCTURE
------------------
0  How to Run
1  Business Problem and Decision Context
2  Data Sources and Data Dictionary
3  Data Loading and Reproducibility Setup
4  Data Quality Checks
5  Data Cleaning and Pre-processing
6  Feature Engineering and Leakage Control
7  Exploratory Data Analysis
8  Modelling Strategy
9  Model Evaluation
10 Cost-based Threshold Selection
11 Interpretation: Feature Importance / SHAP (incl. 11.1 unsupervised cross-check)
12 Exposure-adjusted and Equity Analysis
13 Robustness Checks
14 Business Implications
15 Limitations and Future Improvements
16 Reproducibility Checklist


SCOPE / HONESTY NOTES
---------------------
- IMD deprivation data is England-only; all deprivation results are England, area-level.
- Exposure is motor-vehicle vehicle-miles only (pedestrian/cyclist exposure unobserved).
- Equity findings are area-level (ecological) and must not be read as individual risk.
- SHAP/regression/clustering show predictive associations, NOT causal effects.
- Model performance is moderate (2022 ROC-AUC ~0.61): a prioritisation tool, not a
  precise predictor.
