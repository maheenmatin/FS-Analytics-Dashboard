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
