## Pattern-1: Batch Similar tasks with Static List ##

Using `Data Grid` step to save a list of static mapping of the name/id from different objects 
on different storages. Run a generalized transformation which takes this list as parameter 
to handle the tasks.

In this example, we use one PDI job to update a flag `is_deleted` from all concerned Salesforce objects
to their related warehosue database.

Target:
1. Set up a list of Salesforce Objects, their correponding table name in warehouse DB
   the ID name used in the MySQL table.

2. For each object, retrieve all IDs which was labelled as 'IsDeleted' from Salesforce

3. update related table in warehoue database based on the list in (1)

**The Pentaho jobs**

following is the Pentaho kjb script:

```
   +-------+      +---------------------------------+
   | START | ---> | Transform: Get a list of Object |
   +-------+      +---------------------------------+
                             |
                             |     +---------------------------------+      +---------+
                             +---> | Transform: Delete entries in DB | ---> | Success |
                                   +---------------------------------+      +---------+

```
**Note:** On the job entry `Transform: Delete entries in DB`, double-click and setup the following:
+ In the `Options` tab: Execute every input row
+ In the `Parameters` tab, pass parameter values to sub transformation, add the following:


### Transform: Get a list of Object ###

```
   +-----------+      +---------------------+
   | Data Grid | ---> | Copy rows to result |
   +-----------+      +---------------------+
```

In this transformation, we setup two steps: 
+ `Data Grid` step to save a static table with three fields:
   + salesforce_object_name
   + sf_object_table_name
   + sf_object_id_name
For each salesforce object(Account, Lead, Contact, Opportunity, Task etc), fill in one entry for the 
above information.

+ `Copy rows to result` step
this is a common way to pass data rows to the later job entries

### Transform: Delete entries in DB ###

There are two steps, see below:
  
```
   +--------------------------------------+      +--------------------+
   | Saleforce Input: retrieve Object IDs | ---> | Execute SQL script |
   +--------------------------------------+      +--------------------+
```
1. The first one is `Salesforce Input` Step
In the "Settings" tab, select Specify query:
```
    SELECT ID from ${salesforce_object_name} WHERE IsDeleted = TRUE
```
In the `Fields` tab, specify the `Id` retrieved from the SOQL.

2. The 2nd is to execute the SQL to update warehouse DB (MySQL)
Set up the connection and then the following SQL:
```
DELETE FROM salesforce.${sf_object_table_name} WHERE ${sf_object_id_name} = '?';
```
Select the following options:
   + Execute for each row?
   + Variable substitution
in the `Parameters` box, add `Id`


### Note: ###
Pentaho supports database clustering so that the same Pentaho job/transformation can be used to
update multiple DB servers at the same time. this is very useful when you need to retrieve data 
from a third party API and dont want the overhead to run the same data multiple times.

Click the 'Edit...' button to enter the Database Connection dialog, Click `Clustering` in the Left-top box
click 'Enable Clustering', add the entry here. All connection must be using the 'Native (JDBC)' as the Access
method, `JNDI` will not work.

