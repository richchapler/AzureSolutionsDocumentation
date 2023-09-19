## Synapse SQL Features
* **T-SQL language support**: Synapse SQL allows you to query and analyze your data using the T-SQL language, which is an ANSI-compliant dialect of SQL used in SQL Server and Azure SQL Database.
* **Serverless consumption model**: Synapse SQL provides a serverless consumption model that allows you to query external tables stored in Azure Data Lake storage or Dataverse.
* **Materialized views**: Synapse SQL supports materialized views, which can improve query performance by precomputing and storing the results of complex queries.
* **Caching queries**: Synapse SQL offers multiple forms of query caching, including SSD-based caching, in-memory caching, and result set caching.
* **Support for external tables**: Synapse SQL allows you to create external tables that reference data stored in Azure Data Lake storage or Dataverse.

## Create Table
### ...using a T-SQL query
```
CREATE TABLE TableName (
Column1 INT,
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

### Considerations
* The process of creating tables is similar in both Synapse SQL and Azure SQL
* However, Synapse SQL provides additional features such as automatic table optimization and automatic statistics collection:
  * **Automatic table optimization** is a feature that helps improve the performance of your queries. It automatically creates and updates query-optimization statistics on tables in the dedicated SQL pool. By collecting statistics on your data, the dedicated SQL pool query optimizer can make more informed decisions about how to execute queries efficiently. The more the dedicated SQL pool knows about your data, the faster it can execute queries against it.
  * **Automatic statistics collection** is another feature that works in conjunction with automatic table optimization. When the database AUTO_CREATE_STATISTICS option is enabled, the query optimizer analyzes incoming user queries for missing statistics. If statistics are missing, it creates statistics on individual columns in the query predicate or join condition to improve cardinality estimates for the query plan.
<br>[Table statistics for dedicated SQL pool in Azure Synapse Analytics](https://learn.microsoft.com/en-us/azure/synapse-analytics/sql-data-warehouse/sql-data-warehouse-tables-statistics) 

* When creating tables, it’s important to consider naming conventions and data types. You can create tables using both the GUI and scripting. Here’s an example of creating a table using T-SQL scripting:

## Creating Primary Key, Indexes, etc.
To create a primary key, you can use the PRIMARY KEY constraint in your CREATE TABLE statement. Here’s an example:

```
CREATE TABLE TableName (
Column1 INT PRIMARY KEY,
Column2 VARCHAR(50),
...
);
```

To create indexes, you can use the CREATE INDEX statement. Here’s an example:
CREATE INDEX IndexName ON TableName (Column1, Column2);

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
