# Prioritising Serious Road‑Crash Risk: A Resource‑Allocation Tool for UK Road‑Safety Teams and Motor‑Insurance Risk Managers

## 1. Business problem
- **The problem.** UK road‑safety teams and motor‑insurance risk managers share a resource‑allocation problem: local authorities must prioritise a limited Killed‑or‑Seriously‑Injured (KSI) prevention budget, while insurers must identify road contexts likely to generate high‑cost bodily‑injury exposure — and neither can review every context.
- **Why it matters / real pain point.** KSI collisions impose large social costs on public bodies and high severe‑claim exposure on insurers, yet review capacity is limited, so misdirected effort wastes scarce resources and leaves preventable serious injuries unaddressed.
- **Framed as analytics.** The project delivers a triage and ranking signal — not a real‑time crash predictor, an enforcement tool, an insurance‑pricing model, or a causal / DiD evaluation — to rank road contexts and areas for review, prevention, insurer loss‑prevention advice and public–private partnerships.
- **Stakeholders.** These span road‑safety partnerships and local‑authority highway teams (budgets/review queues), highway engineers and transport planners (delivery), motor/fleet insurers and risk managers (territorial risk awareness and loss‑prevention, not pricing), and budget/partnership holders — at the interface of public‑safety allocation and commercial road‑risk management.

## 2. Dataset
- **Primary dataset.** The Kaggle "Road Accident Dataset" (xavierberge, n.d.) holds 307,973 real GB STATS19‑style collision records (2021–22, 23 source variables).
- **Why suitable.** It is a large STATS19‑style collision dataset recording the pre‑existing road context (speed, road type, urban/rural, lighting, weather, junction, time) needed to characterise severity.
- **Key variables.** The target is binary KSI (the official Department for Transport severity outcome); predictors are at‑scene road, environment and time fields; deprivation and traffic exposure are added externally (Section 3).
- **Size.** The raw 307,973 records reduce to a 270,721‑row England‑matched modelling sample after cleaning.

## 3. Data pre‑processing
- **Cleaning.** STATS19 "unknown / out‑of‑range" sentinels were recoded to missing and kept as an explicit indicator (dummy encoding), so "unknown" is not treated as a real road category.
- **Missing values and outliers.** Listwise deletion removed only the 1,154 rows (~0.4%) lacking core fields; their KSI rate (10.1%) was close to retained rows (14.3%), confirming negligible bias; imputation would invent crash‑scene conditions.
- **Duplicates and corruption.** Whole‑row de‑duplication was applied; the corrupted `Accident_Index` (36% mangled by Excel) was never used as a key.
- **Feature engineering.** An hour‑of‑day feature was parsed from the separate `Time` field, nominal contexts were dummy‑encoded, and a VIF check (≈1.00) confirmed no multicollinearity among the numeric predictors.
- **External data - what, why, how.** Four external sources were joined: IMD 2019 by authority name (MHCLG, 2019), DfT vehicle‑miles by ONS code via an ONS district→county lookup (DfT, 2024; ONS, 2017), multi‑year STATS19 2015–2024 (DfT, 2025), and DfT hospital drive‑times JTS0506 (DfT, 2021) - for equity, exposure‑adjustment and emergency‑care access; coverage was a transparent waterfall (88.3% England‑matched).

## 4. Analysis
- **Leakage control.** Post‑crash fields (severity, casualty/vehicle counts) and place‑memorising coordinates were excluded as unavailable when teams prioritise; their near‑chance AUCs (0.526 and 0.423) confirmed little predictive loss.
- **Methods.** This is a **binary classification** problem: dependent variable `KSI` (1 = killed/seriously‑injured, 0 = slight); independent variables are at‑scene road/environment/time fields. A binary outcome with mixed categorical/numeric predictors makes **logistic regression** (interpretable) and **XGBoost** (strong benchmark) natural; a decision tree, k‑NN and random forest are compared. XGBoost was tuned by randomised search on 2021 (gain +0.002), and k‑means segments added as predictors gave no lift.
- **Validation that mirrors deployment.** Models were trained on 2021 and tested once on 2022 (out‑of‑time); GroupKFold‑by‑authority was a spatial‑leakage and stability diagnostic; the model and threshold were chosen on a 2021 validation split only.
- **Interpretation and equity.** Probabilities were calibrated; a cost‑based threshold reflected the higher cost of a missed KSI; SHAP explained drivers; clustering cross‑checked segments; and a cluster‑robust nested logistic regression with a vehicle‑mile exposure denominator addressed equity.

## 5. Results and interpretation
- **Predictability.** Pre‑existing context is only weakly predictive: calibrated XGBoost reached a 2022 out‑of‑time ROC‑AUC of 0.61 (PR‑AUC 0.205), barely beating logistic regression, while the random forest over‑fitted (training AUC 0.913 vs validation 0.536).
- **Operating point.** At a 5:1 false‑negative:false‑positive cost the model recalls ~40% of KSI contexts (precision ~20%), rising to ~91% at 10:1 — a transparent budget lever.
- **Severity drivers.** SHAP and both clusterings agree that high speed limits, rural settings and poor lighting raise predicted severity — a fast (~62 mph) rural cluster has the highest KSI rate (0.19) — but these are predictive associations, not proven causes.
- **Equity (two channels).** Severity given a crash is mildly lower in deprived areas, but with exposure the absolute KSI burden per vehicle‑mile is broadly regressive (Q4/Q1 ≈ 1.5×, but peaking at Q3 — not strictly monotonic), driven by crash frequency and persisting across 2015–2024, so not a pandemic artefact.
- **Access to care.** A fourth external source (DfT JTS0506; DfT, 2021) adds an access dimension: after selected road‑context controls (urban/rural, speed, road type), longer modelled hospital drive‑time remains weakly associated with KSI (OR 1.07, cluster‑robust 95% CI 1.05–1.10) — a residual remoteness/access pattern, not evidence that distance causes severity.
- **Business meaning.** This answers the opening problem directly: public teams can rank fast‑rural contexts for severity‑reduction engineering and deprived / high‑exposure areas for frequency reduction, while insurers can read it as territorial bodily‑injury risk for loss‑prevention and fleet‑risk advice — both replacing raw counts with risk‑based triage.

## 6. Business insights and recommendations
- **Insight.** Where serious harm concentrates depends on the question asked: raw density favours urban cores, but exposure‑adjusted burden reveals which areas bear disproportionate harm per mile — though these per‑mile leaders are dense urban cores on a motor‑vehicle‑only denominator (a concentration signal, not a deprivation ranking) — relevant to public prioritisation and insurer territorial risk awareness.
- **Actions.** Public bodies should use it as a triage signal (not an automatic rule): prioritise high-speed rural contexts for engineering/speed/lighting review and deprived/high-exposure urban areas for traffic-calming and vulnerable-user protection, threshold set to review capacity. Insurers use the same ranking for loss-prevention, fleet-risk and reserving awareness - not pricing. Illustratively, ~580 of 10,000 contexts are flagged (~29 KSI avoided at a 5% effect).
- **Limitations.** Observational data (associations, not causal); exposure is motor‑vehicle only; deprivation is area‑level (ecological‑fallacy risk, reduced not eliminated by an LSOA check); performance suits prioritisation, not precise prediction.
- **Future enhancements.** Adding road‑geometry, traffic‑flow and pedestrian/cyclist exposure, top‑k targeting, and intervention‑rollout data for quasi‑experimental estimates would strengthen it.

## References
- xavierberge (n.d.) *Road Accident Dataset* [dataset]. Kaggle. Available at: https://www.kaggle.com/datasets/xavierberge/road-accident-dataset
- Ministry of Housing, Communities and Local Government (2019) *English Indices of Deprivation 2019* [dataset]. GOV.UK.
- Department for Transport (2025) *Road Safety Data (STATS19)* [dataset]. GOV.UK.
- Department for Transport (2024) *Road traffic statistics* [dataset]. GOV.UK.
- Office for National Statistics (2017) *Local Authority District to County (2017) Lookup* [dataset]. ONS Open Geography Portal.
- Department for Transport (2021) *Journey Time Statistics (JTS0506: travel time to nearest hospital)* [dataset]. GOV.UK.

---

*AI declaration (not included in the word count).*
- **Why.** A generative AI was used to accelerate scaffolding, structuring and debugging, and to stress‑test the design through a simulated critical review, freeing time for interpretation.
- **What.** It assisted with the notebook skeleton, join logic, evaluation and robustness code and their debugging, and with drafting and tightening this report.
- **Which.** Claude (Anthropic).
- **Where.** Used in code structure, comments and report drafting; all dataset choices, statistical decisions, claim wording and conclusions were reviewed, verified against the outputs, and finalised by me. The work remains my own and I retain intellectual ownership of it.
