# Devlog

## Objective
Create a mini dashboard using fictional historic data for financial services transactions.

## Extraction
Auto-generated 3 `.csv` files (`customers.csv`, `products.csv` and `transactions.csv`) using GPT-5.

## Transformation
Cleaned data manually using **Excel**:
- Added column for `due_date` (only `transaction_date` and `settlement_date` were generated). Assumed a simple homogenous scheme of giving all customers 48 hours to clear payments.
- Clarified data inappropriate for a historic dataset by dropping `status` column and adding `late_settlement` column. Defined "late" settlements as transactions settled after 48 hours.

- **Note:** For simplified retrospective analysis, assumed all transactions in the dataset had been successfully settled eventually. Chose a maximum lateness of 2 months for transactions in the dataset.

## Loading
Imported `customers.csv`, `products.csv` and `transactions.csv` into **Power BI**:
- Created a `Date` table and added new columns to `transactions` table.
- Created relationships between tables (appropriate multiplicities, active and inactive).
- Created `Calculations` table to store all measures
- Used **DAX** throughout the process (collated in `powerbi/dax.md` for reproducability).

## Analysis
- Created page 1 for KPIs, page 2 for risk and retention, and page 3 for tables.
- Used synchronised slicers, cards, tables, line charts, clustered column charts.
- Configured tooltips as appropriate.
- Exported report to `slides/dashboard.pdf` with individual pages exported to `images/kpi.jpg`, `images/risk&retention.jpg` and `images/tables.jpg`.
- Wrote supporting material using templates given by GPT-5: `slides/consulting-report.pdf`, `docs/playbook.md` and `docs/data-reliability.md`
