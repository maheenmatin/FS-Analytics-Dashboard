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
Days Late =
VAR s = transactions[settled_date]
VAR d = transactions[due_date]
RETURN
IF (
    NOT ISBLANK(s) && NOT ISBLANK(d) && s > d,
    DATEDIFF ( d, s, DAY ),
    BLANK()
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

Late Settlement Count :=
COUNTROWS ( FILTER ( transactions, transactions[late_settlement] = TRUE() && NOT ISBLANK ( transactions[settled_date] ) ) )

Late Settlement % = 
DIVIDE ( [Late Settlement Count], [Total Transactions] )

Late Settlement Revenue := 
CALCULATE( SUM ( transactions[net_amount]), transactions[late_settlement] = TRUE() && NOT ISBLANK ( transactions[settled_date] ) )

Late Settlement Revenue % = DIVIDE([Late Settlement Revenue], [Total Revenue])

Late Settlement % (by Settled Date) :=
    VAR LatestEOM = [Latest EOM with Data (by Settled Date)]
    VAR ThisEOM = MAX ( 'Date'[EndOfMonth] )
    RETURN
    IF (
        ISBLANK ( LatestEOM ) || ThisEOM > LatestEOM,
        BLANK(),
        VAR TotalTx =
            [Total Transactions (by Settled Date)]
        VAR LateTx =
            CALCULATE (
                COUNTROWS ( transactions ),
                USERELATIONSHIP ( 'Date'[Date], transactions[settled_date] ),
                transactions[late_settlement] = TRUE ()
                    && NOT ISBLANK ( transactions[settled_date] )
            )
        RETURN DIVIDE ( LateTx, TotalTx )
    )


// ---------------------------
// Cash timing KPI
// ---------------------------
Avg Days to Settle :=
AVERAGEX (
    FILTER ( transactions, NOT ISBLANK ( transactions[settled_date] ) ),
    DATEDIFF ( transactions[transaction_date], transactions[settled_date], DAY )
)


// ---------------------------
// Time-series helpers
// ---------------------------
Revenue by Month :=
CALCULATE ( [Total Revenue], VALUES ( 'Date'[EndOfMonth] ) )

Revenue YTD :=
CALCULATE ( [Total Revenue], DATESYTD ( 'Date'[Date] ) )

// Transaction date
Latest EOM with Data = 
    MAXX (
        FILTER ( VALUES ( 'Date'[EndOfMonth] ), [Total Transactions] > 0 ),
        'Date'[EndOfMonth]
    )

// Settled date
Latest EOM with Data (by Settled Date) :=
CALCULATE (
    MAXX (
        FILTER (
            VALUES ( 'Date'[EndOfMonth] ),
            CALCULATE ( [Total Transactions (by Settled Date)] ) > 0
        ),
        'Date'[EndOfMonth]
    ),
    USERELATIONSHIP ( 'Date'[Date], transactions[settled_date] )
)


// ---------------------------
// Retention KPI
// ---------------------------
Cohort Retention (30D EOM) :=
    VAR LatestEOM = [Latest EOM With Data]
    VAR AxisEOM = MAX ( 'Date'[EndOfMonth] )
    VAR ThisEOM =
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
// Settled-date variants
// ---------------------------
Total Transactions (by Settled Date) = 
CALCULATE ( [Total Transactions],
    USERELATIONSHIP ( 'Date'[Date], transactions[settled_date] )
)

Total Revenue (by Settled Date) :=
CALCULATE ( [Total Revenue],
    USERELATIONSHIP ( 'Date'[Date], transactions[settled_date] )
)
