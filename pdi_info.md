## Pentaho Data Integration (PDI or kettles) ##

Pentaho PDI is one of the ETL tools I've been using since March 2013. Some of the 
most useful things so far I've dealt with PDI include:

* Batching the PRD reports with `Pentaho Reporting Output` step, this can generate 
  hundreds of merchant statements in Excel formats in one run and saved lots of time 
  Accounting used to required and avoided the potential humen errors

* Data de-duplication with `Fuzzy-match` Step. with the carte cluster, the workload can be
  distributed onto 5-7 servers which greatly improved the speed and made a time-sensitive
  recurring project possible

* Salesforce data integrating with the central data warehouse (no need to read Salesforce 
  API manuals, all are simple with PDI)
  + Inserting / updating from the main Salesforce objects(lead, account, opportunities etc.)  
    into the warehouse database
  + Updating object flags(i.e. is_deleted) for data used in main production databases
  + Using the setup of database clustering, one Salesforce transform can update 
    multiple DB servers, thus reduce the overhead on the main Salesforce service.

* Connecting data on the Apache Hadoop HDFS system (using the same shim as HortonWorks HDP)
  + Apache Hive Server: `Table Input` -> Connection Type: 'Hadoop Hive2'
  + Spatk ThriftServer: the same as Apache Hive2
  + `Text file Input/Output` steps support hdfs:// path, by which PDI can read files on HDFS system directly
  + Data format In/Out: Avro and Parquet 
  + Sqoop to transform data between RDBMS and HDFS 

* As a Data source to support PRD/CDE reports. This can join data sources from different 
  servers/systems on the fly and create reports based on the actual live data. 
  It's also very useful when you need complex calculation when PRD is not flexible to handle. 
  for example, Waterfall chart and Heatmap chart which need a pivot table as data source. 

Basically almost all data sources that provide a JDBC driver can be connected into Pentaho PDI
and then you can update the warehouse database or feed the data directly into business reports
(in PRD or CDE). 

**Some interesting topics**
1. [Batching PRPT reports](pdi_automate_prd_reports.md)
2. [Creating waterfall chart](prd_pdi_waterfall-chart.md)



**TODO List:**
1. Setting up Spark Engine for PDI 7.1+, this is not urgent as I am processing most Hadoop 
   dataset with Hive or Spark thriftserver. will discover more details on my next related projects.
2. `Data Service` is one thing could be very useful in protecting data, need some testing.
3. Hadoop cluster with authentication (Kerberos or else).

More coming soon...

