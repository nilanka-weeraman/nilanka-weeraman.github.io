---
layout: post
title: "Iceberg Tables for data lakes "
date: 2026-04-30 10:00:00 +0930
tags: [dataengineering, datalakes, iceberg, tutorial, review]
---

*The reason I’m writeup is to revisit the concepts I’ve learnt during an amazing new course offered by Coursera & Snowflake called [apache iceberg from zero](https://www.coursera.org/learn/apache-iceberg-data-lakehouse). This is one of the best comprehensive courses I’ve taken. It comes with a fully deployable docker setup and a clear explanation by [Russel Spitzer.](https://www.snowflake.com/en/blog/authors/russell-spitzer/) The content is dense and covers valuable practical aspects of a modern data lake design and maintenance. Further, after going through, one can understand the probable inner workings of data clouds like Snowflake.*

### Iceberg open table formats & Data Lakes

Apache Iceberg is an open-table format built for open data lakes. Data lakes have been here for sometime, so what is special about Iceberg ? or rather why do we need Icebergs? <br>

### Shortest history

If you are familiar with big data frameworks & stack ; you may be knowing how the storage, the metadata & the engines were combined into providing a data analytics platform. The original idea was inspired by Google’g big table & map reduce whitepapers ( Jeff Dean et al )  and converted into an open source project by Doug Cutting as Apache Hadoop. The ecosystem exploded thereafter. <br>

The initial stack loosely looks like below for a Hive based stack.<br>

| Engines  | Hive, Spark |
| :---- | :---- |
| Metadata  | Hive Metastore |
| File Types | Parquet, ORC, AVRO…  |
|  File System | Hadoop Distributed File System ( HDFS ) |
| Storage type | disk file systems , Object stores (S3) |

The stack is highly scalable. However that came with compromising on ACID style databases. In simple terms, if it require to update records, update schemas it was required to re-write in most cases.  
Some other projects tacked these problems such as Cassandra, Hbase. <br>

The stack served really well for most use-cases. Netflix who faced scaling issues with Hive ( developed by Facebook/Meta ), developed their own project called Iceberg. They donated this project to Apache foundation. The initial repo addresses their original thinking behind the project and can be found [here](https://github.com/Netflix/iceberg). <br>

With Icberg changes, the stack has evolved into something like below <br>

| Engines  | Hive, Spark |
| :---- | :---- |
| Metadata  | Hive Metastore |
| Table Format (New ) | A new metadata specification for schema & file metadata introduced |
| File Types | Parquet, ORC, AVRO…  |
|  File System ( Obsolete ) | HDFS \- no longer required  |
| Storage type | disk file systems , Object stores (S3) |

The design goals or problems addressed by Iceberg are ; 

Schema evolution, Partition evolution, Versioning & time travel, Continuous Deployment, Metrics for planning, Invisible partitioning, Concurrent writes

Now let's take a look at these concepts while following a practical data engineering workflow.

[How to create my first Iceberg table or migrate to Iceberg](#createmigrate)<br>
[Altering tables (Schema evolution) ](#schemaevolution)<br>
[Adding partitions](#partioning)<br>
[Hidden partitions](#hiddenpartitions)<br>
[Optimizing partitions (partition evolution)](#partitionevolution)<br>
[Recover old data or table versions ( time travel )](#timetravel)<br>
[Continuous table modification with Write Audit Publish ](#wap)<br>
[Continuous experimentating with Branching and merging](#branching)<br>

### Scenario : I have a bunch of parquet files which were created by other engines ( Spark, Athena etc.. ), now I want to convert them to Iceberg {#createmigrate}

Iceberg natively support parquet, avro, orc file types. If you have Polaris as a catalog , this migration can be done in several ways. Assume you have Polaris as the catalog service & Spark as the analytic engine.

| File type | CTAS ( create table as ) | Metadata only procedures |
| :---- | :---- | :---- |
| csv | Only option available if you have plain text files or zip files.  New data files will be created along with new metadata. Possible flow :  read csv to spark dataframe \-\> write to iceberg table  | Not supported |
| Parquet, avro, orc | If you need transformation use this option Possible flow :  Read parquet to spark dataframe \-\> write to iceberg table | If you don’t need any transformations use this option. Use metadata only operation in Polaris  Possible flow :  Create a temp table with spark \-\> use snapshot procedure to create iceberg table  |

Adding data vs inserting

| File type | Insert into | Metadata only operation |
| :---- | :---- | :---- |
| csv  | New data files will be created along with new metadata. Possible flow :  read csv to spark dataframe \-\> insert to iceberg table  | Not available |
| Parquet,avro etc | New data files created. Use only if you have transformations | Polaris add files method supports adding parquet files which are already located to the table's file location. Eg. spark**.**sql("""     CALL polaris.system.add\_files(         table \=\> 'your table name',         source\_table \=\> 'parquet.\`your parquet file location\`'     ) """) |

### Scenario : Now I have migrated existing data to Iceberg tables. Now I want to add some columns and remove unwanted columns {#schemaevolution}

This is one of the Iceberg’s key features addressing previous limitations of Hive. Iceberg supports **schema evolution.** You can add , remove, rename columns as if it's done in a traditional database using alter table (add \| drop \| rename ) column statement. This is a *metadata only operation*, which means that no changes to data files. Iceberg has some robust ways to handle this which you can explore during the course. Refer this [notebook for actual implementations.](https://github.com/Snowflake-Labs/apache-iceberg-from-zero/blob/main/notebooks/E2.3%20-%20SchemaAndPartitionEvolution.ipynb)


### Scenario : My first table is ready to use, but query results are slow. I might need to add partition  {#partioning}

This is a typical optimization option in data lakes. If your queries take time or does full table scans, it is time to partition. Iceberg can easily add partitions by alter table statement. For example adding a daily partition for a timestamp column 

<div style="max-width: 500px;">
```sql
spark.sql("""
    ALTER TABLE your_table_name
    ADD PARTITION FIELD days(your_timestamp_column)
""")
```
</div>

### Scenario : Okay I've added a partition. Should I change my queries to use partition columns  {#hiddenpartitions}

No you don't. This feature is called **hidden partitions**. As long as you use the original column in your query predicate, the partition will be used. ie. you can still use timestamp column which used to create daily partition above, but iceberg will only use relevant data files for the query. <br>

This is a great feature, as data engineers can silently add partitions based on query patterns and there are no downstream or upstream ETL changes required ! The users will feel improved performance.

### Scenario : So far so good. But I might want my partition daily partition to changed to monthly. Should I recreate the table? {#partitionevolution}

No you don't have to recreate. Iceberg's *partition evolution* handles this. Iceberg table can have a new partition for the same column. You can remove old partition and add a new monthly partition. The catch here is ; only the newly inserted data will be partitioned monthly while old partition will be a one big default partition. And if you change your mind again to rever to daily partition, that's fine Iceberg will use older daily partition, consider monthly partition as a different partition and newly inserted data will be daily partitioned into files. 

### Scenario : Oops, I accidently deleted an entire monthly partition. Is there a way to recover?   {#timetravel}

Yes, becauase every change to a table is stored as a _commit_ in Iceberg with metadata ( _snapshots_), you can reinstate the pre-delete records. This is called **time travel** feature. The feature is entirely possible due to isolation of metadata (snapshots) with actual data files. You can fire your SQL with the specific table version as ; 
```sql
SELECT your_columns FROM your_table VERSION AS OF your_table's_snapshot_id
```

How do you find the snapshot ID ? By querying the metadata table as ;
```sql
SELECT snapshot_id, committed_at, operation FROM yout_table_name.snapshots ORDER BY committed_at
```
Refer [this notebook for examples](https://github.com/Snowflake-Labs/apache-iceberg-from-zero/blob/main/notebooks/E2.2%20-%20BranchingAndTagging.ipynb)

If you are coming from a database background, this is a familiar operation whereby reinstating using _redo logs_ in Oracle. So again it is evident that lot of database features are now made available for data lake tables with Iceburg.

### Scenario : That's a big hedeache gone. I want to change a column to match a data source change. I want the change to be tested before commiting to final table. {#wap}

This is typical change request for a data engineer. We could create a temporary table with new logic and provide access for testing or use Iceberg's *Write Audit Publish (WAP)* elegantly.
The operation is similar to rollback, commit procedure in traditional databases, but now available in data lake tables ! The process is similar ; <br>
Stage the write ( spark.conf.set("spark.wap.id", "a_friendly_name") ) -> do the change -> end stage ( spark.conf.unset("spark.wap.id") ) <br>

The change is not visible in the main table. But you can ask a user to validate the change by referring the correct snapshot as ; 

```sql
SELECT columns FROM your_table_name VERSION AS OF snapshot_id_of_the_change
```

If the user is satisfied with the change, you can **publish** the change by using polaris's system.publish_change procedure and then cleaning up the audit stage by using system.expire_snapshot procedure. <br>

These system procedures are metadata only procedures and does not affect any datafile changes. We will see how to cleanup datafiles later.

### Scenario : Good. I want to write a new ETL and compare read performance and switch to new ETL if performance is better. Is there a better way to do this? {#branching}

Yes, this sounds like a typical software feature branch and Iceberg supports branching. **No data is duplilcated** for the new branch, instead only incremental changes on this branch are written to different data filesLets see how its done. This is the same feature Snowflake offers as _cloning_. Now you'd understand probable inner working how this is implemented. Lets see how its done in Spark & Polaris<br>

- First define a branch with 'Alter table create branch' statement.
- You can do bunch of processing for this branch of the table, referring the table as 'table_name.BRANCH_branchname'
- Then merge into the main branch with system.fast_forward procedure.

_if the main branch change before merge, we have to rebase our branch_

### Scenario : Does Iceberg has any limitations ? Can it ingest streaming data ? {#ingestions}

Lets see some issues that arise with all the nice features offered above. As you can guess, Iceberg use its metadata for offering features such as _timetravel_, _ACID type operations_, _version control_. When data is modified each create table **snapshots**. Over time these snapshots will grow. As metadata grows the processing engine incur .