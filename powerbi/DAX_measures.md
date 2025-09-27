// ---------------------------
// Date table
// ---------------------------
Date =
VAR MinDate = DATE ( YEAR ( MIN ( transactions[transaction_date] ) ) - 1, 1, 1 )
VAR MaxDate = DATE ( YEAR ( MAX ( transactions[transaction_date] ) ) + 1, 12, 31 )
RETURN
ADDCOLUMNS (
    CALENDAR ( MinDate, MaxDate ),
    "Year", YEAR ( [Date] ),
    "Month", FORMAT ( [Date], "MMM" ),
    "Month Number", MONTH ( [Date] ),
    "Year-Month", FORMAT ( [Date], "YYYY-MM" ),
    "Quarter", "Q" & FORMAT ( [Date], "Q" ),
    // UK fiscal year starting April
    "Fiscal Year", YEAR ( EDATE ( [Date], -3 ) ),
    "Fiscal Quarter", "Q" & FORMAT ( EDATE ( [Date], -3 ), "Q" ),
    "EndOfMonth", EOMONTH ( [Date], 0 ),
    "Week (ISO)", WEEKNUM ( [Date], 21 )
)


// ---------------------------
// Core volumes & revenue
// ---------------------------
Total Revenue :=
SUM ( transactions[net_amount] )

Total Transactions :=
COUNTROWS ( transactions )

Distinct Customers :=
DISTINCTCOUNT ( transactions[customer_id] )

Total Orders :=
DISTINCTCOUNT ( transactions[transaction_id] )

Average Order Value :=
DIVIDE ( [Total Revenue], [Total Orders] )


// ---------------------------
// Health & risk KPIs
// ---------------------------
Active Customers (30D) :=
VAR CurrEndInContext =
    MAXX (
        FILTER ( VALUES ( 'Date'[Date] ), [Total Transactions] > 0 ),
        'Date'[Date]
    )
RETURN
COALESCE (
    CALCULATE (
        DISTINCTCOUNT ( transactions[customer_id] ),
        DATESINPERIOD ( 'Date'[Date], CurrEndInContext, -30, DAY )
    ),
    0
)

Arrears % :=
VAR TotalTx =
    COUNTROWS ( transactions )
VAR ArrearsTx =
    CALCULATE ( COUNTROWS ( transactions ), transactions[status] = "In Arrears" )
RETURN
    DIVIDE ( ArrearsTx, TotalTx )

Late Settlement % :=
VAR TotalTx =
    COUNTROWS ( transactions )
VAR LateTx =
    CALCULATE ( COUNTROWS ( transactions ), transactions[late_settlement] = TRUE () )
RETURN
    DIVIDE ( LateTx, TotalTx )

Late Settlement % (by Settled Date) :=
VAR TotalTx =
    CALCULATE ( COUNTROWS ( transactions ), USERELATIONSHIP ( 'Date'[Date], transactions[settled_date] ) )
VAR LateTx =
    CALCULATE ( COUNTROWS ( transactions ),
        transactions[late_settlement] = TRUE (),
        USERELATIONSHIP ( 'Date'[Date], transactions[settled_date] )
    )
RETURN DIVIDE ( LateTx, TotalTx )


// ---------------------------
// Time-series helper
// ---------------------------
Revenue by Month :=
CALCULATE ( [Total Revenue], VALUES ( 'Date'[EndOfMonth] ) )

Revenue YTD :=
CALCULATE ( [Total Revenue], DATESYTD ( 'Date'[Date] ) )


// ---------------------------
// Retention (30D rolling, cohort-style)
// Compares customers active in the previous 30 days vs. also active in current 30 days.
// ---------------------------
Cohort Retention (30D) :=
VAR CurrEndInContext =
    MAXX (
        FILTER ( VALUES ( 'Date'[Date] ), [Total Transactions] > 0 ),
        'Date'[Date]
    )
VAR PrevWindow =
    DATESBETWEEN ( 'Date'[Date], CurrEndInContext - 60, CurrEndInContext - 31 )
VAR CurrWindow =
    DATESBETWEEN ( 'Date'[Date], CurrEndInContext - 30, CurrEndInContext )
VAR C1 =
    CALCULATETABLE ( VALUES ( transactions[customer_id] ), PrevWindow )
VAR C2 =
    CALCULATETABLE ( VALUES ( transactions[customer_id] ), CurrWindow )
RETURN
DIVIDE ( COUNTROWS ( INTERSECT ( C1, C2 ) ), COUNTROWS ( C1 ) )
