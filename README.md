# DAAI — UK Road‑Crash Severity & KSI Burden Analysis

*Warwick WBS · IB94V0 Data Analytics & Artificial Intelligence — individual project.*

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/XilaiWang/DAAI-road-safety-ksi-analysis/blob/main/road_safety_severity_analysis.ipynb)  **Main analysis** — predicting and explaining UK collision severity (KSI), with IMD deprivation, traffic‑exposure‑adjusted KSI burden, SHAP, clustering and robustness checks.

## ▶ Open in Colab (one‑click run)
Click the badge above. On Colab a setup cell fetches this repo (code + bundled `data/`) and installs `requirements.txt`, then the notebook runs top‑to‑bottom.
- **Public repo:** one‑click for anyone.
- **Private repo (current):** you are prompted once for a GitHub token (read access) to clone it — hidden input, never stored or committed. First enable *Colab → File → Open notebook → GitHub → Include private repositories*.

## Run locally
1. Python ≥ 3.10; `pip install -r requirements.txt`.
2. Keep the data files in `data/` (see `README.txt`).
3. Open `road_safety_severity_analysis.ipynb` and **Restart & Run All** (seeds fixed, relative paths).

## Contents
- **`road_safety_severity_analysis.ipynb`** — the analysis (16 sections; overview in notebook §0).
- **`data/`** — bundled datasets: Kaggle STATS19‑style collisions (2021–22), IMD 2019 (LAD + LSOA), DfT local‑authority traffic, ONS LAD→county lookup, and an official STATS19 2015–2024 subset.
- **`report.md`** — the 1000‑word report.
- **`requirements.txt`**, **`README.txt`** — dependencies and detailed run notes.

## What it does (one line)
A budget‑constrained resource‑prioritisation tool that ranks road contexts and local areas by KSI severity risk and exposure‑adjusted KSI burden — a triage / ranking signal for road‑safety teams and motor‑insurance risk managers (not pricing or real‑time prediction).
