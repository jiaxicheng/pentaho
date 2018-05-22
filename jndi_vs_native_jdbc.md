## DB Connection with JNDI or Native(JDBC) ##

Pentaho provides the standard JDBC interface to communicate with different data stores including
RDBMS database (i.e. MySQL, Oracle, PostgreSQL etc.) and Bigdata store (i.e. Apache Hive
Spark ThriftServer etc.). With Pentaho, JDBC access can be setup in two forms: `Native(JDBC)` and `JNDI`.

JNDI is a name directive by which Pentaho can look up from a centralized configuration for all 
information required to establish a JDBC connection. Compared to Native(JDBC), JNDI hides all 
connection parameters from the developers. While using the same JNDI name, each developer can set 
up JDNI directives from his/her own workstation which makes the deployment from staging to
production smoothly without the need to modify the JDBC links. 

Below list some benefits from using JNDI connections:

+ Avoid saving your DB credentials (i.e. password) in your transformation/job/report files, especially 
  in CDE report where password is shown as plain text

+ JNDI provides flexibility and fast turnaround for failover. For example, to adjust the DB link for 
  all reports on a BA server, with JNDI you can do it all in one place, while with Native(JDBC) you 
  have to modify every single PRPT/CDE report.

+ Use JNDI as parameters:
  + JNDI can be easily implemented as parameter for PDI transformation/jobs, by which you 
    can use the same transformation/job to update different DB servers

  + In PRD/CDE reports, JNDI can be added as a hidden parameter, for developer to switch between
    production DB and staging DB

**Note:** Native(JDBC) can also be used as parameters, but you end up setting up JDBC links on all 
transformation/job/report files, or have to feed username/password from the command line
or report parameter list which is less secure than managing the credential information in a centrolized 
location when using JNDI. 

To use JNDI, you will need to set up the properties file for the connection parameters:
1. Find the properties file for different tools:
   + BI server: 
     + For PDI transformation/jobs running on BA server: 
       `${BA-Server-Home}\pentaho-solutions\system\simple-jndi\jdbc.properties`
     + For PRD and CDE reports
       1. Login to the BA server website
       2. Click "Manage Data Sources" button
       3. Click '+' and then "JDBC"

   + PDI: `data-integration\simple-jndi\jdbc.properties`
   + PRD: the configuration file is located at user's $HOME folder: 
     `$HOME\.pentaho\simple-jndi\default.properties`

2. Below list the required configurations for an MySQL JNDI entry named `warehouse`. Make sure 
   the MySQL connector jar file (mysql-connector-java-x.x.xx-bin.jar) is copied to the proper lib folder)
```
warehouse/type=javax.sql.DataSource
warehouse/driver=com.mysql.jdbc.Driver
warehouse/url=jdbc:mysql://<mysql_server>:3306/warehouse
warehouse/user=myusername
warehouse/password=mypassword
```

### Compare using JNDI parameter with PDI DB clustering ###
+ With DB clustering, you can run the script once to update all DB servers in the same cluster at the same time. 

+ Using JNDI as parameter, you run the same script once for a single DB. It's useful when you need 
  to isolate your workflow, for example, update staging and production at different timing.

**Note:**

+ `Database clustering` only works with Native(JDBC), JNDI will not work.
+ Some databases, i.e. `Hadoop Hive 2` (including Apache HiveServer2, Spark ThriftServer) 
  only support Native(JDBC)


**Bottom line:** 

If you have a choice between JNDI and Native(JDBC), Use JNDI for production
, and Native(JDBC) for testing/debugging. 
