# Venture Capital Market Intelligence

**Advanced Databases Final Project | Purdue University**

*Muhammad Shaharyar Amjad & Tashfeen Khan*

---

## Overview

A multi-database analytics pipeline built on the Crunchbase dataset (~18,800 companies) to surface venture capital investment patterns and quantify how founding team track records influence funding outcomes.

Data flows from **Google Cloud Storage → MongoDB Atlas** (NoSQL document store with aggregation pipelines) → **BigQuery** (SQL window function analytics), combining the strengths of both databases for a complete market intelligence report.

---

## Research Question

**Does a management team's track record — prior funding raised, exit experience, serial founder status — predict how much capital their next venture will attract?**

---

## Key Findings

1. **Prior funding track record is the strongest predictor** — teams with higher `team_past_funding_exposure` consistently attract more capital (r = 0.551, p < 0.0001)

2. **High-value exit experience doubles funding** — companies with at least one executive who participated in a $50M+ exit raise nearly twice as much on average (statistically significant t-test)

3. **Serial founder status alone is weak signal** — having a serial founder does not meaningfully increase funding unless prior ventures resulted in significant exits

4. **C-level titles and breadth of experience don't predict funding** — quality and magnitude of past financial success matter more than credentials

5. **Software, mobile, web, and cleantech show sustained investor interest** — consistent funding acceleration from mid-2000s onward across all four sectors

---

## Architecture

```
Google Cloud Storage (crunchbase_companies.json — JSONL)
        │
        ▼
MongoDB Atlas (companies collection)
        │
        ▼
Task 1: Aggregation Pipeline → companies_analysis collection
        │   - Unwind funding_rounds → total_funding, latest_funding_year
        │   - Filter relationships → founder_count
        │
        ▼
Task 2: MongoDB Queries
        │   - Sector funding trends over time
        │   - Top funded companies by category
        │
        ▼
Task 3: BigQuery Migration (PyMongo → BigQuery)
        │   - SQL window functions
        │   - Cumulative funding trends by sector and year
        │
        ▼
Task 4: Management Team Analysis
        │   - C-level experience rate, multi-company experience
        │   - Serial founder detection, exit quality scoring
        │   - Correlation analysis + OLS regression + Random Forest
        │
        ▼
Task 5: Executive Summary — Horizon Ventures Investment Recommendations
```

---

## Tasks

### Task 1 — MongoDB Data Transformation Pipeline
- Ingested 18,801 companies from GCS (JSONL) into MongoDB Atlas via batch processing (1,000 docs/batch)
- Built aggregation pipeline to compute:
  - `total_funding` — sum across all funding rounds
  - `latest_funding_year` — most recent round year
  - `founder_count` — regex filter on relationships titles

### Task 2 — MongoDB Analytical Queries
- Sector funding trends over time (category_code × funded_year)
- Top funded companies by category
- Funding round type distribution

### Task 3 — BigQuery SQL Analytics
- Migrated cleaned `companies_analysis` from MongoDB to BigQuery
- SQL window functions: cumulative funding by sector and year, ranking within categories
- Analysis: how founding team track records influence venture funding outcomes

### Task 4 — Management Team Impact Analysis
Built 10 management team metrics from relationship data:
- `c_level_experience_rate` — % of team with prior C-level roles
- `multi_company_experience_rate` — % with 2+ prior companies
- `has_serial_founder` — binary flag
- `premium_exit_experience_count` — exits > $50M
- `team_past_funding_exposure` — total prior capital raised by team
- `team_exit_value_exposure` — total prior exit value by team
- `avg_exit_quality_score` — normalized exit quality
- `executive_turnover_rate` — leadership stability indicator
- `current_team_size` — organizational scale
- `funding_velocity_years` — time between founding and first funding

**Models:**
- OLS Regression (statsmodels) on log-transformed funding
- Random Forest Regressor — feature importance validation
- Pearson correlation + t-tests for significance testing

### Task 5 — Executive Summary
Investment recommendations for Horizon Ventures based on data-driven findings.

---

## Results

| Metric | Finding |
|---|---|
| Team past funding exposure correlation | r = 0.551 (p < 0.0001) |
| Team exit value exposure correlation | r = 0.584 (p < 0.0001) |
| $50M+ exit experience funding premium | ~2x mean funding vs no exit experience |
| C-level experience rate correlation | Not statistically significant |
| Serial founder correlation | Weak without exit outcome |
| Random Forest Test R² | ~0.35 on log-transformed funding |

---

## Quick Start

### Prerequisites
- MongoDB Atlas account (free tier — clear sample databases before loading)
- Google Cloud account with access to `advdb-course-data` bucket
- BigQuery project

### Setup

```bash
pip install pymongo google-cloud-storage pandas numpy scikit-learn statsmodels matplotlib seaborn scipy
```

```python
# Replace placeholder in notebook Cell 2
MONGODB_URI = "mongodb+srv://<username>:<password>@cluster0.xxxxx.mongodb.net/"
```

### Run Order
1. **Cell 1-4:** Connect to MongoDB, download from GCS, load companies
2. **Task 1:** Run aggregation pipeline → creates `companies_analysis`
3. **Task 2:** MongoDB queries and visualizations
4. **Task 3:** Migrate to BigQuery, run SQL window functions
5. **Task 4:** Management team feature engineering + models
6. **Task 5:** Executive summary

---

## Project Structure

```
├── advdb_gp_amjad_khan.ipynb    # Full pipeline notebook with outputs preserved
└── README.md
```

> Note: Raw data files are not included — they are loaded from GCS into MongoDB at runtime. The notebook preserves all outputs from the final run for reference.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Data source | Google Cloud Storage (JSONL) |
| NoSQL store | MongoDB Atlas |
| Query language | MongoDB Aggregation Pipeline |
| Relational warehouse | Google BigQuery |
| Analytics | SQL window functions, pandas |
| ML models | Random Forest, OLS Regression (statsmodels) |
| Visualization | matplotlib, seaborn |
| Language | Python 3 |
