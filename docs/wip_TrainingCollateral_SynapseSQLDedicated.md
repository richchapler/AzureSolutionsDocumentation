Create Table
When creating tables, it’s important to consider naming conventions and data types. You can create tables using both the GUI and scripting. Here’s an example of creating a table using T-SQL scripting:
CREATE TABLE TableName (
Column1 INT,
Column2 VARCHAR(50),
...
);
Create Table: The process of creating tables is similar in both Synapse SQL and Azure SQL. However, Synapse SQL provides additional features such as automatic table optimization and automatic statistics collection1.
Creating Primary Key, Indexes, etc.
To create a primary key, you can use the PRIMARY KEY constraint in your CREATE TABLE statement. Here’s an example:
CREATE TABLE TableName (
Column1 INT PRIMARY KEY,
Column2 VARCHAR(50),
...
);
To create indexes, you can use the CREATE INDEX statement. Here’s an example:
CREATE INDEX IndexName ON TableName (Column1, Column2);
Create Views: Views are virtual tables that are based on the result of an SQL query. You can create views using the CREATE VIEW statement. Here’s an example:
CREATE VIEW ViewName AS
SELECT Column1, Column2
FROM TableName
WHERE Condition;
Copy
Create stored procedure (or equivalent): To create a stored procedure, you can use the CREATE PROCEDURE statement. Here’s an example:
CREATE PROCEDURE ProcedureName
AS
BEGIN
SQL statements here
END;
Queries in GUI vs. Code(script) interface: You can execute queries both in the GUI and through code/scripting interfaces. The choice depends on your preference and requirements.
Joins (special considerations?): When working with joins, it’s important to consider the join type (e.g., inner join, left join) and the join condition. Here’s an example of an inner join:
SELECT *
FROM Table1
INNER JOIN Table2 ON Table1.Column = Table2.Column;
Case Statement Equivalent in Gui? Scripting Only?: The case statement is available in both GUI and scripting interfaces. Here’s an example of using a case statement in T-SQL scripting:
SELECT Column1,
CASE
WHEN Condition1 THEN Result1
WHEN Condition2 THEN Result2
ELSE Result3
END AS NewColumn
FROM TableName;

