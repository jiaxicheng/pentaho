## Pentaho PDI & PRD ##

Pentaho is a BI toolset which we are using to do many ETL tasks at the backend (data warehousing
cleansing, big data etc) and created many core BI reports (KPIs, cashflow etc) and general-purpose 
business reports (merchant statments, commissions etc.)

### Pentaho Data Integration (PDI or kettles) ###

Pentaho PDI is one of the ETL tools I've been using since March 2013. Some of the 
most useful things so far I've dealt with PDI included:

* Batching the PRD reports with `Pentaho Reporting Output` step
  with some extra steps, this can generate hundreds of merchant statements in Excel formats
  in one run and saved lots of time Accounting used to required and avoided the potential
  humen errors

* Data de-duplication with `Fuzzy-match` Step. with the carte cluster, the workload can be
  distributed onto 5-7 servers which can greatly improved the speed.

* Salesforce data integration with the main data warehouse (no need to read Salesforce 
  API manuals, all below are simple with PDI)
  + Inserting / updating from main Salesforce objects lead, account, opportunities etc  
    into the warehouse database
  + Updating object flags(i.e. is_deleted) for data used in main production databases
  + Using the setup of database clustering, one Salesforce transform can update 
    multiple DB servers, thus reduce the overhead on the main Salesforce service.

* Connecting data on the Apache Hadoop HDFS system (using the same shim as HortonWorks HDP)
  + Apache Hive Server: `Table Input` -> Connection Type: 'Hadoop Hive2'
  + Spatk ThriftServer: the same as the above
  + `Text file Input/Output` steps support also hdfs:// path, which you can read files on HDFS system directly
  + Data format In/Out: Avro and Parquet 
  + Sqoop to transform data between RDBMS and HDFS 

* As a Data source to support PRD/CDE reports. This can join data sources from different 
  servers/systems on the fly and create reports based on the actual live data. It's also very useful when you 
  need complex calculation when PRD is not flexible to handle. below are some examples:
  + Waterfall chart
  + Heatmap chart which need a pivot table as data source. 

Basically almost all data sources that provide a JDBC driver can be connected into Pentaho PDI
and then you can update the warehouse database or feed the data directly into business reports
(in PRD or CDE). 

### Pentaho Report Designer (PRD): ###

If you are looking for fancy UIs, PRD is probably not your solution. However, if you
care more about data, the productivity and simplicity, Pentaho report designer
absolutely deservers some of your time. It's especially well designed for the backend developers
who can thus save tons of loads from preparing ad-hoc reports.

* PRD is perfect for a listing report, knowing the basic `layout` design, editing PRD
  reports can be as easy as editing Word documents.
* Support both static(i.e. from Table) and dynamic(SQLs etc) parameters, which makes the PRD 
  reports user-friendly in terms of data
* Creating and automating PDF reports with ease. In the old days, I had to maintain Latex
  documents for business statement in the PDF format
* Creating complex Excel reports including crosstab, and integrting Excel formula directly
* Extendability, you can add Javascript(jquery) for enhanced UI (sortable, dragable etc)

**TODO:**
1. Setting up Spark Engine for PDI 7.1+, this is not urgent as I am processing most Hadoop 
   dataset with Hive or Spark thriftserver. will discover more details on my next related projects.

2. `Data Service` is one thing could be very useful in protecting data, need some testing.

3. Hadoop cluster with authentication (Kerberos or else).

More coming soon...

