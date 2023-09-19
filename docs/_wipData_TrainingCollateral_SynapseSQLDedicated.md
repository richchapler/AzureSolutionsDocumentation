## Synapse SQL Features
* **T-SQL language support**: Synapse SQL allows you to query and analyze your data using the T-SQL language, which is an ANSI-compliant dialect of SQL used in SQL Server and Azure SQL Database.
* **Serverless consumption model**: Synapse SQL provides a serverless consumption model that allows you to query external tables stored in Azure Data Lake storage or Dataverse.
* **Materialized views**: Synapse SQL supports materialized views, which can improve query performance by precomputing and storing the results of complex queries.
* **Caching queries**: Synapse SQL offers multiple forms of query caching, including SSD-based caching, in-memory caching, and result set caching.
* **Support for external tables**: Synapse SQL allows you to create external tables that reference data stored in Azure Data Lake storage or Dataverse.

## Create Table (including Primary Key)
### ...using a T-SQL query
```
CREATE TABLE TableName (
Column1 INT PRIMARY KEY,
Column2 VARCHAR(50),
...
);
```

### ...using an ARM template
```
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json",
  "contentVersion": "1.0.0.0",
  "metadata": {
    "_generator": {
      "name": "bicep",
      "version": "0.6.18.56646",
      "templateHash": "8855658719083429210"
    }
  },
  "parameters": {
    "sqlServerName": {
      "type": "string",
      "defaultValue": "[format('sql{0}', uniqueString(resourceGroup().id))]",
      "metadata": {
        "description": "The SQL Logical Server name."
      }
    },
    "sqlAdministratorLogin": {
      "type": "string",
      "metadata": {
        "description": "The administrator username of the SQL Server."
      }
    },
    "sqlAdministratorPassword": {
      "type": "secureString",
      "metadata": {
        "description": "The administrator password of the SQL Server."
      }
    },
    "databasesName": {
      "type": "string",
      "metadata": {
        "description": "The name of the Database."
      }
    },
    "capacity": {
      "type": "int",
      "maxValue": 54000,
      "minValue": 900,
      "metadata": {
        "description": "DW Performance Level expressed in DTU (i.e. 900 DTU = DW100c)"
      }
    },
    ...
  },
  ...
}
```

### General Considerations / Recommendations
* **Table Category**: Determine whether the table belongs to the fact, dimension, or integration category. Fact tables contain quantitative data generated in a transactional system, while dimension tables store attribute data that changes infrequently. Integration tables provide a place for integrating or staging data.
* **Schema Names**: Schemas are useful for grouping objects used in a similar fashion1. Consider defining user-defined schemas to organize your tables.
* **Table Names**: Migrate all fact, dimension, and integration tables to one SQL pool schema1. When constructing table names, consider using descriptive names that reflect the table’s purpose and content.
* **Table Persistence**: Regular tables are used for permanent storage of data1. Temporary tables are used for temporary storage during query execution1. External tables reference data stored outside of Synapse SQL.
* **Data Types**: Choose appropriate data types for columns based on the nature of the data they will store.
Distributed Tables: Distributed tables are used to distribute data across compute nodes in a dedicated SQL pool1. Hash-distributed tables distribute rows based on a hash function applied to a distribution column1. Replicated tables duplicate rows across all compute nodes1. Round-robin tables distribute rows evenly across compute nodes.
* **Common Distribution Methods for Tables**: Consider using hash-distributed tables for large fact tables and replicated or round-robin tables for smaller dimension tables.
* **Partitions**: Partitions divide a table into smaller, more manageable pieces based on a partitioning column.
* **Columnstore Indexes**: Columnstore indexes improve query performance by storing columnar data in memory.
Statistics: Statistics help the query optimizer generate efficient query plans by providing information about the distribution of data in columns.
* **Primary Key and Unique Key**: Define primary key and unique key constraints to enforce data integrity and uniqueness
Commands for Creating Tables: Familiarize yourself with commands for creating different types of tables, such as regular, temporary, and external tables.
* **Automatic table optimization** is a feature that helps improve the performance of your queries. It automatically creates and updates query-optimization statistics on tables in the dedicated SQL pool. By collecting statistics on your data, the dedicated SQL pool query optimizer can make more informed decisions about how to execute queries efficiently. The more the dedicated SQL pool knows about your data, the faster it can execute queries against it.
* **Automatic statistics collection** is another feature that works in conjunction with automatic table optimization. When the database AUTO_CREATE_STATISTICS option is enabled, the query optimizer analyzes incoming user queries for missing statistics. If statistics are missing, it creates statistics on individual columns in the query predicate or join condition to improve cardinality estimates for the query plan.
<br>[Table statistics for dedicated SQL pool in Azure Synapse Analytics](https://learn.microsoft.com/en-us/azure/synapse-analytics/sql-data-warehouse/sql-data-warehouse-tables-statistics) 

## Create Index
### T-SQL
```
CREATE CLUSTERED COLUMNSTORE INDEX IX_MyTable ON dbo.MyTable;
```

### ARM template
```
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "databaseName": {
            "type": "string",
            "metadata": {
                "description": "The name of the Synapse SQL database."
            }
        },
        "tableName": {
            "type": "string",
            "metadata": {
                "description": "The name of the table to add the index to."
            }
        },
        "indexName": {
            "type": "string",
            "metadata": {
                "description": "The name of the index to be created."
            }
        },
        "indexColumns": {
            "type": "array",
            "metadata": {
                "description": "The columns to include in the index."
            }
        }
    },
    "resources": [
        {
            "type": "Microsoft.Synapse/workspaces/sqlDatabases/tables/indexes",
            "apiVersion": "2020-12-01-preview",
            "name": "[concat(parameters('databaseName'), '/', parameters('tableName'), '/', parameters('indexName'))]",
            "properties": {
                "columns": "[parameters('indexColumns')]"
            }
        }
    ]
}
```

### Index Types
* **Clustered columnstore indexes**
  * Benefits: **Clustered columnstore tables offer the highest level of data compression and the best overall query performance. They are usually the best choice for large tables**.
  * Drawbacks: Columnstore tables do not support varchar (max), nvarchar (max), and varbinary (max) data types. They may also be less efficient for transient data and small tables with less than 60 million rows.
* **Heap tables**
  * Benefits: Loading data to heap tables is faster than loading it to clustered columnstore tables, especially when you’re temporarily landing data in Synapse SQL. Loading data to a temporary table is even faster than loading it to permanent storage.
  * Drawbacks: Clustered columnstore tables begin to achieve optimal compression once there are more than 60 million rows. For small lookup tables with less than 60 million rows, consider using heap or clustered index for faster query performance.
* **Clustered and nonclustered indexes**
  * Benefits: Clustered indexes may outperform clustered columnstore tables when a single row needs to be quickly retrieved. Nonclustered indexes can improve filter on other columns.
  * Drawbacks: Each index added to a table increases both space and processing time during loads.

_By default, Synapse SQL creates a clustered columnstore index when no index options are specified on a table_

[Indexes on dedicated SQL pool tables in Azure Synapse Analytics](https://learn.microsoft.com/en-us/azure/synapse-analytics/sql-data-warehouse/sql-data-warehouse-tables-index)

## Create Views
Views are virtual tables that are based on the result of an SQL query.

```
CREATE VIEW ViewName AS
SELECT Column1, Column2
FROM TableName
WHERE Condition;
```

## Create Stored Procedure
To create a stored procedure, you can use the CREATE PROCEDURE statement. Here’s an example:

```
CREATE PROCEDURE ProcedureName
AS
BEGIN
SQL statements here
END;
```

Queries in GUI vs. Code(script) interface: You can execute queries both in the GUI and through code/scripting interfaces. The choice depends on your preference and requirements.
Joins (special considerations?): When working with joins, it’s important to consider the join type (e.g., inner join, left join) and the join condition. Here’s an example of an inner join:

```
SELECT *
FROM Table1
INNER JOIN Table2 ON Table1.Column = Table2.Column;
```

Case Statement Equivalent in Gui? Scripting Only?: The case statement is available in both GUI and scripting interfaces. Here’s an example of using a case statement in T-SQL scripting:

```
SELECT Column1,
CASE
WHEN Condition1 THEN Result1
WHEN Condition2 THEN Result2
ELSE Result3
END AS NewColumn
FROM TableName;
```
