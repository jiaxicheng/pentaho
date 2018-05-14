## PRD: Datetime Calculations ##

Most of the BI reports involve dates, such as Month to Date, Year to Date, Same Period Last Month etc. 
With Pentaho PRD, these time intelligent values can be calculated through PRD's internal Open Formula, 
Tables or SQLs. They are often needed both in the parameters and in the real data queries. 

### PRD Parameters ###
1. Dynamic year generator using MySQL:
From the current year back to the year 2010 (Using user-defined variables). 
```
SELECT CAST(@_y := @_y - 1 AS UNSIGNED) AS y_key
FROM (SELECT @_y := YEAR(CURDATE())+1) a JOIN warehouse.users
WHERE @_y > 2010
```
**Note:** replace the table named `warehouse`.`users` with any table in your database
which has number of records more than the number of years required for the list.

2. Setting up Date Range by open formula (Parameters): 
```
p_date_range: [from Table]
+----+------------+
| ID | Value      |
+----+------------+
| 0  | Customized |
| 1  | MTD        |
| 2  | QTD        |
| 3  | YTD        |
| 4  | Last Month |
| 5  | Yesterday  |
+----+------------+

p_start_date:
=IF([p_date_range]=0
; [p_start_date]
; CHOOSE([p_date_range]
  ; DATE(YEAR(TODAY());MONTH(TODAY());1)
  ; DATE(YEAR(TODAY());INT((MONTH(TODAY())-1)/3)*3+1;1)
  ; DATE(YEAR(TODAY());1;1)
  ; DATE(YEAR(TODAY());MONTH(TODAY())-1;1)
  ; YESTERDAY()
  )
)

p_end_date:
=IF([p_date_range]=0
; [p_end_date]
; CHOOSE([p_date_range]
  ; TODAY()
  ; TODAY()
  ; TODAY()
  ; DATE(YEAR(TODAY());MONTH(TODAY());0)
  ; YESTERDAY()
  )
)
```

### PRD Data Queries ###

1. Calculate MTD aggregations:

One of the time intelligence in BI calculations is to compare
the aggregating amounts on the same date but different periods. For example
MTD (Month to Day) for every month in a year.

In MySQL:
```
SELECT CAST(LEFT(loan_date,7) AS CHAR) month
,      SUM(loan_amount) monthly_amt
,      SUM(IF(DAY(loan_date) <= {p_check_day_number}, loan_amount, 0)) mtd_amt
FROM warehouse.loans
WHERE loan_date >= ${ph_start_date}
AND loan_date <= ${ph_end_date}
GROUP BY 1
```

where `p_check_day_number` is the day number that you select to compare horizontally by months in the same year.
By default it could be `=DAY(TODAY())` in Pentaho's Open Formula or `DAY(CURRENT_DATE())` in MySQL.
`ph_start_date` and `ph_end_date` are the start and end date of the comparison date range.


...TODO...
