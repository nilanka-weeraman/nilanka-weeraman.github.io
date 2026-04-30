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

**Scenario : I have a bunch of parquet files which were created by other engines ( Spark, Athena etc.. ), now I want to convert them to Iceberg..**

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

**Scenario : Now I have migrated existing data to Iceberg tables. Now I want to add some columns and remove unwanted columns**

This is one of the Iceberg’s key features addressing previous limitations of Hive. Iceberg supports **schema evolution.** You can add , remove, rename columns as if it's done in a traditional database using alter table (add \| drop \| rename ) column statement. This is a *metadata only operation*, which means that no changes to data files. Iceberg has some robust ways to handle this which you can explore during the course. Refer this [notebook for actual implementations.](https://github.com/Snowflake-Labs/apache-iceberg-from-zero/blob/main/notebooks/E2.3%20-%20SchemaAndPartitionEvolution.ipynb)


**Scenario : My first table is ready to use, but query results are slow. I might need to add partitions**

This is a typical optimization option in data lakes. If your queries take time or does full table scans, it is time to partition. Iceberg can easily add partitions by alter table statement. For example adding a daily partition for a timestamp column 
```sql
spark.sql("""
    ALTER TABLE your_table_name
    ADD PARTITION FIELD days(your_timestamp_column)
""")
```

**Scenario : Okay I've added a partition. Should I change my queries to use partition columns ?**

No you don't. This feature is called **hidden partitions**. As long as you use the original column in your query predicate, the partition will be used. ie. you can still use timestamp column which used to create daily partition above, but iceberg will only use relevant data files for the query. <br>

This is a great feature, as the data engineers could silently add partitions based on query patterns and there are no downstream or upstream ETL changes required ! The users will feel improved performance.