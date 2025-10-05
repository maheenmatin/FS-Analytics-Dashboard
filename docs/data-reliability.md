# Data Reliability

## Checks Implemented
- **Freshness:** last load time within 24h (alert if stale).
- **Completeness:** `TransactionID` duplicates < 0.1%; null `Amount` = 0.
- **Validity:** `OrderDate ≤ DueDate`; `SettlementDate` blank or `≥ OrderDate`.
- **Consistency:** status logic matches dates (late if `SettlementDate > DueDate`; in arrears if blank and `DueDate < today-30`).

## Tests to Automate
- Threshold tests via dbt/Great Expectations in CI.
- Slack/Email alerts on breach with owner + runbook link.
- Monthly metric definition review with Finance & Ops.