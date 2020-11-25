# spark-hive-hbase
Spark Hive HBase Integration

## Creating HBase table and Inserting data

### Login to HBase Shell
``` shell
hbase shell
```

### Creating HBase table
``` shell
hbase(main):002:0> create 'hbase_emp_table', [{NAME => 'per', COMPRESSION => 'SNAPPY'}, {NAME => 'prof', COMPRESSION => 'SNAPPY'} ]
Created table hbase_emp_table
Took 1.5483 seconds
=> Hbase::Table - hbase_emp_table

hbase(main):003:0> describe 'hbase_emp_table'
Table hbase_emp_table is ENABLED
hbase_emp_table
COLUMN FAMILIES DESCRIPTION
{NAME => 'per', VERSIONS => '1', EVICT_BLOCKS_ON_CLOSE => 'false', NEW_VERSION_BEHAVIOR => 'false', KEEP_DELETED_CELLS => 'FALSE', CACHE_DATA_ON_WRITE => 'false', DATA_BLOCK_ENCODING =>
 'NONE', TTL => 'FOREVER', MIN_VERSIONS => '0', REPLICATION_SCOPE => '0', BLOOMFILTER => 'ROW', CACHE_INDEX_ON_WRITE => 'false', IN_MEMORY => 'false', CACHE_BLOOMS_ON_WRITE => 'false',
PREFETCH_BLOCKS_ON_OPEN => 'false', COMPRESSION => 'SNAPPY', BLOCKCACHE => 'true', BLOCKSIZE => '65536'}
{NAME => 'prof', VERSIONS => '1', EVICT_BLOCKS_ON_CLOSE => 'false', NEW_VERSION_BEHAVIOR => 'false', KEEP_DELETED_CELLS => 'FALSE', CACHE_DATA_ON_WRITE => 'false', DATA_BLOCK_ENCODING =
> 'NONE', TTL => 'FOREVER', MIN_VERSIONS => '0', REPLICATION_SCOPE => '0', BLOOMFILTER => 'ROW', CACHE_INDEX_ON_WRITE => 'false', IN_MEMORY => 'false', CACHE_BLOOMS_ON_WRITE => 'false',
 PREFETCH_BLOCKS_ON_OPEN => 'false', COMPRESSION => 'SNAPPY', BLOCKCACHE => 'true', BLOCKSIZE => '65536'}
2 row(s)
Took 0.2091 seconds
```

### Inserting data to HBase table
```shell
put 'hbase_emp_table','1','per:name','Ranga Reddy'
put 'hbase_emp_table','1','per:age','32'
put 'hbase_emp_table','1','prof:des','Senior Software Engineer'
put 'hbase_emp_table','1','prof:sal','50000'

put 'hbase_emp_table','2','per:name','Nishanth Reddy'
put 'hbase_emp_table','2','per:age','3'
put 'hbase_emp_table','2','prof:des','Software Engineer'
put 'hbase_emp_table','2','prof:sal','80000'
```

### Checking the HBase table data
```shell
hbase(main):015:0> scan 'hbase_emp_table'
ROW                                             COLUMN+CELL
 1                                              column=per:age, timestamp=1606281940257, value=32
 1                                              column=per:name, timestamp=1606281940212, value=Ranga Reddy
 1                                              column=prof:des, timestamp=1606281940292, value=Senior Software Engineer
 1                                              column=prof:sal, timestamp=1606281940327, value=50000
 2                                              column=per:age, timestamp=1606281940409, value=3
 2                                              column=per:name, timestamp=1606281940381, value=Nishanth Reddy
 2                                              column=prof:des, timestamp=1606281940443, value=Software Engineer
 2                                              column=prof:sal, timestamp=1606281940516, value=80000
2 row(s)
Took 0.0366 seconds
```

## Creating a Hive External Table for HBase and checking data
### Create a Hive table
Apache provides a storage handler and a SerDe that enable Hive to read the HBase table format. **HBaseStorageHandler** allows Hive DDL for managing table definitions in both **Hive metastore** and **HBase’s catalog** simultaneously and consistently. By default, if the table name of hbase is not specified, it is the same as the table name of hive.

**Syntax:**
```sql
CREATE EXTERNAL TABLE hive_table_name colname coltype[, colname coltype,...] 
ROW FORMAT SERDE 'org.apache.hadoop.hive.hbase.HBaseSerDe'
STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler' 
WITH SERDEPROPERTIES ('hbase.columns.mapping'=':key,value:key)
TBLPROPERTIES("hbase.table.name" = "hbase_table_name")
```

**Example:**
```sql
hive> CREATE EXTERNAL TABLE IF NOT EXISTS hive_emp_table(id INT, name STRING, age SMALLINT, designation STRING, salary BIGINT) 
STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler' 
WITH SERDEPROPERTIES ("hbase.columns.mapping" = ":key,per:name,per:age,prof:des,prof:sal") 
TBLPROPERTIES("hbase.table.name" = "hbase_emp_table");
```

The values provided in the **hbase.columns.mapping** property correspond one-for-one with column names of the hive table. HBase column names are **fully qualified by column family** and we will use the special token **:key** to represent the **Rowkey**. The above example makes rows from the HBase table **Employee** available via the Hive table **employee**. The employee column rowkey maps to the HBase’s table’s rowkey, name to name in the E column family, designation to designation in the E column family and salary to salary in the E family.

### Select the Hive table data
```sql
hive> select * from hive_emp_table;
OK
1	Ranga Reddy	Software Engineer	50000
2	Nishanth Reddy	Software Engineer	80000
Time taken: 0.532 seconds, Fetched: 2 row(s)
```
## Launch Spark-Shell and check the table data

### CDH/CDP:

Step1: Find the **hive-hbase-handler** jar
```shell
ls -l /opt/cloudera/parcels/CDH/jars | grep hive-hbase
-rw-r--r-- 1 root root    119538 May 20  2020 hive-hbase-handler-3.1.3000.7.1.1.0-565.jar
```
Step2: Copy the **hive-hbase-handler** jar to spark jars folder.
```shell
ln -s /opt/cloudera/parcels/CDH/jars/hive-hbase-handler-3.1.3000.7.1.1.0-565.jar /opt/cloudera/parcels/CDH/lib/spark/jars/
```
Step3: Copy the **hbase-site.xml** file to **/etc/spark/conf/**
```shell
cp /opt/cloudera/parcels/CDH/lib/hbase/conf/hbase-site.xml /etc/spark/conf/
```
Step4: Launch the **spark-shell** by adding hbase jars and hive-hbase-handler jar.
```shell
sudo -u hive spark-shell --master yarn --jars /opt/cloudera/parcels/CDH/jars/hive-hbase-handler-*.jar, /opt/cloudera/parcels/CDH/lib/hbase/hbase-client-*.jar, /opt/cloudera/parcels/CDH/lib/hbase/hbase-common-*.jar, /opt/cloudera/parcels/CDH/lib/hbase/hbase-server-*.jar, /opt/cloudera/parcels/CDH/lib/hbase/hbase-hadoop2-compat-*.jar, /opt/cloudera/parcels/CDH/lib/hbase/hbase-protocol-*.jar,/opt/cloudera/parcels/CDH/jars/guava-28.*-jre.jar,/opt/cloudera/parcels/CDH/jars/htrace-core-3.2.0-incubating.jar --files /etc/spark/conf/hbase-site.xml
```

### HDP:

Step1: Find the **hive-hbase-handler** jar
```shell
ls /usr/hdp/current/hive-client/lib/ | grep hive-hbase
hive-hbase-handler-3.1.0.3.1.5.0-152.jar
```
Step2: Copy the **hive-hbase-handler** jar to spark jars folder.
```shell
ln -s /usr/hdp/current/hive-client/lib/hive-hbase-handler-3.1.0.3.1.5.0-152.jar /usr/hdp/current/spark2-client/jars/
```
Step3: Copy the **hbase-site.xml** file to **/usr/hdp/current/spark2-client/conf/**
```shell
cp /usr/hdp/current/hbase-client/conf/hbase-site.xml /usr/hdp/current/spark2-client/conf/
```
Step4: Launch the **spark-shell** by adding hbase jars and hive-hbase-handler jar.
```shell
sudo -u hive spark-shell --master local[4] --jars /usr/hdp/current/hive-client/lib/hive-hbase-handler-*.jar,/usr/hdp/current/hive-client/lib/hbase-client-*.jar,/usr/hdp/current/hive-client/lib/hbase-common-*.jar,/usr/hdp/current/hive-client/lib/hbase-server-*.jar,/usr/hdp/current/hive-client/lib/metrics-core-*.jar,/usr/hdp/current/hive-client/lib/hbase-hadoop2-compat-*.jar,/usr/hdp/current/hive-client/lib/hbase-protocol-*.jar,/usr/hdp/current/hive-client/lib/guava-28.*-jre.jar,/usr/hdp/current/hive-client/lib/protobuf-java-2.5.0.jar,/usr/hdp/current/hive-client/lib/htrace-core-*-incubating.jar --files /usr/hdp/current/spark2-client/conf/hbase-site.xml
```
## Checking the hive table data in spark-shell
```shell
scala> spark.catalog.listTables.show(truncate=false)
scala> val empDF = spark.sql("select * from hive_emp_table")
scala> empDF.printSchema()
root
 |-- id: integer (nullable = true)
 |-- name: string (nullable = true)
 |-- age: short (nullable = true)
 |-- designation: string (nullable = true)
 |-- salary: long (nullable = true)

scala> empDF.show(truncate=false)
+---+--------------+---+------------------------+------+
|id |name          |age|designation             |salary|
+---+--------------+---+------------------------+------+
|1  |Ranga Reddy   |32 |Senior Software Engineer|50000 |
|2  |Nishanth Reddy|3  |Software Engineer       |80000 |
+---+--------------+---+------------------------+------+
```

## References
* https://cwiki.apache.org/confluence/display/Hive/HBaseIntegration#HBaseIntegration-StorageHandlers
