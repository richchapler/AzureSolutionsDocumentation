# SQL: Configuration Optimization

This curriculum provides hands-on experience with optimizing SQL Server configuration settings in both on‑premises environments and Azure SQL Database (PaaS). It focuses on three key areas: Storage, Memory, and CPU. Each section includes practical exercises that compare and contrast on‑premises best practices with the performance tuning available in Azure.

## Objectives

- Provide practical hands‑on exercises for optimizing storage, memory, and CPU configurations in SQL Server
- Compare on‑premises configuration techniques with Azure SQL Database (PaaS) performance settings
- Reinforce best practices in instance-level performance tuning using simple, demonstrable exercises

-------------------------

## Storage

### On‑Premises SQL Server: Hands‑On Exercise

- Create two virtual disks (for example, D: and E:) on your local machine or VM  
- In SSMS, create a database named `TestDB_SingleFile` with a single data file and log file on a single disk (e.g., C:)  
  - Run a loop to insert 500,000 rows into a test table and record the execution time  
- Create a second database named `TestDB_MultiFile` with two data files:
  - Place one data file on D: and the other on E:, with the log file on one of the disks  
  - Run the same insert loop and record the execution time  
- Compare the performance to see how distributing data files across disks can improve I/O throughput

### Azure SQL Database (PaaS): Hands‑On Exercise

- In the Azure portal, create a Basic‑tier Azure SQL Database  
- Run a load test by inserting a large volume of data into a test table and record response times or DTU usage  
- Scale up the database to a Standard or Premium tier  
- Rerun the load test and compare performance metrics to observe how the service tier automatically manages storage performance

-------------------------

## Memory

### On‑Premises SQL Server: Hands‑On Exercise

- Open SSMS and navigate to Server Properties → Memory  
- Set “Maximum server memory” to a lower value (for example, 512 MB) and run a memory‑intensive query (such as a large sort or hash join)  
- Record the query execution time  
- Increase “Maximum server memory” and run the query again  
- Compare the execution times to see the impact of memory allocation on performance  
- Optionally, check for memory pressure messages in the SQL Server error log or via dynamic management views

### Azure SQL Database (PaaS): Hands‑On Exercise

- In the Azure portal, open your Basic‑tier Azure SQL Database  
- Execute a moderately large query and record its duration using Query Performance Insight  
- Scale up to a higher service tier and rerun the query  
- Compare the results to observe how the higher tier (with increased memory allocation) improves performance

-------------------------

## CPU

### On‑Premises SQL Server: Hands‑On Exercise

- Run a parallelizable query on your on‑premises database and record its execution time  
- Execute the following T-SQL to set `max degree of parallelism` (MAXDOP) to 1:
  ```sql
  sp_configure 'max degree of parallelism', 1;
  RECONFIGURE;
  ```
- Rerun the query and compare the performance  
- Additionally, create a Resource Governor pool with a classifier function to throttle CPU usage for a specific login, then run a workload under that login and observe CPU usage differences

### Azure SQL Database (PaaS): Hands‑On Exercise

- Create a Basic‑tier Azure SQL Database and run a CPU‑intensive query (for example, one with multiple joins and aggregates)  
- Record the query duration using Query Performance Insight  
- Scale the database to a higher tier (e.g., S2 or S3) and run the query again  
- Compare the performance to see how increased CPU resources (vCores) reduce query time  
- Optionally, refine the query design (e.g., by creating an index) and observe the combined effect with a higher CPU tier
