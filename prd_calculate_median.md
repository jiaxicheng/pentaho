## Calculate medium in PRD ##

Calculating median in MySQL is not a trival task. In PRD, you can get this number
from PDI data source (i.e. Univariate Statistics) if you need to clalculate a bunch of
descriptive statistics. 

The median can also be calculated by using PRD's internal open formula plus the BSF 
javascript script. This provided another option for more complex calculations.

**See below Howto:**

1. Get a list of amounts using for example SQL query and name it: `list_of_loan_amt`
```
SELECT loan_amount FROM warehouse.loans
WHERE loan_date >= ${p_start_date}
AND loan_date <= ${p_end_date}
ORDER BY loan_amount
```

2. Create a open formula named: `amt_list`, this will put the above sorted list of 
amounts into a line of comma-delimited text.
```
=CSVTEXT(MULTIVALUEQUERY("list_of_loan_amt");;",")
```

3. Create a "Bean-Scripting Framework(BSF) function(in javascript): f_median
split the text and calculate median.
```
var records = dataRow.get("amt_list").split(",");
var half = Math.floor(records.length/2);
if (records.length%2) 
    Number(records[half]);
else
    (Number(records[half-1]) + Number(records[half]))/2;

```

This is also an good example how to use BSF script and connect datarow using dataRow.get() method.
The same method can be extended to do calculations with more complexed logic within PRD itself.
