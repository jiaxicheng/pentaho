## Creating Waterfall chart ##

The waterfall chart is a nice way to show the amout increment/decrement over time periods.
For N observations, it starts from an initial abosolute amount and then shows the amount differences over 
the next N-2 points thus you can have a view on which month has upsale or downsale and performance
based on the incresement/decresement are very clear. The last point 'N' will show the final abosolute amount
so you can compare start/end points on the abosolte values (not delta)

For the amount used in waterfall chart, we will need to calculate the numbers differently
between the start/end points and the rest. With Pentaho, we can have different ways:

### Using MySQL User-defined Variables ###
The key is to cache funded amount in the previous month, so that we can
calculate the amount difference between two adjacent months (delta_funded).
Using MySQL user-defined variables, we can save $prev_funded and use it
in the next SELECT loop.

```
SELECT x.funding_month
,      CAST(x.delta_funded AS DECIMAL(12,2)) AS delta_funded
FROM (
    SELECT IF(@_pre_funded = 0, 'Start', b.funding_month) funding_month
    ,      b.amt_funded
    ,      b.amt_funded - @_pre_funded  AS delta_funded
    ,      (@_pre_funded := b.amt_funded) AS prev_funded
    FROM (SELECT @_pre_funded := 0) a 
    JOIN ( 
	SELECT CAST(LEFT(date_funded,7) AS CHAR) funding_month
	,      SUM(funded_amount) amt_funded
	FROM warehouse.funding
	WHERE date_funded >= ${ph_start_date}
	AND date_funded <= ${ph_end_date}
	AND country = ${p_country}
	AND (${p_product_type} = 'Both' OR product_type = ${p_product_type})
	GROUP BY 1
    ) b

    UNION ALL

    SELECT 'End', 0, @_pre_funded,0
) AS x
```
**Caveats:**

MySQL user-defined variables have a dynamic typing, you can not explicitly declare their data types, 
and the CAST() does not work all the time on these variables.

In the above SQL, the outerside temporary table-x is to resolve a problem that 
MySQL forces the delta_funded as 'String' and using CAST( AS DECIMAL(12,2)) 
does not work directly.

The data types are not an issue with MySQL older version 5.0 and 5.1.  Newer
version have more strict type checking and you must make sure data types
matches what we need in Pentaho.

### Using PDI as data source ###

PDI as data source can provide more flexibility than the PRD's query engine
+ 'Table Input' step can take everything as parameters while in PRD, table
  names, keywords can not be parameterized, this makes something like order
  by <col-name> impossible (you can still order by <index-id> where index-id 
  aligned to the position in the SELECT list)
+ With PDI as data source, by default, you should not include multi-select parameters.
  using some tricks in PRD, multi-select parameters chould be possible, but need
  to make sure escapling properly to avoid SQL parsing mistake or enjection

**Below is the Howto:**

1. From main menu, Edit -> Settings... -> Parameters
   Add necessary parameters, in our example, add: ph_start_date, ph_end_date, p_country and p_product_type
   set up default value so you can debug the transform.

2. In the 'Table Input' step, add connection and then the following SQL:
```
SELECT x.funding_month
,      x.amt_funded
FROM (
    SELECT IF(LEFT(date_funded,7) = LEFT('${ph_start_date}',7), 'Start',LEFT(date_funded,7)) funding_month
    ,      SUM(funded_amount) amt_funded
    FROM warehouse.funding
    WHERE date_funded >= '${ph_start_date}'
    AND date_funded <= '${ph_end_date}'
    AND country = '${p_country}'
    AND ('${p_product_type}' = 'Both' OR product_type = '${p_product_type}')
    GROUP BY 1
    ORDER BY NULL
) AS x

UNION ALL

SELECT 'End', 0 
```
**Note:** we want the first month shown as 'Start' and ordered as the first one, `GROUP BY` sorts
the result by default, adding `ORDER BY NULL` will disable the sorting.

make sure to tick: Replace variables in script?

3. Add 'Modified Java Script Value' step with the following code:

```
var delta_funded = amt_funded - (prev_funded||0);
var prev_funded = amt_funded;
if (funding_month == 'End') {
    delta_funded = -delta_funded
}
```
At the last entry, delta_funded should be the actual amount(-delta_funded) since we
have the funded_amount set to zero at 'End'

Click 'Get variables' button at the bottom and make sure the Fields added have proper types, i.e. "Number"  
with Length = 12 and recision= 2

4. Add a 'Select values' step
this step is optional, in case you want to enforce data types or rename fields.

5. save the Ktr and run to check the returning dataset.

**On the PRD end**

1. Add the ktr as a Resource:
   + From main menu: File -> Resources... -> Import 
   + Add the ktr script created above and name it for example 'get_amount'
   + Select Content-type: text/xml

2. Add ktr as the reporting data source:
   + From main menu, Data -> Add Datasource -> Pentaho Data Integration
   + Click the '+' button to add a new query
   + In the File box, enter the resouce name 'get_amount'
   + All 3 steps now should shown in the Steps box, select 'Select values'
   + click 'Edit Parameter' to map parameters from PRD to PDI

3. Under the 'Data' tab of the main window, right click on the newly defined data source
   named 'Query-1' and then select 'Select Query'. now this data source will become the main
   data source. you can verify this from 'Structure' tab -> 'Attributes' tab -> 'query' section -> 'name' attribute
4. drag the `chart` icon from the left-side toolbar onto 'Report Header', double-click 
   the chart and in the 'Edit Chart' dialogue, select 'Waterfall Chart(Image)'
5. setup the following two major items:
   + category-column: the horizontal-axis --> funded_month
   + value-columns: the numbers for the vertical axis --> delta_amount
