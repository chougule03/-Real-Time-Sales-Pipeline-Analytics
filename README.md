# Real-Time Sales Pipeline Analytics
### End-to-End Data Engineering & BI Dashboard | PostgreSQL · dbt · Apache Airflow · Power BI

---

## Project Overview

A production-grade, automated sales pipeline analytics system built from scratch. Raw CSV data flows through a PostgreSQL database, gets transformed by dbt SQL models, is orchestrated daily by Apache Airflow, and surfaces as a 3-page executive Power BI dashboard  all running locally and deployed to AWS. 

This project demonstrates a caoplete analyst- to - engineer skill set : data modeling , pipeline automation , data quality testing and business intelligence

---

## Architecture

```
CSV Files (Faker-generated)
        │
        ▼
PostgreSQL (raw schema)
        │
        ▼
dbt (SQL transformations → analytics schema)
        │
        ▼
Apache Airflow (daily scheduler, runs inside WSL2/Ubuntu)
        │
        ▼
Power BI (3-page executive dashboard)
        │
        ▼
AWS EC2 (cloud deployment)
```

---

## Tech Stack

| Tool | Purpose |
|---|---|
| PostgreSQL 15 | Relational database    stores raw and transformed data |
| Python + Faker | Generates 5 realistic synthetic datasets |
| dbt (Data Build Tool) | SQL transformation layer    builds KPI tables, runs tests |
| Apache Airflow | Pipeline scheduler    runs dbt automatically every morning |
| WSL2 (Ubuntu) | Linux environment inside Windows    required for Airflow |
| Power BI Desktop | 3-page executive dashboard connected to PostgreSQL |
| AWS EC2 | Cloud deployment of the full pipeline |
| GitHub | Version control and portfolio proof |

---

## Dataset

All data generated using Python's **Faker** library. Represents a B2B sales organisation with realistic distributions.

| File | Rows | Key Columns |
|---|---|---|
| `customers.csv` | 1,000 | customer_id, company, industry, country, segment |
| `products.csv` | 50 | product_id, name, category, unit_price |
| `sales_reps.csv` | 30 | rep_id, name, region, team |
| `pipeline.csv` | 5,000 | deal_id, stage, value, close_date, probability |
| `orders.csv` | 3,200 | order_id, customer_id, product_id, quantity, revenue |

---

## Project Structure

```
sales-pipeline-analytics/
│
├── data/
│   ├── customers.csv
│   ├── products.csv
│   ├── sales_reps.csv
│   ├── pipeline.csv
│   └── orders.csv
│
├── dbt_project/
│   ├── models/
│   │   ├── staging/          # Raw data cleaning
│   │   ├── intermediate/     # Business logic
│   │   └── marts/            # Final KPI tables
│   ├── tests/                # dbt data quality tests
│   └── dbt_project.yml
│
├── airflow/
│   └── dags/
│       └── sales_pipeline_dag.py   # Daily scheduler
│
├── sql/
│   ├── 01_create_schema.sql
│   ├── 02_load_data.sql
│   └── 03_kpi_queries.sql
│
├── screenshots/
│   ├── 01_overview_dashboard.png
│   ├── 02_pipeline_analysis.png
│   └── 03_rep_performance.png
│
└── README.md
```

---

## Build Phases

| Phase | Task | Time |
|---|---|---|
| Phase 0 | Requirements & setup |  30 min |
| Phase 1 | Install WSL2 (Ubuntu inside Windows) | 30 min |
| Phase 2 | Install & configure PostgreSQL | 30 min |
| Phase 3 | Install Python, generate Faker datasets | 20 min |
| Phase 4 | Load CSVs into PostgreSQL | 45 min |
| Phase 5 | Build dbt models & tests | 2–3 hours |
| Phase 6 | Configure Apache Airflow in WSL2 | 1–2 hours |
| Phase 7 | Build Power BI 3-page dashboard | 3–4 hours |
| Phase 8 | Push to GitHub | 30 min |

---

## Power BI Dashboard    3 Pages

### Page 1    Executive Overview
- Total Revenue, Pipeline Value, Win Rate, Average Deal Size KPI cards
- Revenue trend by month (line chart)
- Pipeline by stage (funnel chart)
- Revenue by product category (bar chart)

### Page 2    Pipeline Analysis
- Deal stage conversion rates
- Pipeline value by region
- Win/loss analysis by industry segment
- Days to close distribution

### Page 3    Sales Rep Performance
- Revenue per rep (ranked bar)
- Win rate per rep
- Activity metrics (calls, emails, meetings)
- Rep vs. target comparison

---

## dbt Models

```sql
-- Example: marts/fct_pipeline_kpis.sql
-- Builds the core KPI table Power BI reads

SELECT
    DATE_TRUNC('month', close_date) AS month,
    stage,
    region,
    SUM(deal_value)                 AS total_pipeline_value,
    COUNT(deal_id)                  AS total_deals,
    SUM(CASE WHEN stage = 'Closed Won'
        THEN deal_value ELSE 0 END) AS won_revenue,
    ROUND(
        100.0 * COUNT(CASE WHEN stage = 'Closed Won' THEN 1 END)
        / NULLIF(COUNT(deal_id), 0), 1
    )                               AS win_rate_pct
FROM {{ ref('stg_pipeline') }}
GROUP BY 1, 2, 3
```

---

## Airflow DAG    Daily Automation

```python
# airflow/dags/sales_pipeline_dag.py
# Runs every morning at 6 AM    refreshes all KPI tables

from airflow import DAG
from airflow.operators.bash import BashOperator
from datetime import datetime

with DAG('sales_pipeline', schedule_interval='0 6 * * *',
         start_date=datetime(2024, 1, 1)) as dag:

    run_dbt = BashOperator(
        task_id='run_dbt_models',
        bash_command='cd /path/to/dbt_project && dbt run'
    )

    test_dbt = BashOperator(
        task_id='test_dbt_models',
        bash_command='cd /path/to/dbt_project && dbt test'
    )

    run_dbt >> test_dbt
```

---



---

## License

This project is open source under the [MIT License](LICENSE).
