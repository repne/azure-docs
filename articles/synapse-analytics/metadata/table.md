---
title: Shared metadata tables
description: Azure Synapse Analytics provides a shared metadata model where creating a table in serverless Apache Spark pool will make it accessible from serverless SQL pool and dedicated SQL pool without duplicating the data. 
services: sql-data-warehouse 
author: MikeRys 
ms.service:  synapse-analytics 
ms.topic: overview 
ms.subservice: metadata
ms.date: 05/01/2020 
ms.author: mrys 
ms.reviewer: jrasnick
ms.custom: devx-track-csharp
---

# Azure Synapse Analytics shared metadata tables


Azure Synapse Analytics allows the different workspace computational engines to share databases and Parquet-backed tables between its Apache Spark pools and serverless SQL pool.

Once a database has been created by a Spark job, you can create tables in it with Spark that use Parquet as the storage format. Table names will be converted to lower case and need to be queried using the lower case name. These tables will immediately become available for querying by any of the Azure Synapse workspace Spark pools. They can also be used from any of the Spark jobs subject to permissions.

The Spark created, managed, and external tables are also made available as external tables with the same name in the corresponding synchronized database in serverless SQL pool. [Exposing a Spark table in SQL](#expose-a-spark-table-in-sql) provides more detail on the table synchronization.

Since the tables are synchronized to serverless SQL pool asynchronously, there will be a delay until they appear.

## Manage a Spark created table

Use Spark to manage Spark created databases. For example, delete it through a serverless Apache Spark pool job, and create tables in it from Spark.

If you create objects in such a database from serverless SQL pool or try to drop the database, the operation will fail. The original Spark database cannot be changed via serverless SQL pool.

## Expose a Spark table in SQL

### Shared Spark tables

Spark provides two types of tables that Azure Synapse exposes in SQL automatically:

- Managed tables

  Spark provides many options for how to store data in managed tables, such as TEXT, CSV, JSON, JDBC, PARQUET, ORC, HIVE, DELTA, and LIBSVM. These files are normally stored in the `warehouse` directory where managed table data is stored.

- External tables

  Spark also provides ways to create external tables over existing data, either by providing the `LOCATION` option or using the Hive format. Such external tables can be over a variety of data formats, including Parquet.

Azure Synapse currently only shares managed and external Spark tables that store their data in Parquet or CSV format with the SQL engines. Tables backed by other formats are not automatically synced. You may be able to sync such tables explicitly yourself as an external table in your own SQL database if the SQL engine supports the table's underlying format.

### Share Spark tables

The shareable managed and external Spark tables exposed in the SQL engine as external tables with the following properties:

- The SQL external table's data source is the data source representing the Spark table's location folder.
- The SQL external table's file format is Parquet.
- The SQL external table's access credential is pass-through.

Since all Spark table names are valid SQL table names and all Spark column names are valid SQL column names, the Spark table and column names will be used for the SQL external table.

Spark tables provide different data types than the Synapse SQL engines. The following table maps Spark table data types map to the SQL types:

| Spark data type | SQL data type | Comments |
|---|---|---|
| `byte`      | `smallint`       ||
| `short`     | `smallint`       ||
| `integer`   |    `int`            ||
| `long`      |    `bigint`         ||
| `float`     | `real`           |<!-- need precision and scale-->|
| `double`    | `float`          |<!-- need precision and scale-->|
| `decimal`      | `decimal`        |<!-- need precision and scale-->|
| `timestamp` |    `datetime2`      |<!-- need precision and scale-->|
| `date`      | `date`           ||
| `string`    |    `varchar(max)`   | With collation `Latin1_General_100_BIN2_UTF8` |
| `binary`    |    `varbinary(max)` ||
| `boolean`   |    `bit`            ||
| `array`     |    `varchar(max)`   | Serializes into JSON with collation `Latin1_General_100_BIN2_UTF8` |
| `map`       |    `varchar(max)`   | Serializes into JSON with collation `Latin1_General_100_BIN2_UTF8` |
| `struct`    |    `varchar(max)`   | Serializes into JSON with collation `Latin1_General_100_BIN2_UTF8` |

## Security model

The Spark databases and tables, as well as their synchronized representations in the SQL engine will be secured at the underlying storage level. Since they do not currently have permissions on the objects themselves, the objects can be seen in the object explorer.

The security principal who creates a managed table is considered the owner of that table and has all the rights to the table as well as the underlying folders and files. In addition, the owner of the database will automatically become co-owner of the table.

If you create a Spark or SQL external table with authentication pass-through, the data is only secured at the folder and file levels. If someone queries this type of external table, the security identity of the query submitter is passed down to the file system, which will check for access rights.

For more information on how to set permissions on the folders and files, see [Azure Synapse Analytics shared database](database.md).

## Examples

### Create a managed table backed by Parquet in Spark and query from serverless SQL pool

In this scenario, you have a Spark database named `mytestdb`. See [Create and connect to a Spark database with serverless SQL pool](database.md#create-and-connect-to-spark-database-with-serverless-sql-pool).

Create a managed Spark table with SparkSQL by running the following command:

```sql
    CREATE TABLE mytestdb.myparquettable(id int, name string, birthdate date) USING Parquet
```

This command creates the table `myparquettable` in the database `mytestdb`. Table names will be converted to lowercase. After a short delay, you can see the table in your serverless SQL pool. For example, run the following statement from your serverless SQL pool.

```sql
    USE mytestdb;
    SELECT * FROM sys.tables;
```

Verify that `myparquettable` is included in the results.

>[!NOTE]
>A table that is not using Parquet as its storage format will not be synchronized.

Next, insert some values into the table from Spark, for example with the following C# Spark statements in a C# notebook:

```csharp
using Microsoft.Spark.Sql.Types;

var data = new List<GenericRow>();

data.Add(new GenericRow(new object[] { 1, "Alice", new Date(2010, 1, 1)}));
data.Add(new GenericRow(new object[] { 2, "Bob", new Date(1990, 1, 1)}));

var schema = new StructType
    (new List<StructField>()
        {
            new StructField("id", new IntegerType()),
            new StructField("name", new StringType()),
            new StructField("birthdate", new DateType())
        }
    );

var df = spark.CreateDataFrame(data, schema);
df.Write().Mode(SaveMode.Append).InsertInto("mytestdb.myparquettable");
```

Now you can read the data from your serverless SQL pool as follows:

```sql
SELECT * FROM mytestdb.dbo.myparquettable WHERE name = 'Alice';
```

You should get the following row as result:

```
id | name | birthdate
---+-------+-----------
1 | Alice | 2010-01-01
```

### Create an external table backed by Parquet in Spark and query from serverless SQL pool

In this example, create an external Spark table over the Parquet data files that got created in the previous example for the managed table.

For example, with SparkSQL run:

```sql
CREATE TABLE mytestdb.myexternalparquettable
    USING Parquet
    LOCATION "abfss://<fs>@arcadialake.dfs.core.windows.net/synapse/workspaces/<synapse_ws>/warehouse/mytestdb.db/myparquettable/"
```

Replace the placeholder `<fs>` with the file system name that is the workspace default file system and the placeholder `<synapse_ws>` with the name of the synapse workspace you're using to run this example.

The previous example creates the table `myexternalparquettable` in the database `mytestdb`. After a short delay, you can see the table in your serverless SQL pool. For example, run the following statement from your serverless SQL pool.

```sql
USE mytestdb;
SELECT * FROM sys.tables;
```

Verify that `myexternalparquettable` is included in the results.

Now you can read the data from your serverless SQL pool as follows:

```sql
SELECT * FROM mytestdb.dbo.myexternalparquettable WHERE name = 'Alice';
```

You should get the following row as result:

```
id | name | birthdate
---+-------+-----------
1 | Alice | 2010-01-01
```

## Next steps

- [Learn more about Azure Synapse Analytics' shared metadata](overview.md)
- [Learn more about Azure Synapse Analytics' shared metadata database](database.md)
