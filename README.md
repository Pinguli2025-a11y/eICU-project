# eICU-project
Comparative association of hypotension event frequency, time‑weighted average, and cumulative duration with hospital length of stay in critically ill adults: a retrospective cohort study using the eICU database

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

## Project Overview

This project investigates the association between different measures of hypotension burden and time to alive hospital discharge in critically ill ICU patients, using the **eICU Collaborative Research Database** (multi‑center, 200,859 ICU stays, 208 US hospitals).

**Key finding:** Greater hypotension exposure (MAP < 65 mmHg) is independently associated with delayed hospital discharge. The **time‑weighted average (TWA) hypotension burden** shows the strongest association (adjusted HR = 0.944, 95% CI: 0.934–0.955) and has a more pronounced effect during the first 7 days of hospitalization.

## Data Source

- **Database:** eICU Collaborative Research Database v2.0 (publicly available via PhysioNet)
- **Population:** 25,771 adult ICU patients with valid MAP monitoring and ICU stay ≥ 12 hours
- **Original data size:** 200,859 initial ICU stays

## Methods

### 1. SQL Data Extraction (PostgreSQL)

All data were extracted from a PostgreSQL instance hosting the eICU database. The following tables were queried using **complex SQL** (joins, aggregations, subqueries, window functions):

- `patient` – demographics, admission/discharge times, unit information
- `vitalperiodic` – mean arterial pressure (MAP) at 5‑minute intervals
- `admissiondx`, `pastHistory` – diagnoses and comorbidities
- `apachepatientresult`, `apachepredvar` – severity scores and outcomes

> **Example SQL logic:** Merging consecutive ICU stays within the same hospitalization when the gap between stays ≤ 120 minutes, and defining operative/non‑operative index stays.

### 2. Data Processing & Feature Engineering (R + DBI/RPostgres)

- **Time‑window merging** of ICU stays to create clinically meaningful critical illness episodes.
- **MAP cleaning:** Implausible values (<20 or >200 mmHg) set to missing.
- **Short‑gap interpolation:** Linear interpolation for missing MAP gaps ≤ 15 minutes (using pre/post 15‑minute windows).
- **Quality exclusion:** Stays with first valid MAP ≥ 6 hours after ICU admission or >50% missing MAP in the first 18 hours were removed.
- **Hypotension definition:** MAP < 65 mmHg.
- **Three exposure metrics calculated:**
  - **Hypotension episode count** (`hypo_event_n`)
  - **Cumulative hypotension duration** (`hypo_duration_hours`)
  - **Time‑weighted average (TWA) hypotension burden** (`hypo_twa`) = Σ(65 - MAP) × interval / total monitoring time

### 3. Statistical Analysis (R `survival` package)

- **Outcome:** Time to alive hospital discharge (event = alive discharge, censored = in‑hospital death)
- **Unadjusted models:** Univariate Cox regression for each exposure.
- **Adjusted models:** Multivariable Cox proportional hazards models controlling for age, BMI, sex, ethnicity, surgery type, and 7 comorbidities.
- **Piecewise time‑interaction models:** Split follow‑up at 7 days to assess early vs. late effects.
- **Sensitivity analysis:** Median BMI imputation to test robustness.

## Main Results

| Exposure | Adjusted HR (95% CI) for Alive Discharge | Effect on LOS |
|----------|-------------------------------------------|----------------|
| Hypotension episode count (per episode) | 0.987 (0.986–0.988) | Longer LOS |
| Cumulative duration (per hour) | 0.977 (0.976–0.978) | Longer LOS |
| **TWA burden (per mmHg)** | **0.944 (0.934–0.955)** | **Strongest association** |

- Hypotension occurred in **80.3%** of patients.
- Piecewise analysis: HRs were consistently lower (stronger association) during **0–7 days** than after 7 days (all p for difference < 0.001).
- Sensitivity analysis using median BMI imputation yielded nearly identical results.

> *HR < 1 indicates a lower instantaneous rate of alive discharge → longer hospital stay.*

## Repository Contents

| File | Description |
|------|-------------|
| `eICU_hypotension_analysis.Rmd` | Complete R Markdown source code (data extraction, cleaning, exposure calculation, survival analysis, figures) |
| `eICU_hypotension_analysis_report.pdf` | Final project report (PDF) describing background, methods, results, and discussion |
| `eICU_cohort_final.rds` | Final complete‑case dataset (25,771 patients) used for Cox modeling |
| `eICU_cohort_final.csv` | Same dataset in CSV format |
| `README.md` | This file |

**Note:** The raw eICU database is not included because it requires credentialed access via PhysioNet. You can request access [here](https://physionet.org/content/eicu-crd/2.0/).

## How to Reproduce

### Prerequisites

- R (≥4.0) with packages: `DBI`, `RPostgres`, `dplyr`, `tidyr`, `stringr`, `survival`, `ggplot2`, `survminer`
- PostgreSQL database with eICU-CRD v2.0 loaded (or you can modify connection settings)

### Steps

1. **Clone this repository**
   ```bash
   git clone https://github.com/your-username/eICU-hypotension-analysis.git
   cd eICU-hypotension-analysis
