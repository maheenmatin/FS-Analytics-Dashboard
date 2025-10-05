# Delivery Playbook (Consulting)

## Data Quality Checks
- **Schema & types:** ensure dates are typed; amounts numeric; IDs are strings.
- **Nulls:** check `SettlementDate`, `DueDate`, `Amount`.
- **Duplicates:** unique `TransactionID`; check `CustomerID+OrderDate` pair.
- **Ranges:** non-negative `Amount`; `OrderDate ≤ DueDate`; `SettlementDate ≥ OrderDate` or blank.
- **Outliers:** review orders beyond P99 AOV.
- **Drift:** monitor Late % and Avg Days to Settle against last 8 weeks.

## Stakeholder Cadence
- **Weekly (Ops/Collections, 30m):** Late %, Late £, top exposures; actions per region/segment.
- **Monthly (ELT, 45m):** Cohort Retention (30D), active customers trend, impact of nudges/policy changes.
- **Quarterly (Finance/Product):** payment terms & collections policy review by product/segment.

## Productionise (ETL → Model → Dashboard → BAU)
- **ETL:** daily ingest orders & payments; dedupe; standardise status logic.
- **Model:** star schema; incremental load on `OrderDate`.
- **Dashboard:** role-level filters (Collections vs Exec); tooltip-driven drill-through.
- **BAU:** data tests (freshness, row counts, null thresholds); alerting on breaches; SLA ownership.
