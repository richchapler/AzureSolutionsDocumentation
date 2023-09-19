## Synapse SQL Features
* **T-SQL language support**: Synapse SQL allows you to query and analyze your data using the T-SQL language, which is an ANSI-compliant dialect of SQL used in SQL Server and Azure SQL Database.
* **Serverless consumption model**: Synapse SQL provides a serverless consumption model that allows you to query external tables stored in Azure Data Lake storage or Dataverse.
* **Materialized views**: Synapse SQL supports materialized views, which can improve query performance by precomputing and storing the results of complex queries.
* **Caching queries**: Synapse SQL offers multiple forms of query caching, including SSD-based caching, in-memory caching, and result set caching.
* **Support for external tables**: Synapse SQL allows you to create external tables that reference data stored in Azure Data Lake storage or Dataverse.

## CREATE TABLE (including Primary Key)
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

## CREATE INDEX
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

## CREATE VIEW
### T-SQL
```
CREATE VIEW schema_name.view_name
AS
SELECT column1, column2, ...
FROM table_name;
```

### ARM template
```
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "workspaceName": {
      "type": "string",
      "metadata": {
        "description": "The name of the Synapse workspace."
      }
    },
    "databaseName": {
      "type": "string",
      "metadata": {
        "description": "The name of the database."
      }
    },
    "viewName": {
      "type": "string",
      "metadata": {
        "description": "The name of the view."
      }
    }
  },
  "resources": [
    {
      "name": "[concat(parameters('workspaceName'), '/', parameters('databaseName'), '/', parameters('viewName'))]",
      "type": "Microsoft.Synapse/workspaces/databases/views",
      "apiVersion": "2021-06-01-preview",
      "properties": {
        "definition": {
          "$schema": "https://schema.management.azure.com/schemas/2019-06-01/sql.json#",
          "contentVersion": "1.0.0.0",
          "views": [
            {
              "name": "[parameters('viewName')]",
              "definition": "<your SQL query here>"
            }
          ]
        }
      }
    }
  ]
}
```

### Considerations
* **Purpose**: Clearly define the purpose of the view. Is it for data abstraction, security, or simplifying complex queries? Understanding the purpose will help you design the view effectively.
* **Schema Design**: Ensure that the underlying schema of the tables or other views used in the view is well-designed. This includes proper normalization, appropriate data types, and indexing for performance optimization.
* **Query Optimization**: Optimize the SQL query used in the view to ensure efficient execution. Consider using appropriate join conditions, filtering criteria, and aggregations to minimize resource consumption.
* **Data Security**: Implement necessary security measures to protect sensitive data accessed through the view. This may involve setting up appropriate permissions and access controls.
* **Maintenance and Updates**: Plan for regular maintenance and updates of the view as per your data requirements. This includes handling schema changes, data updates, and ensuring data consistency.
* **Documentation**: Document the purpose, structure, and usage guidelines of the view for future reference. This will help other developers understand and utilize the view effectively.

### JOINs
When working with joins, it’s important to consider the join type (e.g., inner join, left join) and the join condition. Here’s an example of an inner join:

```
SELECT *
FROM Table1
INNER JOIN Table2 ON Table1.Column = Table2.Column;
```

## CREATE PROCEDURE
### T-SQL
```
CREATE PROCEDURE procedure_name
AS
BEGIN
    -- SQL statements go here
END;
```

### ARM Template
```
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "databaseName": {
            "type": "string",
            "metadata": {
                "description": "The name of the database where the stored procedure will be created."
            }
        },
        "procedureName": {
            "type": "string",
            "metadata": {
                "description": "The name of the stored procedure."
            }
        },
        "procedureBody": {
            "type": "string",
            "metadata": {
                "description": "The body of the stored procedure."
            }
        }
    },
    "resources": [
        {
            "name": "[concat(parameters('databaseName'), '/procedures/', parameters('procedureName'))]",
            "type": "Microsoft.Sql/servers/databases/procedures",
            "apiVersion": "2021-02-01-preview",
            "properties": {
                "definition": "[parameters('procedureBody')]"
            }
        }
    ]
}
```

### Views vs. Stored Procedures
* Views
  * Represent virtual tables.
  * Allow you to query data from multiple tables.
  * Provide a consistent interface to the underlying data.
  * Are generally used for querying data.
  * Do not allow modifications to the underlying tables.
* Stored procedures
  * Are a group of SQL statements.
  * Can perform various operations.
  * Encapsulate business logic.
  * Provide a reusable and efficient way to execute database operations.

### Considerations
* **Syntax and Features**: Synapse SQL supports many of the T-SQL features used in SQL Server. However, there may be some differences and limitations specific to Synapse SQL. You can refer to the official documentation provided by Microsoft Learn for more information on the supported features and functionality.
* **Performance Optimization**: To maximize the performance of your stored procedures, you can leverage scale-out specific features available in Synapse SQL. These features can help you optimize the execution of your SQL code against the data stored in your data warehouse.
* **Validation and Error Handling**: It’s important to implement proper validation rules and error handling mechanisms in your stored procedures. This ensures that your code handles unexpected scenarios gracefully and provides meaningful error messages when necessary.
* **Modularity and Reusability**: Stored procedures are a great way to encapsulate your SQL code into manageable units and promote code reusability. Consider designing your stored procedures in a modular fashion, allowing them to be easily reused across different parts of your solution.
* **Security and Permissions**: Ensure that you have appropriate permissions to create and execute stored procedures in Synapse SQL. You must have CREATE PROCEDURE permission in the database and ALTER permission on the schema where the procedure is being created.
* **Testing and Debugging**: It’s good practice to thoroughly test your stored procedures before deploying them to a production environment. Use debugging techniques available in Synapse SQL to identify and fix any issues during development.

## Conditional Logic (e.g., CASE expression)
* **CASE**: The CASE expression is a conditional expression that provides if-then-else type of logic to SQL. It allows you to perform different actions based on conditions specified in the expression.
```
CASE 
    WHEN condition1 THEN result1
    WHEN condition2 THEN result2
    ...
    ELSE result
END
```

* **IF…ELSE**: The IF…ELSE statement allows you to perform commands based on a condition. If the condition is true, it executes a SQL statement, and if the condition is false, it executes another SQL statement.
```
IF condition 
    BEGIN
        -- SQL statement
    END
ELSE 
    BEGIN
        -- SQL statement
    END
```

* **IIF**: The IIF function returns one of two values, depending on whether the Boolean expression evaluates to true or false.
```
IIF ( boolean_expression, true_value, false_value )
```

* **CHOOSE**: The CHOOSE function returns the item at the specified index from a list of values.
```
CHOOSE ( index, val_1, val_2 [, val_n ] )
```

* **COALESCE**: The COALESCE function returns the first non-null value in a list.
```
COALESCE ( expression [ ,...n ] )
```

* **NULLIF**: The NULLIF function returns a null value if the two specified expressions are equal.
```
NULLIF ( expression , expression )
```
