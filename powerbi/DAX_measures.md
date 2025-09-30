// ---------------------------
// Date table
// ---------------------------
Date =
VAR MinTx  = MIN ( transactions[transaction_date] )
VAR MinSet = MIN ( transactions[settled_date] )
VAR MaxTx  = MAX ( transactions[transaction_date] )
VAR MaxSet = MAX ( transactions[settled_date] )
VAR MinDate = DATE ( YEAR ( MIN ( MinTx, MinSet ) ) - 1, 1, 1 )
VAR MaxDate = DATE ( YEAR ( MAX ( MaxTx, MaxSet ) ) + 1, 12, 31 )
RETURN
ADDCOLUMNS (
    CALENDAR ( MinDate, MaxDate ),
    "Year", YEAR ( [Date] ),
    "Month", FORMAT ( [Date], "MMM" ),
    "Month Number", MONTH ( [Date] ),
    "YearMonthNumber", YEAR([Date]) * 100 + MONTH([Date]),
    "Year-Month", FORMAT ( [Date], "yyyy-MM" ),
    "Quarter", "Q" & QUARTER ( [Date] ),
    "Fiscal Year", YEAR ( EDATE ( [Date], -3 ) ),
    "Fiscal Quarter", "Q" & QUARTER ( EDATE ( [Date], -3 ) ),
    "EndOfMonth", EOMONTH ( [Date], 0 ),
    "Week (ISO)", WEEKNUM ( [Date], 21 )
)


// ---------------------------
// New columns for transactions table
// ---------------------------
// Days from transaction to settlement (blank if not settled)
Days to Settle =
VAR s = transactions[settled_date]
RETURN IF( ISBLANK(s), BLANK(), DATEDIFF( transactions[transaction_date], s, DAY ) )

// Show days only for late rows (cleaner for triage)
Days Late =
IF( transactions[late_settlement] = TRUE(), [Days to Settle], BLANK() )


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
VAR CurrEnd =
    MAXX(
        FILTER(ALL('Date'[Date]), 'Date'[Date] <= MAX('Date'[Date]) && [Total Transactions] > 0),
        'Date'[Date]
    )
RETURN
CALCULATE(
    DISTINCTCOUNT(transactions[customer_id]),
    DATESINPERIOD('Date'[Date], CurrEnd, -30, DAY)
)

Arrears Amount := CALCULATE(SUM(transactions[net_amount]), transactions[status] = "In Arrears")
Arrears % (Amount) = DIVIDE([Arrears Amount], [Total Revenue])

Late Settlement % = 
VAR TotalTx =
    COUNTROWS ( transactions )
VAR LateTx =
    CALCULATE ( COUNTROWS ( transactions ), transactions[late_settlement] = TRUE () )
RETURN
    DIVIDE ( LateTx, TotalTx )

Late Settlement % (by Settled Date) = 
VAR TotalTx =
    CALCULATE ( COUNTROWS ( transactions ), USERELATIONSHIP ( 'Date'[Date], transactions[settled_date] ) )
VAR LateTx =
    CALCULATE ( COUNTROWS ( transactions ),
        transactions[late_settlement] = TRUE (),
        USERELATIONSHIP ( 'Date'[Date], transactions[settled_date] )
    )
RETURN DIVIDE ( LateTx, TotalTx )


// ---------------------------
// Cash timing KPI
// ---------------------------
Avg Days to Settle :=
AVERAGEX (
    FILTER ( transactions, NOT ISBLANK ( transactions[settled_date] ) ),
    DATEDIFF ( transactions[transaction_date], transactions[settled_date], DAY )
)


// ---------------------------
// Time-series helper
// ---------------------------
Revenue by Month :=
CALCULATE ( [Total Revenue], VALUES ( 'Date'[EndOfMonth] ) )

Revenue YTD :=
CALCULATE ( [Total Revenue], DATESYTD ( 'Date'[Date] ) )


// ---------------------------
// Retention (30D rolling)
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

Cohort Retention (30D) - Monthly :=
AVERAGEX ( VALUES ( 'Date'[EndOfMonth] ), [Cohort Retention (30D)] )


// ---------------------------
// EOM snapshotted & CLAMPED measures
// ---------------------------
// Latest EOM with data (booked/transaction date)
Latest EOM With Data = 
MAXX (
    FILTER ( VALUES ( 'Date'[EndOfMonth] ), [Total Transactions] > 0 ),
    'Date'[EndOfMonth]
)

Cohort Retention (30D) - EOM Snapshot :=
VAR LatestEOM = [Latest EOM With Data]
VAR AxisEOM   = MAX ( 'Date'[EndOfMonth] )
VAR ThisEOM   =
    IF ( ISBLANK ( AxisEOM ) || AxisEOM > LatestEOM, LatestEOM, AxisEOM )
RETURN
IF (
    ISBLANK ( LatestEOM ),
    BLANK(),
    VAR CurrEnd =
        MAXX (
            FILTER (
                ALL ( 'Date'[Date] ),
                'Date'[Date] <= ThisEOM && [Total Transactions] > 0
            ),
            'Date'[Date]
        )
    VAR PrevWindow = DATESBETWEEN ( 'Date'[Date], CurrEnd - 60, CurrEnd - 31 )
    VAR CurrWindow = DATESBETWEEN ( 'Date'[Date], CurrEnd - 30, CurrEnd )
    VAR C1 = CALCULATETABLE ( VALUES ( transactions[customer_id] ), PrevWindow )
    VAR C2 = CALCULATETABLE ( VALUES ( transactions[customer_id] ), CurrWindow )
    RETURN DIVIDE ( COUNTROWS ( INTERSECT ( C1, C2 ) ), COUNTROWS ( C1 ) )
)


// ---------------------------
// Settled-date helpers for tooltips and clamping
// ---------------------------
Total Transactions (by Settled Date) = 
CALCULATE ( [Total Transactions],
    USERELATIONSHIP ( 'Date'[Date], transactions[settled_date] )
)

Cash Collected (by Settled Date) :=
CALCULATE ( [Total Revenue],
    USERELATIONSHIP ( 'Date'[Date], transactions[settled_date] )
)


// ---------------------------
// Latest EOM with data under settled-date context
// ---------------------------
Latest Settled EOM With Data = 
CALCULATE (
    MAXX (
        FILTER ( VALUES ( 'Date'[EndOfMonth] ), [Total Transactions] > 0 ),
        'Date'[EndOfMonth]
    ),
    USERELATIONSHIP ( 'Date'[Date], transactions[settled_date] )
)

// ---------------------------
// Clamped Late % by Settled Date (so line stops after last settled month with data)
// ---------------------------
Late Settlement % (by Settled Date) - Clamped = 
VAR LatestEOM = [Latest Settled EOM With Data]
VAR ThisEOM   = MAX ( 'Date'[EndOfMonth] )
RETURN IF ( ThisEOM > LatestEOM, BLANK(), [Late Settlement % (by Settled Date)] )
