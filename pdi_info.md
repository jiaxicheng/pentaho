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

**Some topics**
1. [Pattern-1: Batch Similar Tasks with Static List](pdi_pattern_batching_with_static_list.md)
2. [Pattern-2: Batch PRD Reports using Parameters](pdi_pattern_automate_with_parameters.md)
3. [Creating waterfall chart](prd_pdi_waterfall-chart.md)
4. [Data Deduplication Using Fuzzy Match Method](pdi_fuzzy_match.md)
5. [ETL Metadata Injection: Reuse Transformation templates](pdi_metadata_injection.md)

More coming soon...

