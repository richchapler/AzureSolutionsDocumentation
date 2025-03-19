# SQL: Configuration Optimization

This curriculum provides hands‑on experience with optimizing SQL Server configuration settings in both On-Prem environments and Azure. It focuses on three key areas: Memory, CPU, and Storage. Each section includes practical exercises that compare and contrast On-Prem best practices with the performance tuning available in Azure.

## Objectives

- Provide practical Exercises for optimizing memory, CPU, and storage configurations in SQL Server
- Compare On-Prem configuration techniques with Azure performance settings
- Reinforce best practices in instance‑level performance tuning using simple, demonstrable exercises

------------------------- ------------------------- ------------------------- -------------------------

## Memory

### On-Prem

#### Discussion

##### Maximum Server Memory

Controls the upper limit of memory that SQL Server can use. This setting is critical because it prevents SQL Server from consuming so much memory that the operating system and other applications become starved. Adjusting it helps ensure optimal performance and stability.
 Steps to find:

- Open SSMS and connect to your SQL Server instance.
- Right-click the server in Object Explorer and select Properties.
- Go to the Memory page and locate "Maximum server memory."

##### Minimum Server Memory

Ensures SQL Server reserves a baseline amount of memory. This setting helps maintain a consistent performance foundation and prevents sudden drops in available memory for query processing.
 Steps to find:

- In SSMS, open Server Properties for your SQL Server instance.
- Navigate to the Memory page and find the "Minimum server memory" setting.

##### Index Creation Memory

Specifies the amount of memory allocated for index creation operations. This setting can affect the speed of index rebuilds and creations; if set too low, index operations may run slower due to insufficient memory.
 Steps to find:

- Open SSMS and execute the following T‑SQL command:

  ```sql
  EXEC sp_configure 'index creation memory';
  ```

- Review the current value in the results.

##### Minimum Memory per Query

Determines the minimum amount of memory allocated to each query during execution. This ensures that complex queries have sufficient memory to run efficiently, thereby preventing performance degradation due to memory constraints.
 Steps to find:

- Open SSMS and execute:

  ```sql
  EXEC sp_configure 'minimum memory per query';
  ```

- Review the returned configuration value.

##### Buffer Pool Extension

Allows the use of fast storage (usually SSDs) as an extension of the buffer pool. This can improve performance in memory-constrained environments by reducing disk I/O, although it is not a substitute for having adequate physical memory.
 Steps to find:

- Open SSMS and execute:

  ```sql
  EXEC sp_configure 'buffer pool extension enabled';
  ```

- Check the configuration value; note that enabling or disabling this setting requires T‑SQL commands.

##### In‑Memory OLTP Memory Settings

Relate to memory‑optimized tables and indexes. Optimizing these settings can dramatically improve transaction processing and overall performance for workloads that utilize in‑memory OLTP.
 Steps to find:

- Open the database properties in SSMS.
- Navigate to the Files page and check for the presence of a Memory‑Optimized Filegroup.
- Optionally, query `sys.database_files` to review file settings for memory‑optimized objects.

##### Optimize for Ad Hoc Workloads

Configures SQL Server to cache only plan stubs for single-use queries, thereby reducing memory usage in the plan cache. This setting can free up memory for more frequently executed queries, improving overall performance.
 Steps to find:

- Open SSMS and run the following T‑SQL command:

  ```sql
  EXEC sp_configure 'optimize for ad hoc workloads';
  ```

- Review the current configuration; use sp_configure with RECONFIGURE to change the setting if necessary.

##### Resource Governor (Memory Configuration)

Allows you to allocate and limit memory usage for specific workloads. This ensures that high-priority tasks receive enough memory while preventing less critical workloads from consuming excessive resources.
 Steps to find:

- Open SSMS and expand the Management folder in Object Explorer.
- Right-click Resource Governor and select Properties to view the current resource pools.
- Alternatively, run T‑SQL commands (e.g., CREATE RESOURCE POOL or ALTER RESOURCE POOL) to view and configure memory limits.

##### Lock Pages in Memory

Prevents SQL Server memory from being paged out to disk, ensuring that critical data remains in RAM. This OS-level setting is important for performance, especially on servers with heavy workloads, but it requires administrator rights and a restart of the SQL Server service to take effect.
 Steps to find:

- Open SQL Server Configuration Manager.
- Right-click the SQL Server service, select Properties, and then go to the Advanced tab.
- Locate the "Lock pages in memory" setting and verify if it is enabled.

------------------------- -------------------------

#### Exercise

These step-by-step instructions allow you to demonstrate how adjusting the Maximum Server Memory setting affects query performance and overall memory usage on an On-Prem SQL Server.

Open SQL Server Management Studio (SSMS) and connect to your SQL Server instance:

- Launch SSMS
- In the Connect to Server dialog, enter your server name and authentication details, then click "Connect"

Navigate to Server Properties → Memory:

- In Object Explorer, right-click the server node (the top-level instance)
- Select "Properties" from the context menu
- In the Server Properties dialog, click on the "Memory" tab to view memory settings

Record the current Maximum Server Memory setting:

- Note the current value for "Maximum server memory (in MB)"
- This value represents the upper limit that SQL Server can use for memory allocation

Create a new database:

- Execute the following T‑SQL commands:

  ```sql
  CREATE DATABASE trainingdb;
  USE trainingdb;
  ```

Create a large table and generate sample data:

- Create a new table using a set-based approach to generate sample data quickly. For example:

  ```sql
  IF OBJECT_ID('dbo.LargeTestTable') IS NOT NULL
      DROP TABLE dbo.LargeTestTable;
  
  CREATE TABLE dbo.LargeTestTable
  (
      ID INT IDENTITY(1,1) PRIMARY KEY,
      DataValue VARCHAR(100)
  );
  
  ;WITH Tally AS (
      SELECT TOP (500000) ROW_NUMBER() OVER (ORDER BY (SELECT NULL)) AS n
      FROM sys.all_objects a CROSS JOIN sys.all_objects b
  )
  INSERT INTO dbo.LargeTestTable (DataValue)
  SELECT REPLICATE('X', 100)
  FROM Tally;
  ```

- Verify that the table is populated by running:

  ```sql
  SELECT COUNT(*) FROM dbo.LargeTestTable;
  ```

Lower the Maximum Server Memory:

- In SSMS, navigate to Server Properties → Memory
- Change “Maximum server memory” to 512MB
- Click "OK" to apply the change

_Note: Restart the SQL Server service if required_

Run a memory‑intensive query:

- Open a new query window in SSMS

- Execute a query that stresses memory. For example:

  ```sql
  SET STATISTICS TIME ON;
  SELECT * FROM dbo.LargeTestTable ORDER BY DataValue;
  SET STATISTICS TIME OFF;
  ```

- Record the execution time displayed in the Messages tab

Increase the Maximum Server Memory:

- Return to the Server Properties → Memory dialog
- Change “Maximum server memory” to default 2147483647
- Click "OK" to apply the change

_Note: Restart the SQL Server service if required_

Re-run the same memory‑intensive query:

- Open a new query window or clear the previous results

- Execute the identical query with SET STATISTICS TIME ON:

  ```sql
  SET STATISTICS TIME ON;
  SELECT * FROM dbo.LargeTestTable ORDER BY DataValue;
  SET STATISTICS TIME OFF;
  ```

- Record the new execution time from the Messages tab

Compare the execution times:

   - Analyze the differences between the execution times from the low-memory and high-memory settings
   - Observe if the query runs faster with higher memory allocation, indicating reduced memory pressure and improved performance

-------------------------

#### Recap

When SQL Server is installed by default, it’s set to use as much memory as it can because it’s designed to run on dedicated servers. However, on a VM with 8GB of memory, you need to leave enough memory for the operating system and any other applications. Here are some points and guidelines:

- Default Setting:  
  By default, SQL Server’s "max server memory" is set very high (2,147,483,647 MB) to allow it to use as much available memory as possible on a dedicated machine. This isn’t ideal for a VM where the OS also needs memory.

- Rule of Thumb:  
  A common approach is to reserve about 20–25% of total memory for the OS and other background processes. On an 8GB VM, that might mean allocating around 6GB for SQL Server (i.e. 75% of 8GB).

- Calculations:  
  - Total Memory: 8GB  
  - Reserved for OS and other services: ~2GB (this can vary based on what else is running)  
  - Memory for SQL Server: 8GB − 2GB = 6GB  
    This 6GB is a starting point; you may adjust based on your workload.

- Absolute Minimum:  
  While SQL Server can run on very low memory (1GB or even less for small, light workloads), settings as low as 32MB are far below what is necessary for even minimal functionality. For production or even testing environments, at least 1–2GB is usually needed, though more is recommended for better performance.

- Practical Steps:  
  1. Monitor your current usage: Use Performance Monitor or DMVs (like `sys.dm_os_memory_clerks`) to see how much memory SQL Server is actually using.  
  2. Gradual adjustment: Instead of drastically lowering the memory, adjust it incrementally while observing performance and stability.  
  3. Test different settings: In your case, you might try setting "Maximum server memory" to 6GB and see how the system behaves compared to higher or lower settings.

By using these guidelines and calculations, you can determine a good memory allocation for SQL Server on your 8GB VM, ensuring the OS remains responsive while SQL Server has enough resources to perform efficiently.

------------------------- -------------------------

#### Quiz

1. A DBA notices that the plan cache is bloated with single-use query plans, causing memory pressure. Which setting should be enabled to reduce excessive memory consumption by these ad hoc queries?  
   A. Maximum server memory  
   B. Minimum server memory  
   C. Optimize for ad hoc workloads  
   D. Buffer pool extension

2. In order to ensure that SQL Server does not starve the operating system and other applications of memory, which configuration setting is most critical?  
   A. Minimum memory per query  
   B. Maximum server memory  
   C. Index creation memory  
   D. Resource Governor

3. To guarantee that complex queries receive sufficient memory during execution even under peak loads, which memory setting directly governs the baseline memory allocation per query?  
   A. Optimize for ad hoc workloads  
   B. Maximum server memory  
   C. Minimum memory per query  
   D. Buffer pool extension

-------------------------

##### Answers

1. Answer: C  
   The "Optimize for ad hoc workloads" setting reduces the memory footprint of single-use query plans, helping to keep the plan cache lean and alleviate memory pressure.

2. Answer: B  
   The "Maximum server memory" setting is critical as it limits the amount of memory SQL Server can consume, ensuring the operating system and other applications retain sufficient resources.

3. Answer: C  
   "Minimum memory per query" determines the minimum memory allocated to each query, ensuring that complex queries have enough resources even during high workload periods.

------------------------- ------------------------- -------------------------

### Azure

#### Discussion

##### Service Tier Selection

Determines the memory allocation by automatically managing resources based on the selected performance tier. Higher tiers provide more memory, which can improve query performance.
 Steps to find:

- Open the Azure portal and navigate to your Azure SQL Database resource.
- Select "Configure" under Compute + Storage.
- Review the current service tier and its vCore/DTU allocation, which indirectly reflects the memory available.

##### Monitoring Memory with Query Performance Insight

Provides insights into query performance and memory usage. Monitoring these metrics helps determine if the current service tier is sufficient for your workload.
 Steps to find:

- In the Azure portal, open your Azure SQL Database resource.
- Click on "Query Performance Insight."
- Review the memory usage metrics and observe how memory allocation affects query duration.

##### Scaling Up for Increased Memory Allocation

Scaling up the service tier increases CPU, memory, and I/O resources. This is an automated process that reallocates memory based on the new tier.
 Steps to find:

- In the Azure portal, open your Azure SQL Database resource.
- Click on "Configure" under Compute + Storage.
- Increase the pricing tier to a higher level and save your changes.
- Monitor performance improvements using Query Performance Insight.

##### Automatic Memory Management

Azure SQL Database automatically manages memory allocation based on the selected service tier and workload. No manual configuration is available, which simplifies administration.
 Steps to find:

- In the Azure portal, navigate to the "Overview" or "Metrics" section of your Azure SQL Database.
- Review the current performance metrics to understand how memory is being utilized under the selected tier.

------------------------- -------------------------

#### Exercise

These step-by-step instructions allow you to demonstrate how memory allocation in Azure SQL Database impacts query performance and how Azure manages these settings.

##### Pre-Requisites

- Azure SQL Server and Database

##### Generate a Large Dataset

Connect to your Azure SQL Database using Azure Data Studio or SQL Server Management Studio (SSMS).

Execute the following to create and populate a large test table:
```sql
CREATE TABLE dbo.LargeMemoryTable (
    ID INT IDENTITY(1,1) PRIMARY KEY,
    DataValue NVARCHAR(MAX)
);

INSERT INTO dbo.LargeMemoryTable (DataValue)
SELECT REPLICATE(N'AzureSQL', 100)
FROM (SELECT TOP 500000 ROW_NUMBER() OVER (ORDER BY (SELECT NULL)) AS n
      FROM sys.all_objects a CROSS JOIN sys.all_objects b) AS Tally;
```

##### Run Initial Performance Test

Execute a memory-intensive query to test database performance:
```sql
SET STATISTICS TIME ON;
SELECT * FROM dbo.LargeMemoryTable ORDER BY DataValue;
SET STATISTICS TIME OFF;
```

Navigate to your Azure SQL Database resource in the Azure portal:

Select "Query Performance Insight" from the left menu

Record execution time, CPU percentage, and DTU usage from the displayed metrics for this query

3. Scale Up the Service Tier

- Navigate to your Azure SQL Database resource in the Azure portal.
- Under "Compute + Storage," click "Configure."
- Select a higher service tier (e.g., Standard S2 or Premium P1) to increase memory and resource availability.
- Save changes and allow Azure to complete the scaling operation.

------

3. Run Post-Scaling Performance Test

- Execute the same memory-intensive query again:

```sql
SET STATISTICS TIME ON;
SELECT * FROM dbo.LargeMemoryTable ORDER BY DataValue;
SET STATISTICS TIME OFF;
```

- Record the execution time, comparing results against your initial test.

------

4. Analyze Results Using Query Performance Insight

- In the Azure portal, navigate to your Azure SQL Database resource.
- Open the "Query Performance Insight" tool.
- Compare the recorded memory usage and performance metrics between your Basic and scaled-up service tiers.
- Observe differences in query duration, resource consumption, and memory management efficiency.

------

5. Conclude and Document Observations

- Summarize how Azure's automatic memory management impacts your workload.
- Discuss observations about query performance differences and how Azure's automated memory management adjusts to your workload needs.
- Identify scenarios where adjusting the service tier could lead to improved performance.

Here's a concise recap section for Memory optimization in Azure, following your exercise:

------

#### Recap

Azure SQL Database automatically manages memory allocation based on the service tier you select. Unlike On-Prem environments, you don't directly configure memory settings in Azure. Instead, optimizing memory in Azure involves:

- Choosing the Appropriate Service Tier Selecting a higher service tier (Standard or Premium) directly increases available memory, improving query performance, especially for memory-intensive operations.
- Monitoring Performance with Azure Tools Query Performance Insight allows you to monitor query duration, CPU usage, and DTU/vCore utilization, helping identify if your current memory allocation is sufficient for your workload.
- Scaling Dynamically Azure provides flexibility to scale resources quickly. Increasing your database’s service tier improves memory availability automatically, enhancing overall performance without manual memory tuning.

This exercise demonstrated how Azure's automatic memory management adjusts to your workload, improving efficiency and performance simply by scaling the service tier.

------

#### Quiz

1. A DBA is analyzing an Azure SQL Database workload and notices that certain queries are performing poorly. Which feature primarily determines how much memory is allocated to the database without manual intervention?  
   A. Manual configuration  
   B. Service tier selection  
   C. Resource Governor  
   D. Index creation memory

2. After scaling up an Azure SQL Database to a higher service tier, what is the expected effect on memory allocation?  
   A. Memory allocation remains unchanged  
   B. Memory allocation increases  
   C. Memory allocation decreases  
   D. Memory allocation requires manual adjustment

3. To monitor memory usage and its impact on query performance in Azure SQL Database, which Azure portal tool should be used?  
   A. SQL Server Management Studio  
   B. Query Performance Insight  
   C. Resource Monitor  
   D. Azure Data Studio

-------------------------

##### Answers

1. Answer: B  
   Service tier selection automatically governs the memory allocation in Azure SQL Database.

2. Answer: B  
   Scaling up to a higher service tier increases the available memory, which can improve query performance.

3. Answer: B  
   Query Performance Insight is the recommended tool for monitoring query performance and memory usage in Azure SQL Database.

------------------------- ------------------------- ------------------------- -------------------------

## CPU

### On-Prem

#### Discussion

##### Processor Affinity and MAXDOP

Controls how SQL Server utilizes CPU cores by specifying which processors are used for query execution and limiting the number of processors used in parallel processing. Proper configuration can improve query response times and prevent inefficient CPU usage.
 Steps to find:

- Open SSMS and connect to your SQL Server instance.
- Right-click the server in Object Explorer, select Properties, and go to the "Processors" page.
- Review the "Automatically set processor affinity mask" and "Maximum degree of parallelism" (MAXDOP) settings.

##### Resource Governor for CPU

Allows you to define resource pools and workload groups to limit and allocate CPU usage among different sessions or workloads. This helps ensure that critical tasks receive sufficient CPU resources while preventing less critical tasks from over-consuming CPU.
 Steps to find:

- Open SSMS and expand the "Management" folder in Object Explorer.
- Right-click "Resource Governor" and select Properties to view the current configuration.
- Alternatively, execute T‑SQL commands (CREATE RESOURCE POOL, ALTER RESOURCE POOL) to view or modify CPU limits.

##### Monitoring CPU Usage

Monitoring CPU performance is key to understanding workload demands and identifying bottlenecks. It helps you decide when to adjust CPU-related configurations or scale hardware resources.
 Steps to find:

- In SSMS, execute dynamic management view queries such as:

  ```sql
  SELECT * FROM sys.dm_os_performance_counters WHERE counter_name LIKE '%CPU%';
  ```

- Alternatively, use Performance Monitor (PerfMon) on the server to track CPU metrics during query execution.

##### Cost Threshold for Parallelism

Determines the threshold at which SQL Server considers a query for parallel execution. Lowering this value can force more queries to run in parallel, while a higher value limits parallelism to only expensive queries.
 Steps to find:

- Open SSMS and connect to your SQL Server instance.
- Right-click the server, select Properties, and go to the "Advanced" page.
- Locate the "Cost Threshold for Parallelism" setting to review or adjust its value.

------------------------- -------------------------

#### Exercise

- Run a parallelizable query on your On-Prem database and record its execution time

- Execute the following T‑SQL to set 

  ```
  max degree of parallelism
  ```

   (MAXDOP) to 1:

  ```sql
  sp_configure 'max degree of parallelism', 1;
  RECONFIGURE;
  ```

- Rerun the query and compare the performance

- Additionally, create a Resource Governor pool with a classifier function to throttle CPU usage for a specific login, then run a workload under that login and observe CPU usage differences

------------------------- -------------------------

#### Quiz

1. A complex analytical query on an On-Prem SQL Server is not utilizing parallelism even though its estimated cost is high. Which configuration setting adjustment is most likely to encourage parallel execution?  
   - A. Increase MAXDOP  
   - B. Lower the cost threshold for parallelism  
   - C. Adjust processor affinity  
   - D. Increase maximum server memory

2. A DBA wants to ensure that a resource-intensive batch job does not monopolize CPU resources on an On-Prem SQL Server, affecting critical queries. Which configuration tool should be used to limit CPU usage for specific workloads?  
   - A. Adjust MAXDOP  
   - B. Use Resource Governor to set CPU limits  
   - C. Configure processor affinity  
   - D. Increase the cost threshold for parallelism

3. An On-Prem SQL Server experiences sporadic CPU spikes due to ad-hoc queries, which affects scheduled analytical workloads. Which strategy is most effective in stabilizing CPU usage?  
   - A. Lower the cost threshold for parallelism  
   - B. Increase MAXDOP  
   - C. Use Resource Governor to prioritize scheduled queries over ad-hoc queries  
   - D. Configure processor affinity for critical workloads

-------------------------

##### Answers

1. Answer: B  
   Lowering the cost threshold for parallelism allows more queries to qualify for parallel execution, which can improve performance for complex analytical queries.

2. Answer: B  
   Using Resource Governor to set CPU limits prevents a single batch job from consuming excessive CPU, ensuring that other critical queries maintain performance.

3. Answer: C  
   Prioritizing scheduled queries with Resource Governor helps stabilize CPU usage by limiting the impact of sporadic ad-hoc queries.

------------------------- ------------------------- -------------------------

### Azure

#### Discussion

##### Service Tier Selection for CPU

In Azure SQL Database (PaaS), the selected service tier (DTU/vCore model) directly determines the available CPU resources. Upgrading to a higher tier increases the number of vCores or DTUs, thereby enhancing CPU performance.
 Steps to find:

- Open the Azure portal and navigate to your Azure SQL Database resource.
- Click on "Configure" under Compute + Storage.
- Review the current service tier and its associated CPU (vCore/DTU) allocation.

##### Monitoring CPU Usage in Azure

Monitoring CPU performance in Azure SQL Database helps identify workload bottlenecks and validates the adequacy of the chosen service tier.
 Steps to find:

- In the Azure portal, navigate to the "Metrics" or "Query Performance Insight" section of your Azure SQL Database.
- Select CPU-related metrics (such as CPU percentage) to observe usage trends during query execution.

##### Scaling Up for Improved CPU Performance

Scaling up the service tier in Azure SQL Database automatically increases CPU resources, which can reduce query execution time for CPU-intensive operations.
 Steps to find:

- In the Azure portal, open your Azure SQL Database resource and click on "Configure" under Compute + Storage.
- Choose a higher service tier and save your changes.
- Monitor performance improvements via Query Performance Insight or the Metrics section.

##### Query Optimization for CPU Efficiency

While direct CPU settings cannot be manually configured in Azure SQL Database (PaaS), refining query design can reduce CPU usage and improve performance.
 Steps to find:

- In the Azure portal or SSMS, run a CPU-intensive query and record its duration using Query Performance Insight.
- Review the query execution plan to identify inefficiencies.
- Optimize the query (for example, by adding an appropriate index or rewriting complex joins) and re-run the query to compare performance improvements.

------------------------- -------------------------

#### Exercise

- Create a Basic‑tier Azure SQL Database and run a CPU‑intensive query (for example, one with multiple joins and aggregates)
- Record the query duration using Query Performance Insight
- Scale the database to a higher tier (e.g., S2 or S3) and run the query again
- Compare the performance to see how increased CPU resources (vCores) reduce query time
- Optionally, refine the query design (for example, by creating an index) and observe the combined effect with a higher CPU tier

------------------------- -------------------------

#### Quiz

1. A DBA is reviewing an Azure SQL Database workload. Which factor primarily determines the CPU resources available for query processing in Azure SQL Database?  
   A. Manual configuration by the DBA  
   B. The selected service tier (vCores or DTUs)  
   C. Resource Governor settings  
   D. SQL Server Agent scheduling

2. When an Azure SQL Database is scaled up to a higher service tier, what is the expected impact on CPU performance?  
   A. CPU resources remain unchanged  
   B. CPU resources decrease  
   C. CPU resources increase  
   D. CPU resources must be manually adjusted

3. To optimize CPU performance in Azure SQL Database, which strategy is most effective?  
   A. Configuring processor affinity  
   B. Refining query design while choosing an appropriate service tier  
   C. Enabling Resource Governor  
   D. Increasing the number of data files

-------------------------

##### Answers

1. Answer: B  
   The selected service tier determines the number of vCores or DTUs, which directly governs the CPU resources available in Azure SQL Database.

2. Answer: C  
   Scaling up to a higher service tier increases the available CPU resources, thereby improving query performance.

3. Answer: B  
   Refining query design along with selecting the appropriate service tier is the recommended strategy to optimize CPU performance in Azure SQL Database.

------------------------- ------------------------- ------------------------- -------------------------

## Storage

### On-Prem

#### Discussion

##### File Groups

File groups help organize database files into logical units so that data can be spread across multiple physical disks. This can balance the I/O load and improve overall performance.
 Steps to find:

- Open SSMS and connect to your SQL Server instance.
- Expand Databases, right-click a specific database, and select Properties.
- Click on the Files page to view the list of file groups and their associated files.

##### Storage Pools and Striping

Storage pools and striping combine multiple disks into a single logical volume, allowing data to be distributed evenly across disks. This enhances I/O throughput by reducing bottlenecks on any single disk.
 Steps to find:

- In a demo environment, simulate storage pools by creating multiple virtual disks (using different drive letters) in your operating system.
- When creating a new database in SSMS, assign different data files to these separate disks.
- Verify file distribution by reviewing the file paths in the New Database dialog or under the Files page in database properties.

##### Manual File Placement

Manual file placement involves intentionally placing data files and log files on separate physical disks to minimize I/O contention. This strategy ensures that the high-write operations of the log file do not interfere with data file I/O.
 Steps to find:

- In SSMS, during the creation of a new database, click the Options or Files tab.
- Specify different file paths for data files (e.g., on one disk) and the log file (on a separate disk).
- Review and confirm the file paths in the database properties after creation.

------------------------- -------------------------

#### Exercise

- Create two virtual disks (for example, D: and E:) on your local machine or VM

- In SSMS, create a database named 

  ```
  TestDB_SingleFile
  ```

   with a single data file and log file on a single disk (e.g., C:)

  - Run a loop to insert 500,000 rows into a test table and record the execution time

- Create a second database named 

  ```
  TestDB_MultiFile
  ```

   with two data files

  - Place one data file on D: and the other on E:, with the log file on one of the disks
  - Run the same insert loop and record the execution time

- Compare the performance to see how distributing data files across disks can improve I/O throughput

------------------------- -------------------------

#### Quiz

1. A DBA is troubleshooting I/O performance on an On-Prem SQL Server and discovers that data and log files are stored on the same disk. Which configuration change is most likely to reduce I/O contention?  
   - A. Consolidate file groups  
   - B. Place data files and log files on separate high-performance disks  
   - C. Increase RAID striping across all disks  
   - D. Decrease file growth increments

2. Which strategy is most effective in distributing the database I/O load across multiple disks in an On-Prem environment?  
   - A. Using a single large file for all database operations  
   - B. Using storage pools and striping  
   - C. Implementing dynamic data masking  
   - D. Caching query results in memory

3. To verify that file distribution is optimized in an On-Prem SQL Server, which tool within SSMS would be most appropriate?  
   - A. The Files page under database properties in Object Explorer  
   - B. Query Performance Insight  
   - C. SQL Server Agent  
   - D. Resource Governor

-------------------------

##### Answers

1. Answer: B  
   Placing data files and log files on separate high-performance disks reduces I/O contention and improves performance.

2. Answer: B  
   Using storage pools and striping distributes the I/O load evenly across multiple disks.

3. Answer: A  
   The Files page in SSMS shows file distribution, allowing the DBA to verify that files are optimally placed.

------------------------- ------------------------- -------------------------

### Azure

#### Discussion

##### Performance Tiers

Determines the overall performance of an Azure SQL Database, including storage throughput. The chosen tier (DTU or vCore model) automatically allocates resources based on workload demands.
 Steps to find:

- Open the Azure portal and navigate to your Azure SQL Database resource.
- Click on "Configure" under the Compute + Storage section.
- Review the current service tier, which reflects the performance and storage allocation.

##### Built‑in High Availability and Automatic Backups

Ensures that storage performance and data integrity are maintained without manual intervention. Azure SQL Database automatically manages backup schedules and high availability, reducing administrative overhead.
 Steps to find:

- In the Azure portal, open your Azure SQL Database resource.
- Click on the "Overview" or "Backup" section to review backup history and settings.
- Note how the service provides high availability by default through geo-replication and built‑in failover capabilities.

##### Automatic Storage Management

Azure SQL Database leverages the service tier to automatically manage storage configuration, including file distribution and I/O optimization. This means no manual file placement is required, simplifying administration.
 Steps to find:

- In the Azure portal, navigate to the "Metrics" section for your Azure SQL Database.
- Monitor performance metrics such as DTU consumption and storage I/O.
- Observe how changes in workload affect these metrics, reflecting the underlying automated storage management.

------------------------- -------------------------

#### Exercise

- In the Azure portal, create a Basic‑tier Azure SQL Database.
- Run a load test by inserting a large volume of data into a test table and record response times or DTU usage.
- Scale up the database to a Standard or Premium tier.
- Rerun the load test and compare performance metrics to observe how the service tier automatically manages storage performance.

------------------------- -------------------------

#### Quiz

1. In an Azure SQL Database environment, which feature automatically manages storage performance?  
   - A. Performance tiers (DTU/vCore)  
   - B. Resource Governor  
   - C. Manual filegroup configuration  
   - D. SQL Server Agent

2. Which built‑in feature of Azure SQL Database helps ensure storage efficiency through high availability and automatic backups?  
   - A. Manual RAID configuration  
   - B. Automatic high availability and backups  
   - C. Custom disk partitioning  
   - D. Resource Governor

3. When scaling an Azure SQL Database to a higher tier, which action is most likely to improve storage performance?  
   - A. Enabling Resource Governor  
   - B. Configuring additional filegroups  
   - C. Increasing the service tier  
   - D. Manually redistributing data files

-------------------------

##### Answers

1. Answer: A  
   Performance tiers (DTU/vCore) automatically manage storage performance in Azure SQL Database, eliminating the need for manual filegroup configuration.

2. Answer: B  
   Automatic high availability and backups are built into Azure SQL Database, ensuring storage efficiency without manual intervention.

3. Answer: C  
   Increasing the service tier boosts resource allocation, which improves storage performance in Azure SQL Database.
