# Customer Churn Analysis: Python, SQL and Power BI

An end-to-end analytics project that takes a small relational customer database, cleans and joins it with Python and SQL, scales it into a 2,100-customer dataset, and presents churn, retention and revenue loss in an interactive Power BI dashboard.

## Table of Contents

- [Project Overview](#project-overview)
- [Important Note on the Data](#important-note-on-the-data)
- [Tech Stack](#tech-stack)
- [Workflow](#workflow)
- [Dashboard](#dashboard)
- [Metric Definitions](#metric-definitions)
- [What the Analysis Shows](#what-the-analysis-shows)
- [Repository Structure](#repository-structure)
- [How to Run](#how-to-run)
- [Limitations and Next Steps](#limitations-and-next-steps)
- [Author](#author)

## Project Overview

Subscription businesses lose revenue when customers cancel. This project answers four practical questions:

1. How many customers churn, and how much monthly revenue does that cost?
2. Which plans, contracts and regions have the highest churn rate?
3. How do support experience (CSAT, complaints, escalations) and churn relate?
4. What does a churned customer look like compared with a retained one?

## Important Note on the Data

The source data is a **21-customer sample** stored in a SQLite database (customer, subscription and support tables). To make dashboard analysis meaningful, the notebook generates a **synthetic dataset of 2,100 customers** with a fixed random seed (`np.random.seed(42)`).

Because the data is simulated, the patterns in it come from the rules used to create it, not from real customer behaviour:

| Field | How it was generated |
|---|---|
| Churn flag | Random draw with a probability built from the rules below (15% base, capped between 5% and 85%) |
| Plan and contract effect | Basic plan adds 20 points of churn risk; Monthly contract adds 15 points |
| Support effect | An escalation adds 25 points; a CSAT of 2 or lower adds 20 points. Escalated customers are assigned low CSAT and 2 to 5 complaints by construction |
| State, gender, subscription type | Assigned randomly, with no effect on churn |
| Cancellation reason | Chosen uniformly at random, so no reason is genuinely "top" |
| Churn score | Assigned from the churn flag (70 to 99 if churned, 1 to 44 if retained), so it mirrors the outcome and is **not** a predictive score |

Treat this project as a demonstration of the analytics workflow: data engineering, KPI design and dashboarding. It is not evidence about real customers.

## Tech Stack

| Area | Tools |
|---|---|
| Data storage and querying | SQLite, SQL |
| Cleaning and feature engineering | Python (Pandas, NumPy) |
| Visualisation and reporting | Power BI, DAX |
| Environment | Jupyter Notebook |

## Workflow

1. **Load.** Read four tables from a SQLite database into Pandas: `db_customer`, `db_subscription`, `db_support` and `customer_churn`.
2. **Clean.**
   - Renamed `name` to `customer_name` and converted date columns to datetime.
   - Standardised gender labels (`Men` and `Women` to `male` and `female`).
   - Filled missing countries using the known country for the same state.
   - Removed empty or unused columns (`col_1`, `comment`).
3. **Engineer features.**
   - `churn_flag`: 1 when a cancellation date exists, otherwise 0.
   - `complaint_count`: number of complaints per customer.
   - `tenure_days`: days from subscription start to cancellation, or to today for active customers.
4. **Join.** Left-joined subscription, customer and support tables on `customerid` (21 rows, 23 columns).
5. **Compute KPIs** on the sample: churn rate, retention rate, churn by plan, ARPU, average tenure, revenue lost and escalation rate.
6. **Scale up.** Generated 2,100 customers with the rules described above and saved them as CSV and as a SQLite table (`customer_churn_analytics`).
7. **Query with SQL.** Aggregated churn and revenue by plan:

   ```sql
   SELECT
       plan_type,
       COUNT(*)                                             AS total_customers,
       SUM(churn_flag)                                      AS churned_customers,
       ROUND(AVG(churn_flag) * 100, 2)                      AS churn_rate_pct,
       ROUND(AVG(monthly_charges), 2)                       AS arpu,
       SUM(CASE WHEN churn_flag = 1 THEN monthly_charges ELSE 0 END) AS revenue_lost
   FROM customer_churn_analytics
   GROUP BY plan_type
   ORDER BY churn_rate_pct DESC;
   ```

8. **Visualise.** Built a two-page Power BI dashboard on the scaled dataset.

## Dashboard

The Power BI file is `churn_analysis_dashboard.pbix`. It contains two pages with a shared slicer panel.

![Executive overview](screenshots/executive_overview.png)

![Churn drivers](screenshots/churn_drivers.png)

**Page 1: Executive overview**

- KPI cards: Total Customers, Churned Customers, Churn Rate, Retention Rate, Monthly Revenue Lost
- Churn rate by plan type, contract type and state
- Total customers by status (100% stacked bar)
- Top cancellation reasons (count of churned customers)
- Executive key insights panel

**Page 2: Churn drivers**

- Churn rate by CSAT score
- Churned vs retained snapshot (customers, monthly charges, CSAT, CLTV, tenure)
- Customer detail table (plan, subscription type, CLTV, CSAT, complaints, escalation, churn score, churn status)

**Interactivity:** slicers for plan, contract, subscription type, state and country apply across both pages, and a page navigator moves between them.

## Metric Definitions

| Metric | Definition |
|---|---|
| Churn rate | Churned customers divided by total customers |
| Retention rate | Retained customers divided by total customers |
| Monthly revenue lost | Sum of monthly charges for churned customers |
| ARPU | Average monthly charges per customer |
| Tenure | Days from subscription start to cancellation, or to today if still active |
| Escalation rate | Share of customers with an escalated complaint |

## What the Analysis Shows

On the 2,100-customer dataset, overall churn is about 44%. The patterns below match the rules used to generate the data, which is the expected result for a simulation and a useful check that the pipeline and DAX measures work correctly.

| Segment | Churn rate |
|---|---|
| Basic plan | about 55% |
| Standard and Premium plans | about 36% to 37% |
| Monthly contract | about 48% |
| Annual contract | about 38% |
| CSAT of 1 or 2 | about 79% |
| CSAT of 3 to 5 | about 31% to 39% |

For comparison, the original 21-customer sample showed 28.57% churn, with Basic at 60%, Standard at 22.22% and Premium at 14.29%. That sample is too small to draw conclusions from.

## Repository Structure

```text
.
├── Churn_Analysis.ipynb            # Cleaning, feature engineering, KPIs, data scaling, SQL
├── churn_analysis_dashboard.pbix   # Power BI dashboard
├── customer_churn_scaled.csv       # Synthetic 2,100-customer dataset
├── screenshots/                    # Dashboard images used in this README
└── README.md
```

The notebook also reads `customer_churn.db` (the original 21-customer sample) and writes `exported_churn_data.csv` and `customer_churn_scaled.db`.

## How to Run

**Requirements:** Python 3.10 or later, Jupyter, and Power BI Desktop (Windows) to open the dashboard.

```bash
pip install numpy pandas matplotlib seaborn jupyter
jupyter notebook Churn_Analysis.ipynb
```

1. Place `customer_churn.db` in the project folder, or skip to the data-generation cell to rebuild the scaled dataset.
2. Run the notebook from top to bottom.
3. Open `churn_analysis_dashboard.pbix` in Power BI Desktop. If it asks for the data source, point it to `customer_churn_scaled.csv`.

The generator uses today's date for cancellation dates and tenure, so re-running it can change the counts by a few rows. The CSV in this repository is the dataset the dashboard was built on.

## Limitations and Next Steps

**Limitations**

- The data is simulated, so the findings describe the simulation, not real customers.
- Churn score mirrors the churn outcome and should not be used as a predictor.
- Relationships are associations only. Support data may be recorded after a customer has already decided to leave.
- No currency is specified for monthly charges.

**Possible extensions**

- Apply the same pipeline to a real public churn dataset.
- Build a classification model (for example logistic regression or random forest) with a proper train and test split, leaving out the churn score.
- Add cohort and retention-curve analysis by subscription start month.
- Add exploratory charts in Python alongside the Power BI report.

## Author

**Devansh Joshi**
M.Sc. Computer Science, Central University of Rajasthan

- LinkedIn: [linkedin.com/in/contactdevanshJoshi](https://linkedin.com/in/contactdevanshJoshi)
- GitHub: [github.com/Devanshjoshi001](https://github.com/Devanshjoshi001)
