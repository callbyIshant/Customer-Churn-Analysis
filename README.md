# 📉 Customer Churn Analysis — Turning Attrition Data into Retention Strategy

> **Identifying at-risk customers, quantifying revenue exposure, and delivering actionable retention insights through end-to-end data analysis with Python and SQL.**

---

## 📌 Table of Contents

- [Business Problem](#-business-problem)
- [Tech Stack](#-tech-stack)
- [Project Structure](#-project-structure)
- [Methodology](#-methodology)
- [Key Findings & Business Impact](#-key-findings--business-impact)
- [What I Learned](#-what-i-learned)
- [How to Run Locally](#-how-to-run-locally)

---

## 🔍 Business Problem

In subscription-based businesses, **customer churn is the single greatest threat to sustainable revenue growth**. Losing a customer is not just a one-time transaction loss — it represents the entire future lifetime value of that relationship.

Studies consistently show that acquiring a new customer costs **5× to 7× more** than retaining an existing one, making churn reduction one of the highest-ROI activities a business can invest in.

This project simulates a real-world business scenario in which a data analyst is tasked with answering four critical questions for the leadership team:

| # | Business Question |
|---|---|
| 1 | What is our current churn rate, and how has it trended over time? |
| 2 | Which customer segments are most at risk of churning? |
| 3 | How much revenue is at risk from high-churn-probability customers? |
| 4 | What operational factors (e.g., support escalations) correlate with churn? |

The goal was to move beyond surface-level metrics and deliver insights that a product, marketing, or customer success team could act on immediately.

---

## 🛠️ Tech Stack

| Tool / Library | Role in Project |
|---|---|
| **Python 3.x** | Core programming language for all analysis |
| **SQLite3** | Relational database engine; source of raw data |
| **Pandas** | Data ingestion, cleaning, transformation, and aggregation |
| **Matplotlib** | Base visualization layer for charts and plots |
| **Seaborn** | Statistical visualizations (heatmaps, categorical plots) |
| **Jupyter Notebook** | Interactive development and narrative documentation |

---

## 📁 Project Structure

```
customer-churn-analysis/
│
├── notebooks/
│   └── churn_analysis.ipynb      # Main analysis notebook (full pipeline)
│
├── data/
│   └── customer_churn.db         # SQLite database (Customers, Subscriptions, Support)
│
├── outputs/
│   └── *.csv                     # Exported DataFrames and summary tables
│
├── images/
│   └── *.png                     # Saved chart exports for documentation
│
├── .gitignore                    # Standard Python gitignore
└── README.md                     # Project documentation (this file)
```

---

## 🔬 Methodology

The project followed a structured, end-to-end analytical pipeline. Each phase built directly on the last, mirroring a professional data analyst workflow.

---

### Phase 1 — Data Ingestion via SQL

The raw data resided in a relational SQLite database (`customer_churn.db`) containing three normalized tables: **Customers**, **Subscriptions**, and **Support**.

Rather than working with flat CSV files, data was extracted programmatically using `sqlite3` and loaded directly into Pandas DataFrames, simulating a real-world database environment.

```python
import sqlite3
import pandas as pd

conn = sqlite3.connect('data/customer_churn.db')

customers_df    = pd.read_sql_query("SELECT * FROM Customers", conn)
subscriptions_df = pd.read_sql_query("SELECT * FROM Subscriptions", conn)
support_df      = pd.read_sql_query("SELECT * FROM Support", conn)

conn.close()
```

---

### Phase 2 — Data Cleaning & Standardization

Raw data is rarely analysis-ready. The following cleaning operations were applied:

**Missing Value Treatment**

Null values were identified across all three tables. Missing entries in categorical columns (e.g., plan type, support tier) were imputed or flagged, while rows with missing primary identifiers were dropped to preserve relational integrity.

**Formatting Standardization**

Inconsistent capitalization in categorical columns like `gender` (e.g., `"male"`, `"Male"`, `"MALE"`) was resolved by applying `.str.strip().str.title()` normalization, ensuring clean groupBy aggregations.

**Date Parsing**

Subscription start and end dates were stored as plain text strings. These were converted to proper `datetime` objects using `pd.to_datetime()`, unlocking time-based trend analysis.

```python
subscriptions_df['start_date'] = pd.to_datetime(subscriptions_df['start_date'])
subscriptions_df['end_date']   = pd.to_datetime(subscriptions_df['end_date'])
```

---

### Phase 3 — Feature Engineering

This was the most analytically intensive phase. Several derived features were created to enrich the dataset before EDA.

**Churn Flag**

A binary column (`churned`: 1 / 0) was created by checking whether a customer's subscription end date had passed and their status was inactive, establishing the target variable for the entire analysis.

**Customer Tenure**

Calculated as the difference in days (then converted to months) between a customer's subscription start date and end date (or the analysis reference date for active customers). Tenure is a critical predictor of churn risk.

```python
subscriptions_df['tenure_days'] = (
    subscriptions_df['end_date'] - subscriptions_df['start_date']
).dt.days
```

**Churn Risk Score**

A composite risk score was mapped to each customer using a rule-based scoring system that factored in tenure bracket, support escalation history, and plan type. Customers were bucketed into **Low**, **Medium**, and **High** risk tiers.

**Revenue at Risk**

By cross-referencing the monthly revenue per customer with their churn risk tier, a "Revenue at Risk" figure was computed — quantifying the potential monthly recurring revenue (MRR) exposure from high-risk customers.

---

### Phase 4 — Table Merging & Deduplication

The three cleaned DataFrames were merged into a single master analytical table using Pandas `.merge()` operations on the common `customer_id` key.

Where duplicate entries existed (e.g., customers with multiple support tickets), records were consolidated using `.groupby()` with appropriate aggregation functions (e.g., `sum` for ticket count, `max` for escalation flag), and then `.sort_values()` was applied to ensure deterministic ordering before deduplication.

```python
master_df = customers_df \
    .merge(subscriptions_df, on='customer_id', how='left') \
    .merge(support_agg_df,   on='customer_id', how='left')
```

---

### Phase 5 — KPI Computation & EDA

With a clean master DataFrame in place, key performance indicators were calculated and exploratory analysis was conducted.

**Core KPIs Computed**

| KPI | Formula | Insight |
|---|---|---|
| **Churn Rate** | `churned_customers / total_customers` | Overall health benchmark |
| **Retention Rate** | `1 - Churn Rate` | Inverse signal; used for stakeholder framing |
| **ARPU** | `total_revenue / total_active_customers` | Revenue efficiency per customer |
| **Escalation Rate** | `escalated_tickets / total_tickets` | Support quality proxy |

**Visualizations Produced**

| Chart Type | Purpose |
|---|---|
| **Line Chart** | Monthly churn trend over time |
| **Bar Chart** | Churn rate broken down by customer segment and plan type |
| **Correlation Heatmap** | Relationship between numerical features (tenure, tickets, revenue) |
| **Categorical Plot (Seaborn)** | Multi-dimensional view of churn across gender, plan, and risk tier |

---

## 📊 Key Findings & Business Impact

> *Note: The following represent illustrative findings consistent with the type of patterns this analysis was designed to surface.*

**Finding 1 — Early-Tenure Customers Are Disproportionately at Risk**

Customers within their **first 90 days** of subscription exhibited the highest churn rates, suggesting that the onboarding experience is a critical intervention point. Targeting this cohort with proactive engagement (check-in emails, feature walkthroughs) could meaningfully reduce churn.

**Finding 2 — Support Escalations Are a Leading Indicator of Churn**

A strong positive correlation was observed between the number of escalated support tickets and churn probability. Customers who escalated 2 or more tickets were significantly more likely to churn within the following billing cycle. This signals an opportunity for the support team to flag at-risk accounts in real time.

**Finding 3 — High-Tier Plans Show Lower Churn but Higher Revenue Risk**

While premium plan customers churned less frequently (lower churn *rate*), the absolute revenue at risk concentrated in this segment was substantial due to high per-customer MRR. Losing a small number of premium customers has an outsized impact on total revenue — warranting a dedicated enterprise retention program.

**Finding 4 — Churn Is Seasonal, Not Uniform**

The monthly trend analysis revealed cyclical churn spikes, suggesting that renewal timing and seasonal factors influence cancellation behavior. This insight supports the business case for implementing time-sensitive retention campaigns around known high-churn periods.

---

## 🎓 What I Learned

**Technical Takeaways**

- Connecting Python directly to a relational SQLite database with `sqlite3` and executing SQL queries programmatically — a workflow directly transferable to production environments using `psycopg2` (PostgreSQL) or `pyodbc` (SQL Server).
- Designing a multi-step feature engineering pipeline that transforms raw transactional data into analytically rich, decision-ready features.
- Resolving complex deduplication challenges in merged DataFrames using `groupby` aggregation strategies — a common and non-trivial real-world data quality problem.
- Selecting the right visualization type for each analytical question (e.g., heatmaps for correlation, categorical plots for multi-dimensional comparisons), rather than defaulting to generic chart types.

**Analytical Takeaways**

- Framing data cleaning and EDA around specific business questions — not just exploring data for its own sake — keeps the analysis focused and ensures every output is stakeholder-relevant.
- The difference between a vanity metric (e.g., raw churn count) and an actionable metric (e.g., revenue at risk, escalation rate) is the foundation of impactful analysis.
- Quantifying business impact in financial terms (MRR at risk, retention cost savings) is essential for translating technical findings into executive decisions.

---

## ▶️ How to Run Locally

Follow these steps to reproduce the full analysis on your own machine.

**Prerequisites**

Ensure you have Python 3.8+ and `pip` installed. A virtual environment is strongly recommended.

**Step 1 — Clone the Repository**

```bash
git clone https://github.com/your-username/customer-churn-analysis.git
cd customer-churn-analysis
```

**Step 2 — Create and Activate a Virtual Environment**

```bash
# On macOS / Linux
python -m venv venv
source venv/bin/activate

# On Windows
python -m venv venv
venv\Scripts\activate
```

**Step 3 — Install Dependencies**

```bash
pip install pandas matplotlib seaborn jupyter
```

**Step 4 — Launch the Notebook**

```bash
jupyter notebook notebooks/churn_analysis.ipynb
```

**Step 5 — Run All Cells**

Inside Jupyter, navigate to **Kernel → Restart & Run All** to execute the full pipeline from data ingestion through to visualization.

> **Note:** The SQLite database file (`customer_churn.db`) must be present in the `data/` directory. The notebook is configured to read from this relative path automatically.

---

## 📬 Connect

If you have questions about this project or want to discuss the methodology, feel free to reach out via [LinkedIn](https://www.linkedin.com/in/your-profile) or open an issue on this repository.

---

*Built with Python · Pandas · Matplotlib · Seaborn · SQLite*
