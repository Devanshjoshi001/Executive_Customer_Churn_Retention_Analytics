📊 Executive Customer Churn & Retention Analytics
An end-to-end data analytics solution designed to analyze customer churn drivers, quantify revenue at risk, and deliver an interactive executive dashboard designed with modern SaaS visual guidelines.
📌 Executive Summary
Customer churn directly impacts recurring revenue and long-term business growth. This project combines Python (Pandas) for data manipulation, SQLite for relational database querying, and Power BI for executive visual storytelling.
Key Highlights:
Revenue at Risk: Identified churn trends across subscription tiers to isolate high-value customer loss.
Plan Tier Insights: Granular breakdown of churn rates (churn_rate_pct) and Average Revenue Per User (ARPU) across plan_type cohorts.
Modern SaaS Design: Engineered a custom 2-page Power BI dashboard (#F1F5F9 slate canvas, white container cards, pastel indicator badges, dark navy typography).
🖼️ Dashboard Preview
Page 1: Executive Churn Overview
High-level overview of active KPIs, churn trends, MoM deltas, and cohort segmentation.
Page 2: Churn Drivers & Risk Analysis
Detailed breakdown of customer risk factors, contract lengths, and account characteristics.
🛠️ Tech Stack & Tools
Data Processing & Analysis: Python (pandas, numpy, sqlite3)
Database Engine: SQLite (customer_churn_scaled.db)
Business Intelligence: Power BI Desktop (DAX formulas, Custom SaaS Styling)
Documentation & Data Source: Microsoft Excel (.xlsx), Markdown
📁 Repository Structure
churn-analysis-dashboard/
│
├── data/
│   ├── customer_churn_scaled.xlsx             # Raw dataset source
│   └── customer_churn_scaled.db               # SQLite database file
│
├── notebooks/
│   └── Churn_Analysis.ipynb                   # Data cleaning, EDA, and SQL integration
│
├── dashboard/
│   └── churn_analysis_dashboard.pbix          # Interactive Power BI report
│
├── docs/
│   ├── Churn_Analysis_Project_Documentation.docx
│   └── assets/                                # Screenshots for README
│       ├── dashboard_page1.png
│       └── dashboard_page2.png
│
├── .gitignore                                 # Git ignore file
└── README.md                                  # Repository landing page



🗄️ Database Querying & SQL Analysis
The core analytical queries are executed against the customer_churn_analytics table in SQLite.
Example SQL Query: Plan Type Performance & Revenue at Risk
SELECT 
    plan_type,
    COUNT(*) AS total_customers,
    SUM(churn_flag) AS churned_customers,
    ROUND(AVG(churn_flag) * 100, 2) AS churn_rate_pct,
    ROUND(AVG(monthly_charges), 2) AS arpu,
    SUM(CASE WHEN churn_flag = 1 THEN monthly_charges ELSE 0 END) AS revenue_at_risk
FROM customer_churn_analytics
GROUP BY plan_type
ORDER BY churn_rate_pct DESC;



🚀 How to Run & Reproduce
1. Clone the Repository
git clone https://github.com/your-username/churn-analysis-dashboard.git
cd churn-analysis-dashboard



2. Run the Python & SQL Notebook
Open Jupyter Notebook or VS Code.
Navigate to notebooks/Churn_Analysis.ipynb.
Ensure Python dependencies are installed:
pip install pandas numpy



Execute the cells to connect to data/customer_churn_scaled.db and review the outputs.
3. Open the Power BI Dashboard
Open Power BI Desktop.
File  Open  Navigate to dashboard/churn_analysis_dashboard.pbix.
If prompted to update data source paths, point to data/customer_churn_scaled.xlsx or data/customer_churn_scaled.db.
🤝 Contact & Connect
Author: Devansh Joshi
LinkedIn: linkedin.com/contactdevanshjoshi
