## Calculate medium in PRD ##

Calculating median in MySQL is not a trival task. In PRD, you can get this number
by using PDI data source and a transformation with `Univariate Statistics` step.
This step also gets you a bunch of other descriptive statistics about your data.

Below is another way to calculate median in PRD by using PRD's internal open 
formula and the BSF Javascript script. This could open a door for more complex calculations 
completely inside PRD itself.

**See below Howto:**

1. Get a list of amounts using for example SQL query and name it: `list_of_loan_amt`
```
SELECT loan_amount FROM warehouse.loans
WHERE loan_date >= ${p_start_date}
AND loan_date <= ${p_end_date}
ORDER BY loan_amount
```

2. Create an open formula named: `amt_list`, this will put the above sorted list of 
amounts into a line of comma-delimited text.
```
=CSVTEXT(MULTIVALUEQUERY("list_of_loan_amt");;",")
```

3. Create a `Bean-Scripting Framework`(BSF) function (in javascript) named `f_median`
to split the text and calculate median.
```
var records = dataRow.get("amt_list").split(",");
var half = Math.floor(records.length/2);
if (records.length%2) 
    Number(records[half]);
else
    (Number(records[half-1]) + Number(records[half]))/2;

```
