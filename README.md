# FS-Analytics-Dashboard

- Analytics dashboard using fictional historic data

- Auto-generated 3 CSV files ("transactions", "products", "customers") using GPT-5.

- Cleaned data manually using Excel
    - Added column for "due_date" (only "transaction_date" and "settlement_date" were
    generated). Assumed a simple homogenous scheme of giving all customers 48 hours to
    clear payments.
    - Clarified data inappropriate for a historic dataset by dropping "status" column
    and adding "late_settlement" column. Defined a "late" settlement as transaction
    settled after 48 hours.
    - For simplified retrospective analysis, assumed all transactions in the dataset had been
    successfully settled eventually. Chose a maximum lateness of 2 months for transactions in
    the dataset.

- Imported into Power BI
- Wrote DAX code 
- Created Date table and relationships between tables
- Created Calculations table to store all measures
- Used DAX throughout the process (collated in DAX_measured.md for reproducability)

## FinServ Transactions — Power BI Mini Dashboard

## Problem
Leaders need a fast read on revenue health, customer activity, and cash risk (late settlements), with enough detail to act by region/segment/product.

## Approach
- Two-page Power BI report over transactions + customers
- Star schema: `Transactions` fact with `Date`/`Customers` dimensions
- Time intelligence via dedicated `Date` table (marked as Date)
- KPIs: Total Revenue, Active Customers (30D), AOV, Late Settlement %, Avg Days to Settle, Cohort Retention (30D)

## Metrics (as of latest refresh)
- **Total Revenue:** £1.3M
- **Active Customers (30D):** 271
- **AOV:** £158
- **Late Settlement %:** 44.7%
- **Average Days to Settle:** 8.72
- **Cohort Retention (30D):** 58.5%
- **Late Settlement Revenue:** £563.9K

## Decisions enabled
- Prioritise collections where late-settlement revenue is highest (by region/segment/product).
- Focus on channels/products driving late settlements to reduce days-to-settle.
- Track returning-customer momentum to forecast demand and guide lifecycle messaging.

## How to run
1. Open `/pbix/FinServ-Mini.pbix` in Power BI Desktop.
2. Point to `/data/customers.csv`, `/data/transactions.csv` (or your source tables).
3. Refresh; export two screenshots to `/img/`:
   - `kpi.png` (KPI page)
   - `retention_trends.png` (Retention/Trends page)

## Screenshots
![KPI](img/kpi.png)
![Retention & Trends](img/retention_trends.png)

## Data
Synthetic sample data can be used to demo structure; replace with your live source as needed.

---

**Credits:** Built with Power BI + DAX.
