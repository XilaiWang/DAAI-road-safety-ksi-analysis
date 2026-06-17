# Prioritising Serious Road‑Crash Risk: A Resource‑Allocation Tool for UK Road‑Safety Teams

## 1. Business problem
- **The problem.** UK road‑safety partnerships and local‑authority highway teams must decide where to spend a limited Killed‑or‑Seriously‑Injured (KSI) prevention budget — engineering reviews, speed‑management, lighting and junction redesign — because they cannot inspect every road context.
- **Why it matters / real pain point.** KSI collisions impose very large social and budgetary costs, yet safety budgets are fixed, so directing scarce effort at low‑risk contexts wastes money and leaves preventable serious injuries unaddressed.
- **Framed as analytics.** The project delivers a data‑driven triage and ranking signal — not a real‑time crash predictor or an enforcement tool — to identify which road contexts and local areas should be investigated first.
- **Stakeholders.** These are regional road‑safety partnerships and local‑authority road‑safety teams (which own priorities and budgets), highway engineers and transport‑safety planners (who turn risk into schemes), and public‑sector budget holders who must allocate funds cost‑effectively.

## 2. Dataset
- **Primary dataset.** The Kaggle "Road Accident Dataset" (xavierberge) holds 307,973 real GB STATS19‑style collision records for 2021–22 across 23 variables.
- **Why suitable.** It is large, genuinely collected (single‑feature AUCs of only 0.50–0.55 rule out synthetic separation), and records the pre‑existing road context — speed limit, road type, urban/rural, lighting, weather, junction and time — needed to characterise severity.
- **Key variables.** The target is binary KSI (the official Department for Transport severity outcome); predictors are at‑scene road, environment and time fields; deprivation and traffic exposure are added externally (Section 3).
- **Size.** The raw 307,973 records reduce to a 270,721‑row England‑matched modelling sample after cleaning.

## 3. Data pre‑processing
- **Cleaning.** STATS19 "unknown / out‑of‑range" sentinels were recoded to missing and kept as an explicit indicator through dummy encoding, so the model does not treat "unknown" as a genuine road category.
- **Missing values and outliers.** Listwise deletion removed only the 1,154 rows (~0.4%) lacking core fields; their KSI rate (10.1%) was close to retained rows (14.3%), confirming negligible selection bias, while imputation was rejected because it would invent crash‑scene conditions.
- **Duplicates and corruption.** Whole‑row de‑duplication was applied, and the corrupted `Accident_Index` (36% mangled by Excel into scientific notation) was never used as a key.
- **Feature engineering.** An hour‑of‑day feature was parsed from the separate `Time` field, nominal contexts were dummy‑encoded, and a variance‑inflation check (≈1.00) confirmed no multicollinearity.
- **External data — what, why, how.** Three sources were joined: English Indices of Deprivation 2019 (by local‑authority name), DfT local‑authority traffic / vehicle‑miles (by ONS code via an ONS district‑to‑county lookup), and the official multi‑year STATS19 file (2015–2024). They were necessary to measure equity and to convert raw counts into exposure‑adjusted burden; coverage was reported transparently as a waterfall (88.3% England‑matched).

## 4. Analysis
- **Leakage control.** Post‑crash fields (severity, casualty and vehicle counts) and place‑memorising coordinates were excluded because they are unavailable when teams prioritise; their near‑chance AUCs (0.528 and 0.424) confirmed little predictive loss.
- **Methods.** A transparent logistic‑regression baseline, a decision tree, k‑nearest neighbours, a random forest and XGBoost were compared, giving both interpretability and a strong benchmark for tabular data.
- **Validation that mirrors deployment.** Models were trained on 2021 and tested once on 2022 (out‑of‑time); GroupKFold‑by‑local‑authority served as a spatial‑leakage and stability diagnostic, while the model and threshold were chosen on a 2021 validation split only.
- **Interpretation and equity.** Probabilities were calibrated; a cost‑based threshold reflected the asymmetric cost of a missed KSI; SHAP attributed predictions to drivers; k‑means and hierarchical clustering cross‑checked the segments; and a cluster‑robust nested logistic regression with a vehicle‑mile exposure denominator addressed equity.

## 5. Results and interpretation
- **Predictability.** Pre‑existing context is only weakly predictive: calibrated XGBoost reached a 2022 out‑of‑time ROC‑AUC of 0.61 (PR‑AUC 0.205) and barely beat logistic regression, while the random forest over‑fitted (training AUC 0.913 against validation 0.536).
- **Operating point.** At a 5:1 false‑negative‑to‑false‑positive cost the model recalls about 40% of KSI contexts (precision ~20%), rising to ~91% recall at 10:1, so the threshold is a transparent budget lever.
- **Severity drivers.** SHAP and both clusterings agree that high speed limits, rural settings and poor lighting raise predicted severity — a fast (~62 mph) rural cluster carries the highest KSI rate (0.19) — though these are predictive associations, not proven causes.
- **Equity (two channels).** Severity given a crash is mildly lower in deprived areas, but once exposure is added the absolute KSI burden per vehicle‑mile is roughly 1.5× higher in the most‑deprived areas, driven by crash frequency, and this persists across 2015–2024, so it is not a pandemic artefact.
- **Business meaning.** This answers the opening problem directly: teams can rank fast‑rural contexts for severity reduction and deprived / high‑exposure areas for frequency reduction, rather than chasing raw collision counts.

## 6. Business insights and recommendations
- **Insight.** Where serious harm concentrates depends on the question asked: raw density favours urban cores, but exposure‑adjusted burden reveals which authorities bear disproportionate harm per mile travelled.
- **Actions.** The model should be used as a triage signal rather than an automatic rule; high‑speed rural contexts should be prioritised for engineering, speed and lighting review; deprived or high‑exposure urban areas should be targeted for traffic‑calming and vulnerable‑user protection; and the threshold should be set to match the available review budget. Illustratively, screening 10,000 contexts could surface ~580 KSI‑risk cases and, at a 5% intervention effect, avoid ~29 KSI outcomes.
- **Limitations.** The data are observational (associations, not causal effects), exposure covers motor vehicles only, deprivation is area‑level (an ecological‑fallacy risk reduced but not eliminated by an LSOA‑level check), and performance suits prioritisation rather than precise prediction.
- **Future enhancements.** Adding road‑geometry, traffic‑flow and pedestrian/cyclist‑exposure features, evaluating top‑k targeting for fixed budgets, and pairing the framework with intervention‑rollout data for quasi‑experimental causal estimates would strengthen the analysis.

## References
- xavierberge (n.d.) *Road Accident Dataset* [dataset]. Kaggle. Available at: https://www.kaggle.com/datasets/xavierberge/road-accident-dataset
- Ministry of Housing, Communities & Local Government (2019) *English Indices of Deprivation 2019* [dataset]. GOV.UK.
- Department for Transport (2025) *Road Safety Data (STATS19)* [dataset]. GOV.UK.
- Department for Transport (2024) *Road traffic statistics* [dataset]. GOV.UK.
- Office for National Statistics (2017) *Local Authority District to County (2017) Lookup* [dataset]. ONS Open Geography Portal.

---

*AI declaration (not included in the word count).*
- **Why.** A generative AI was used to accelerate scaffolding, structuring and debugging, and to stress‑test the design through a simulated critical review, freeing time for interpretation.
- **What.** It assisted with the notebook skeleton, join logic, evaluation and robustness code and their debugging, and with drafting and tightening this report.
- **Which.** Claude (Anthropic).
- **Where.** Used in code structure, comments and report drafting; all dataset choices, statistical decisions, claim wording and conclusions were reviewed, verified against the outputs, and finalised by me. The work remains my own and I retain intellectual ownership of it.
