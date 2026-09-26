# Customer Churn Analysis

A SQL + Python analytics project that consolidates raw customer, subscription, and support data into a single dataset, engineers churn-focused features, and quantifies the KPIs and drivers behind customer attrition.

## Overview

Subscription businesses live and die by retention. This project takes three disconnected data sources — customer profiles, subscription records, and support tickets — stored in a SQLite database, and turns them into a clean, analysis-ready dataset that answers:

- How many customers are churning, and how much revenue is at risk?
- Which plans, contract types, and regions churn the most?
- Does support experience (escalations, complaint volume) predict churn?
- Which customers are the highest risk *right now*, based on their churn score?

## Project Structure

```
.
├── churn_analysis.ipynb     # Full analysis notebook (cleaning → features → KPIs → charts)
├── customer_churn.db        # Source SQLite database (3 tables)
├── churn_data.csv           # Cleaned, merged, feature-engineered export
└── README.md
```

## Data Sources

The SQLite database (`customer_churn.db`) contains three tables, joined on `customerid`:

| Table | Key Columns |
|---|---|
| `db_customer` | `customerid`, `name`, `country`, `state`, `gender`, `dob`, `interests`, `pincode` |
| `db_subscription` | `customerid`, `subscription_start_date`, `subscription_type`, `renewal_date`, `plan_type`, `contract_type`, `cancellation_date`, `cancellation_reason`, `monthly_charges`, `cltv`, `churn_score` |
| `db_support` | `customerid`, `complaint_date`, `escalations`, `csat_score`, `comment` |

## Methodology

### 1. Extract
Load each SQLite table into a pandas DataFrame via `sqlite3` + `pandas.read_sql`.

### 2. Clean
- Renamed `name` → `customer_name` for clarity across joins
- Dropped low-value columns (`interests`, `pincode`, `col_1`, free-text `comment`)
- Converted all date columns (`dob`, `subscription_start_date`, `renewal_date`, `cancellation_date`, `complaint_date`) from text to `datetime`
- Standardized inconsistent category labels (`Men`/`Women` → `Male`/`Female`)
- Imputed missing `country` values using a `state → country` lookup built from complete records
- De-duplicated support tickets, keeping each customer's most recent complaint after aggregating a `complaint_count`

### 3. Merge
Left-joined `db_customer` → `db_subscription` → `db_support` on `customerid` into a single master DataFrame, then exported the result to `churn_data.csv` as a reusable checkpoint.

### 4. Engineer Features
| Feature | Definition |
|---|---|
| `churn flag` | `1` if `cancellation_date` is present, else `0` |
| `tenure_days` | Days between `subscription_start_date` and `cancellation_date` (or today, if still active) |
| `complaint_count` | Number of support tickets raised by each customer |
| `churn_risk` | Buckets `churn_score` into `low` (<50), `med` (50–69), `high` (≥70) |

### 5. Analyze & Visualize
Computed core KPIs with pandas, then visualized trends and distributions with Matplotlib and Seaborn (bar charts, a monthly churn trend line, and a correlation heatmap).

## Key Metrics

| Metric | Value |
|---|---|
| Churn Rate | 28.57% |
| Retention Rate | 71.43% |
| ARPU (avg. monthly revenue/user) | ₹18.85K |
| Avg. Customer Tenure | ~1,547 days (≈4.2 years) |
| Revenue at Risk (from churned users) | ₹73.94K/month |
| Escalation Rate | 19.05% |
| Avg. Complaints per User | 0.43 |
| Escalation ↔ Churn Correlation | +0.47 |

**Churn by plan type:** Basic 60.0% · Standard 22.2% · Premium 14.3%
**Churn by contract type:** Monthly 55.6% · Annual 8.3%
**Churn risk segments:** Low 13 · Medium 2 · High 6 (of 21 customers)

## Key Insights

1. **Plan and contract type are the strongest churn levers.** Basic-plan and month-to-month customers churn several times more often than Premium or Annual customers.
2. **Support escalations correlate with churn (+0.47).** Customers whose tickets get escalated are meaningfully more likely to cancel — support experience is a leading indicator, not just an operational metric.
3. **Risk is concentrated, not evenly spread.** 6 of 21 customers fall into the "high" churn-risk band and are the clearest candidates for proactive retention outreach.
4. **Sample size caveat:** this dataset has 21 customers, so state-level and other granular splits are directional, not statistically conclusive — treat them as hypotheses to validate on more data.

## Recommendations

- Prioritize retention offers and upgrade incentives for Basic-plan, monthly-contract customers.
- Offer a modest discount to migrate month-to-month customers onto annual terms.
- Invest in faster first-response and better agent training to reduce escalations.
- Route high-risk customers (`churn_score` ≥ 70) to a dedicated retention specialist ahead of renewal.

## Tech Stack

- **Python** — Pandas, NumPy
- **SQLite3** — relational data storage and extraction
- **Matplotlib & Seaborn** — visualization
- **Jupyter Notebook** — analysis environment

## How to Run

```bash
pip install pandas numpy matplotlib seaborn
jupyter notebook churn_analysis.ipynb
```

Run all cells top to bottom — the notebook reads directly from `customer_churn.db` and writes its cleaned output to `churn_data.csv`.

## Deliverables

- `churn_analysis.ipynb` — full, reproducible analysis
- `churn_data.csv` — cleaned, merged, feature-engineered dataset
- `Customer_Churn_Analysis.pptx` — executive-ready presentation of methodology, KPIs, and recommendations
