## PRD: Datetime Calculations ##

This can be maintained, generated by SQL, by Table(static list) or by open formula.

1. Dynamic year generator (MySQL):
From the current year back to the year 2010 (Using uder-defined variables). 
```
SELECT CAST(@_y := @_y - 1 AS UNSIGNED) AS y_key
FROM (SELECT @_y := YEAR(CURDATE())+1) a JOIN warehouse.users
WHERE @_y > 2010
```
**Note:** replace the table named `warehouse`.`users` with any table in your database
which has number of records more than the number of years required for the list.

2. Setting Date Range by open formula:
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

3. calculate MTD aggregations:

One of the datetime intelligence functions in BI projects is to compare
the aggregate amounts on the same day but different periods. For example
MTD (Month to Day) aggregate amount for every month in a year.

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
by default it could be `=DAY(TODAY())` in Pentaho open formula or `DAY(CURRENT_DATE())` in MySQL.
`ph_start_date` and `ph_end_date` are the start and end date of the comparison date range.


...TODO...