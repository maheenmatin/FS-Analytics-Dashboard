# FS-Analytics-Dashboard

## Problem
Leaders need a fast read on revenue health, customer activity, and cash risk (late settlements), with enough detail to act by region/segment/product.

## Approach
- Three-page Power BI report over transactions + customers
- **Star schema:** `Transactions` fact with `Date`, `Customers` and `Products` dimensions
- Time intelligence via dedicated `Date` table (marked as Date)
- **KPIs:** Total Revenue, Active Customers (30D), AOV, Late Settlement %, Avg Days to Settle, Cohort Retention (30D)

## Metrics (as of latest refresh)
- **Total Revenue:** £1.3M
- **Active Customers (30D):** 271
- **AOV:** £158
- **Late Settlement %:** 44.7%
- **Average Days to Settle:** 8.72
- **Cohort Retention (30D):** 58.5%
- **Late Settlement Revenue:** £563.9K

## Decisions enabled
- **Target high-impact groups to cut late payments.** <br>
**Evidence**: Rank *segments/regions/products* by **Late Settlement %** and **Late Settlement Revenue**. <br>
**Action**: Focus process fixes where both are high (payment-reminder cadence, invoice clarity, term tweaks).
- **Shrink time-to-cash in the worst product/segment cohorts.** <br>
**Evidence:** Track **Avg Days to Settle** by month with a slicer on Product/Segment; spot the top laggards. <br>
**Action:** Run a 2-week experiment (e.g., pre-due reminders, different net terms) in the top 1–2 cohorts; success = ↓ in **Avg Days to Settle** trend.
- **Boost repeat usage where retention lags.**
**Evidence:** Compare **Cohort Retention (30D)** across Segment/Region/Product; sanity-check against **Active Customers (30D)**.
**Action:** Trigger lifecycle messaging for underperforming cohorts (win-back nudges, onboarding tips); success = +Δ in 30D retention.

## How to run
1. Open `/powerbi/dashboard.pbix` in Power BI Desktop.
2. Point to `/data/customers.csv`, `/data/products.csv` and `/data/transactions.csv` (or your source tables).
3. Refresh; export three screenshots to `/images/`:
   - `kpi.jpg` (KPI page)
   - `risk&retention.jpg` (Risk & Retention page)
   - `tables.jpg` (Tables page)
4. Open `.csv`, `.jpg`, `.pdf`, and `.md` files using compatible applications.

## Screenshots
![KPI](images/kpi.jpg)
![Retention & Trends](images/risk&retention.jpg)
![Tables](images/tables.jpg)

## Data
Synthetic sample data can be used to demo structure; replace with your live source as needed.

---

**Credits:** Built with Power BI + DAX.
