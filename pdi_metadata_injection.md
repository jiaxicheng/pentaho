## ETL Metadata Injection: Reuse Transformation templates ##

There are situations when you want to re-use the transformation to handle raw data 
with the same workflow but different field definitions, layouts etc. 
For example, to update a table, you might have raw CSV data from different vendors 
and each of them provides different column names and layout. To reuse your PDI transformation
, you can combine the `ETL Metadata Injection` step to specify the changing metadata at run time. 

Below is a simple example to use YAML files to manage the concerned information which defines 
field names and some step properties. The configurations could be adjusted without openning
the PDI program (spoon.bat or spoon.sh).

![ETL Metadata Injection](images/pentaho_etl_metadata_injection.jpg)


### YAML Configuration Files ###

In the step_config.yml file, all properties are on the same row of the PDI flow. 

In the field_config.yml file, each field definition is on its own row, `YAML Input` step 
reads the data as an array of hashes, and each row is an hash, i.e. '{Name: date, Type: Date,Format: yyyy-MM-dd}'.
The result is feed into the `JSON Input` step and converted into separate columns using JsonPath.
```
$ cat step_config.yml
---
csv_field_delimiter: ','
csv_field_enclosed: '"'
csv_escape_char: '"'
csv_cnt_header_lines: 1
csv_cnt_footer_lines: 0
csv_skip_empty_lines: Y
csv_file_format: mixed
csv_file_name: '${Internal.Entry.Current.Directory}\purchased.csv'
csv_file_mask: 
csv_include_subfolder: N
xls_output_filepath: '${Internal.Entry.Current.Directory}\file.xls'
xls_no_file_at_start: Y

$ cat field_config.yml
---
- Name: date
  Type: Date
  Format: yyyy-MM-dd
  RenameTo: entrydate
- Name: cnt_perchansed
  Type: Integer
  RenameTo: number_of_deals
  Length: 3
  Precision: 0
```

In both YAML Input steps, do the following:
+ In the 'File' tab, browse and add the YAML file defined above:

**Note:** the file-path of the YAML configuration files can be parameterized, this adds 
more flexibility when running on different vendors. No need to use fixed name on these configuration files.

+ In the 'Fields' tab, click 'Get fields', and take the default.

In the `JSON Input` step followed the `YAML Input: field definitions` step:

+ From the `File` tab, select the following and the default `Value` from the upstream step.
  + [x] Source is from a previous step

+ From the `Fields` tab, set up the fields with JSONPath:
```
   +-----------------+-------------+--------+
   | Name            | Path        | Type   |
   +-----------------+-------------+--------+
   | field_name      | $.Name      | String |
   | field_format    | $.Format    | String |
   | field_type      | $.Type      | String |
   | field_rename_to | $.RenameTo  | String |
   | field_length    | $.Length    | String |
   | field_precision | $.Precision | String |
   +-----------------+-------------+--------+

```
### ETL Metadata Injection ###
In the `ETL Metadata Injection` step

Add the template transformation in the field 'Transformations:',
all the steps and fields which can be injected in the template transformation will be listed 
under the 'Inject Metadata' tab:

Now you have all the fields defined in step_config.yml and the 3 new fields defined above
in the PDI flow, you can use these fields to fill in the metadata, see attached zip files
for all configurations.

### Template Transformation ###

The template transformation is very simple, just 3 steps and nothing to setup, all will be
specified through the ETL metadata Injection step mentioned above.

```
   +-----------------+      +---------------+      +------------------------+
   | Text file input | ---> | Select values | ---> | Microsoft Excel Output |
   +-----------------+      +---------------+      +------------------------+
```

In the real applications, the last step could be `Table output` step and the like.


Another example using `ETL Metadata Injection` is to denormalize the data (Using `Row Denormaliser` step)
and the resulting crosstab table can be used to make a heat grid map, see an example below:

![Head Grid Chart](images/pentaho_heat_grid_chart.jpg)

**Note:** check the [official document](https://help.pentaho.com/Documentation/8.0/Products/Data_Integration/Transformation_Step_Reference/ETL_Metadata_Injection/Steps_Supporting_MDI) 
to see which steps support Metadata Injection.

The example transformations (ktr) in this page can also be downloaded:[etl_metadata_injection_example.tgz](images/etl_metadata_injection_example.tgz).


