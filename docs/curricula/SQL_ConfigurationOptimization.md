# SQL: Configuration Optimization

This curriculum provides hands‑on experience with optimizing SQL Server configuration settings in both on-prem environments and Azure. It focuses on three key areas: Memory, CPU, and Storage. Each section includes practical exercises that compare and contrast on-prem best practices with the performance tuning available in Azure.

## Objectives

- Provide practical Exercises for optimizing memory, CPU, and storage configurations in SQL Server
- Compare on-prem configuration techniques with Azure performance settings
- Reinforce best practices in instance‑level performance tuning using simple, demonstrable exercises

## Pre-Requisites

- Virtual Machine with multiple CPUs and SQL Server
- Azure SQL Server and Database

------------------------- ------------------------- ------------------------- -------------------------

## Memory

### On-Prem

#### Discussion

##### Maximum Server Memory

Controls the upper limit of memory that SQL Server can use. This setting is critical because it prevents SQL Server from consuming so much memory that the operating system and other applications become starved. Adjusting it helps ensure optimal performance and stability.

- Open SQL Server Management Studio and connect to your SQL Server instance.
- Right-click the server in Object Explorer and select Properties.
- Go to the Memory page and locate "Maximum server memory."

##### Minimum Server Memory

Ensures SQL Server reserves a baseline amount of memory. This setting helps maintain a consistent performance foundation and prevents sudden drops in available memory for query processing.

- In SQL Server Management Studio, open Server Properties for your SQL Server instance.
- Navigate to the Memory page and find the "Minimum server memory" setting.

##### Index Creation Memory

Specifies the amount of memory allocated for index creation operations. This setting can affect the speed of index rebuilds and creations; if set too low, index operations may run slower due to insufficient memory.

- Open SQL Server Management Studio and execute the following T‑SQL command:

  ```sql
  EXEC sp_configure 'index creation memory';
  ```

- Review the current value in the results.

##### Minimum Memory per Query

Determines the minimum amount of memory allocated to each query during execution. This ensures that complex queries have sufficient memory to run efficiently, thereby preventing performance degradation due to memory constraints.

- Open SQL Server Management Studio and execute:

  ```sql
  EXEC sp_configure 'minimum memory per query';
  ```

- Review the returned configuration value.

##### Buffer Pool Extension

Allows the use of fast storage (usually SSDs) as an extension of the buffer pool. This can improve performance in memory-constrained environments by reducing disk I/O, although it is not a substitute for having adequate physical memory.

- Open SQL Server Management Studio and execute:

  ```sql
  EXEC sp_configure 'buffer pool extension enabled';
  ```

- Check the configuration value; note that enabling or disabling this setting requires T‑SQL commands.

##### In‑Memory OLTP Memory Settings

Relate to memory‑optimized tables and indexes. Optimizing these settings can dramatically improve transaction processing and overall performance for workloads that utilize in‑memory OLTP.

- Open the database properties in SQL Server Management Studio.
- Navigate to the Files page and check for the presence of a Memory‑Optimized Filegroup.
- Optionally, query `sys.database_files` to review file settings for memory‑optimized objects.

##### Optimize for Ad Hoc Workloads

Configures SQL Server to cache only plan stubs for single-use queries, thereby reducing memory usage in the plan cache. This setting can free up memory for more frequently executed queries, improving overall performance.

- Open SQL Server Management Studio and run the following T‑SQL command:

  ```sql
  EXEC sp_configure 'optimize for ad hoc workloads';
  ```

- Review the current configuration; use sp_configure with RECONFIGURE to change the setting if necessary.

##### Resource Governor (Memory Configuration)

Allows you to allocate and limit memory usage for specific workloads. This ensures that high-priority tasks receive enough memory while preventing less critical workloads from consuming excessive resources.

- Open SQL Server Management Studio and expand the Management folder in Object Explorer.
- Right-click Resource Governor and select Properties to view the current resource pools.
- Alternatively, run T‑SQL commands (e.g., CREATE RESOURCE POOL or ALTER RESOURCE POOL) to view and configure memory limits.

##### Lock Pages in Memory

Prevents SQL Server memory from being paged out to disk, ensuring that critical data remains in RAM. This OS-level setting is important for performance, especially on servers with heavy workloads, but it requires administrator rights and a restart of the SQL Server service to take effect.

- Open SQL Server Configuration Manager.
- Right-click the SQL Server service, select Properties, and then go to the Advanced tab.
- Locate the "Lock pages in memory" setting and verify if it is enabled.

------------------------- -------------------------

#### Exercise

Follow these step-by-step instructions to demonstrate how adjusting the Maximum Server Memory setting affects query performance and overall memory usage on an on-prem SQL Server.

##### Connect 

- Open SQL Server Management Studio and connect to your SQL Server instance

##### Review Settings

- In Object Explorer, right-click the server node and select "Properties" from the resulting menu
- In the Server Properties dialog, click on the "Memory" tab
- Note the current value for "Maximum server memory (in MB)" (default: 2147483647)

##### Prepare Sample Data

- Create a new database:

  ```sql
  CREATE DATABASE trainingdb;
  USE trainingdb;
  ```

- Create a new table using a set-based approach to generate sample data quickly:

  ```sql
  IF OBJECT_ID('dbo.LargeTestTable') IS NOT NULL
      DROP TABLE dbo.LargeTestTable;
  
  CREATE TABLE dbo.LargeTestTable
  (
      ID INT IDENTITY(1,1) PRIMARY KEY,
      DataValue VARCHAR(100)
  );
  
  WITH X AS (
      SELECT TOP (500000) ROW_NUMBER() OVER (ORDER BY (SELECT NULL)) AS n
      FROM sys.all_objects a CROSS JOIN sys.all_objects b
  )
  INSERT INTO dbo.LargeTestTable (DataValue)
  SELECT REPLICATE('X', 100)
  FROM X;
  ```

- Verify that the table is populated by running:

  ```sql
  SELECT COUNT(*) FROM dbo.LargeTestTable;
  ```

##### Lower Memory Setting

- In SQL Server Management Studio, navigate to Server Properties → Memory
- Change “Maximum server memory” to 512MB
- Click "OK" to apply the change
- Restart the SQL Server service

##### Stress Memory

- Execute the following T-SQL:

  ```sql
  SET STATISTICS TIME ON;
  SELECT * FROM dbo.LargeTestTable ORDER BY DataValue;
  SET STATISTICS TIME OFF;
  ```

- Record the execution time displayed in the Messages tab

##### Increase Memory Setting

- Return to the Server Properties → Memory dialog
- Change “Maximum server memory” to default 2147483647
- Click "OK" to apply the change
- Restart the SQL Server service

##### Re-Stress Memory

- Execute the following T-SQL:

  ```sql
  SET STATISTICS TIME ON;
  SELECT * FROM dbo.LargeTestTable ORDER BY DataValue;
  SET STATISTICS TIME OFF;
  ```

- Record the new execution time from the Messages tab

##### Compare Execution Times

- Observe whether the query runs faster with higher memory allocation, indicating reduced memory pressure and improved performance

-------------------------

#### Recap

When SQL Server is installed by default, it’s set to use as much memory as it can because it’s designed to run on dedicated servers. However, on a VM with 8GB of memory, you need to leave enough memory for the operating system and any other applications. Here are some points and guidelines:

- **Default Setting**: By default, SQL Server’s "max server memory" is set very high (2,147,483,647 MB) to allow it to use as much available memory as possible on a dedicated machine. This isn’t ideal for a VM where the OS also needs memory.

- **Rule of Thumb**: A common approach is to reserve about 20–25% of total memory for the OS and other background processes. On an 8GB VM, that might mean allocating around 6GB for SQL Server (i.e. 75% of 8GB).

- **Calculations**:  

  - Total Memory: 8GB  
  - Reserved for OS and other services: ~2GB (this can vary based on what else is running)  
  - Memory for SQL Server: 8GB − 2GB = 6GB  
    This 6GB is a starting point; you may adjust based on your workload.

- **Absolute Minimum**: While SQL Server can run on very low memory (1GB or even less for small, light workloads), settings as low as 32MB are far below what is necessary for even minimal functionality. For production or even testing environments, at least 1–2GB is usually needed, though more is recommended for better performance.

- Practical Steps:  

  - **Monitor your current usage**: Use Performance Monitor or DMVs (like `sys.dm_os_memory_clerks`) to see how much memory SQL Server is actually using.  

  - **Gradual adjustment**: Instead of drastically lowering the memory, adjust it incrementally while observing performance and stability.  

  - **Test different settings**: In your case, you might try setting "Maximum server memory" to 6GB and see how the system behaves compared to higher or lower settings.

By using these guidelines and calculations, you can determine a good memory allocation for SQL Server on your 8GB VM, ensuring the OS remains responsive while SQL Server has enough resources to perform efficiently.

------------------------- -------------------------

#### Quiz

1. A database administrator notices that the plan cache is bloated with single-use query plans, causing memory pressure. Which setting should be enabled to reduce excessive memory consumption by these ad hoc queries?  
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

##### Service Tier Selection

Determines the memory allocation by automatically managing resources based on the selected performance tier. Higher tiers provide more memory, which can improve query performance.

- Open the Azure portal and navigate to your Azure SQL Database resource.
- Select "Configure" under Compute + Storage.
- Review the current service tier and its vCore/DTU allocation, which indirectly reflects the memory available.

##### Monitoring Memory with Query Performance Insight

Provides insights into query performance and memory usage. Monitoring these metrics helps determine if the current service tier is sufficient for your workload.

- In the Azure portal, open your Azure SQL Database resource.
- Click on "Query Performance Insight."
- Review the memory usage metrics and observe how memory allocation affects query duration.

##### Scaling Up for Increased Memory Allocation

Scaling up the service tier increases CPU, memory, and I/O resources. This is an automated process that reallocates memory based on the new tier.

- In the Azure portal, open your Azure SQL Database resource.
- Click on "Configure" under Compute + Storage.
- Increase the pricing tier to a higher level and save your changes.
- Monitor performance improvements using Query Performance Insight.

##### Automatic Memory Management

Azure SQL Database automatically manages memory allocation based on the selected service tier and workload. No manual configuration is available, which simplifies administration.

- In the Azure portal, navigate to the "Overview" or "Metrics" section of your Azure SQL Database.
- Review the current performance metrics to understand how memory is being utilized under the selected tier.

------------------------- -------------------------

#### No Exercise

In Azure SQL Database, **it's not possible to directly configure memory settings** like you can on-prem (e.g., "Maximum Server Memory"). Azure manages memory **automatically** based on the chosen **service tier** and **compute resources** (vCores/DTUs).

------------------------- -------------------------

#### Quiz

1. A database administrator is analyzing an Azure SQL Database workload and notices that certain queries are performing poorly. Which feature primarily determines how much memory is allocated to the database without manual intervention?  
   A. Manual configuration  
   B. Service tier selection  
   C. Resource Governor  
   D. Index creation memory
2. After scaling up an Azure SQL Database to a higher service tier, what is the expected effect on memory allocation?  
   A. Memory allocation remains unchanged  
   B. Memory allocation increases  
   C. Memory allocation decreases  
   D. Memory allocation requires manual adjustment

-------------------------

##### Answers

1. Answer: B  
   Service tier selection automatically governs the memory allocation in Azure SQL Database.
2. Answer: B  
   Scaling up to a higher service tier increases the available memory, which can improve query performance.

------------------------- ------------------------- ------------------------- -------------------------

## CPU

### On-Prem

#### Discussion

##### Processor Affinity and MAXDOP

Controls how SQL Server utilizes CPU cores by specifying which processors are used for query execution and limiting the number of processors used in parallel processing. Proper configuration can improve query response times and prevent inefficient CPU usage.

- Open SQL Server Management Studio and connect to your SQL Server instance.
- Right-click the server in Object Explorer, select Properties, and go to the "Processors" page.
- Review the "Automatically set processor affinity mask" and "Maximum degree of parallelism" (MAXDOP) settings.

##### Resource Governor for CPU

Allows you to define resource pools and workload groups to limit and allocate CPU usage among different sessions or workloads. This helps ensure that critical tasks receive sufficient CPU resources while preventing less critical tasks from over-consuming CPU.

- Open SQL Server Management Studio and expand the "Management" folder in Object Explorer.
- Right-click "Resource Governor" and select Properties to view the current configuration.
- Alternatively, execute T‑SQL commands (CREATE RESOURCE POOL, ALTER RESOURCE POOL) to view or modify CPU limits.

##### Monitoring CPU Usage

Monitoring CPU performance is key to understanding workload demands and identifying bottlenecks. It helps you decide when to adjust CPU-related configurations or scale hardware resources.

- In SQL Server Management Studio, execute dynamic management view queries such as:

  ```sql
  SELECT * FROM sys.dm_os_performance_counters WHERE counter_name LIKE '%CPU%';
  ```

- Alternatively, use Performance Monitor (PerfMon) on the server to track CPU metrics during query execution.

##### Cost Threshold for Parallelism

Determines the threshold at which SQL Server considers a query for parallel execution. Lowering this value can force more queries to run in parallel, while a higher value limits parallelism to only expensive queries.

- Open SQL Server Management Studio and connect to your SQL Server instance.
- Right-click the server, select Properties, and go to the "Advanced" page.
- Locate the "Cost Threshold for Parallelism" setting to review or adjust its value.

------------------------- -------------------------

#### Exercise

Follow these step-by-step instructions to demonstrate how adjusting CPU configuration settings—such as MAXDOP and Resource Governor—affects query performance and overall CPU resource management on an on‑prem SQL Server.

##### VM Configuration Note

To ensure you can fully explore parallel query execution, memory usage, and storage best practices, provision a VM with at least 4 vCPUs and 16GB of RAM (for example, a D4s_v5 in Azure). This allows SQL Server to effectively scale parallel queries and ensures you have enough memory overhead to experiment with various configuration settings.

##### Connect

- Open SQL Server Management Studio and connect to your SQL Server instance

##### Review Settings

- In Object Explorer, right-click the server node and select "Properties" from the resulting menu
- In the Server Properties dialog, click on the "Advanced" tab
- Note the current value for "Max Degree of Parallelism" (default: 0)

##### Prepare Sample Data

- Click "New Query" and execute:

  ```sql
  CREATE DATABASE trainingdb;
  USE trainingdb;
  ```

- Create a new table:

  ```sql
  IF OBJECT_ID('dbo.LargeCPUTable') IS NOT NULL DROP TABLE dbo.LargeCPUTable;
  
  CREATE TABLE dbo.LargeCPUTable ( ID INT IDENTITY(1,1) PRIMARY KEY, DataValue VARCHAR(100) );
  
  WITH X AS (
      SELECT TOP (1000000) ROW_NUMBER() OVER (ORDER BY (SELECT NULL)) AS n
      FROM sys.all_objects a CROSS JOIN sys.all_objects b
  )
  INSERT INTO dbo.LargeCPUTable (DataValue)
  SELECT REPLICATE('X', 100)
  FROM X;
  ```

##### Review Settings

- Confirm setting values in the `run_value` column:

  ```sql
  EXEC sp_configure 'show advanced options';
  EXEC sp_configure 'max degree of parallelism';
  EXEC sp_configure 'cost threshold for parallelism';
  ```

  - **show advanced options**:
    - Possible values: 0 (disabled) or 1 (enabled)
  - **max degree of parallelism**:
    - Possible values: 0 (use all available CPUs) or any positive integer (typically 1 up to the number of logical processors, with an upper bound of 32767)
  - **cost threshold for parallelism**:
    - Possible values: Any integer from 0 up to 32767 (default is 5)

##### Minimize Settings

- Force serial execution:

  ```sql
  EXEC sp_configure 'show advanced options', 1;
  RECONFIGURE;
  
  EXEC sp_configure 'max degree of parallelism', 1;
  RECONFIGURE;
  ```

###### Estimate Execution Plan

- Paste the following T-SQL command and click "Display Estimated Execution Plan":

  ```sql
  SELECT COUNT_BIG(*)
  FROM dbo.LargeCPUTable t1 CROSS JOIN dbo.LargeCPUTable t2
  WHERE t1.ID % 2 = 0 AND t2.ID % 3 = 0;
  ```

- Observe that the Execution Plan does not include "Parallelism"

##### Maximize Settings

- Allow parallel execution:

  ```sql
  EXEC sp_configure 'show advanced options', 1;
  RECONFIGURE;
  
  EXEC sp_configure 'max degree of parallelism', 0;
  RECONFIGURE;
  
  EXEC sp_configure 'cost threshold for parallelism', 0;
  RECONFIGURE;
  ```

###### Estimate Execution Plan

- Paste the following T-SQL command and click "Display Estimated Execution Plan":

  ```sql
  SELECT COUNT_BIG(*)
  FROM dbo.LargeCPUTable t1 CROSS JOIN dbo.LargeCPUTable t2
  WHERE t1.ID % 2 = 0 AND t2.ID % 3 = 0;
  ```

- Observe that the Execution Plan includes "Parallelism"


------------------------- -------------------------

#### Quiz

1. A complex analytical query on an on-prem SQL Server is not utilizing parallelism even though its estimated cost is high. Which configuration setting adjustment is most likely to encourage parallel execution?  
   A. Increase MAXDOP  
   B. Lower the cost threshold for parallelism  
   C. Adjust processor affinity  
   D. Increase maximum server memory

2. A database administrator wants to ensure that a resource-intensive batch job does not monopolize CPU resources on an on-prem SQL Server, affecting critical queries. Which configuration tool should be used to limit CPU usage for specific workloads?  
   A. Adjust MAXDOP  
   B. Use Resource Governor to set CPU limits  
   C. Configure processor affinity  
   D. Increase the cost threshold for parallelism

3. An on-prem SQL Server experiences sporadic CPU spikes due to ad-hoc queries, which affects scheduled analytical workloads. Which strategy is most effective in stabilizing CPU usage?  
   A. Lower the cost threshold for parallelism  
   B. Increase MAXDOP  
   C. Use Resource Governor to prioritize scheduled queries over ad-hoc queries  
   D. Configure processor affinity for critical workloads

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

- Open the Azure portal and navigate to your Azure SQL Database resource.
- Click on "Configure" under Compute + Storage.
- Review the current service tier and its associated CPU (vCore/DTU) allocation.

##### Monitoring CPU Usage in Azure

Monitoring CPU performance in Azure SQL Database helps identify workload bottlenecks and validates the adequacy of the chosen service tier.

- In the Azure portal, navigate to the "Metrics" or "Query Performance Insight" section of your Azure SQL Database.
- Select CPU-related metrics (such as CPU percentage) to observe usage trends during query execution.

##### Scaling Up for Improved CPU Performance

Scaling up the service tier in Azure SQL Database automatically increases CPU resources, which can reduce query execution time for CPU-intensive operations.

- In the Azure portal, open your Azure SQL Database resource and click on "Configure" under Compute + Storage.
- Choose a higher service tier and save your changes.
- Monitor performance improvements via Query Performance Insight or the Metrics section.

##### Query Optimization for CPU Efficiency

While direct CPU settings cannot be manually configured in Azure SQL Database (PaaS), refining query design can reduce CPU usage and improve performance.

- In the Azure portal or SQL Server Management Studio, run a CPU-intensive query and record its duration using Query Performance Insight.
- Review the query execution plan to identify inefficiencies.
- Optimize the query (for example, by adding an appropriate index or rewriting complex joins) and re-run the query to compare performance improvements.

------------------------- -------------------------

#### Exercise

Follow these step-by-step instructions using the Query Editor in the Azure Portal to demonstrate how scaling an Azure SQL Database can improve performance for CPU‑intensive queries. Note that while Azure SQL Database manages memory automatically, scaling up increases both CPU resources and available memory.

##### Connect

- In the Azure portal, navigate to your existing SQL Database resource.
- Click on **Query Editor (preview)** from the left-hand menu.
- Log in using your database credentials.

##### Prepare Sample Data

- Create and populate table:

  ```sql
  IF OBJECT_ID('dbo.CPU_Test', 'U') IS NOT NULL
      DROP TABLE dbo.CPU_Test;
  
  CREATE TABLE dbo.CPU_Test
  (
      ID INT IDENTITY(1,1) PRIMARY KEY,
      DataValue VARCHAR(100)
  );
  
  WITH X AS (
      SELECT TOP (25000) ROW_NUMBER() OVER (ORDER BY (SELECT NULL)) AS n
      FROM sys.all_objects a CROSS JOIN sys.all_objects b
  )
  INSERT INTO dbo.CPU_Test (DataValue)
  SELECT REPLICATE('X', 100)
  FROM X;
  ```

- Verify success:

  ```sql
  SELECT COUNT(*) FROM dbo.CPU_Test;
  ```

##### Review Settings

- **Check Service Tier and Edition**
  This query displays the current service tier and edition for your database:

  ```sql
  SELECT 
      DATABASEPROPERTYEX(DB_NAME(), 'Edition') AS Edition,
      DATABASEPROPERTYEX(DB_NAME(), 'ServiceObjective') AS ServiceObjective;
  ```

  These values tell you the selected tier (for example, Standard S0, S2, or S3) and provide an indication of the CPU and memory resources available based on your chosen tier.

- **Review System Information**
  Although Azure SQL Database is a managed service and doesn’t expose detailed hardware specs, you can get approximate resource details by running:

  ```sql
  SELECT * FROM sys.dm_os_sys_info;
  ```

  Note values:

  - **cpu_ticks**: Cumulative CPU ticks since SQL Server started
  - **ms_ticks**: Cumulative time in milliseconds since SQL Server started
  - **cpu_count**: Number of logical CPUs available
  - **hyperthread_ratio**: Ratio indicating the hyper-threading multiplier

  These bullets provide a concise overview of each key column, which helps in understanding what information is available from **sys.dm_os_sys_info**.

##### Execute Query

```sql
SELECT COUNT_BIG(*)
FROM dbo.CPU_Test t1 JOIN dbo.CPU_Test t2 ON t1.ID <> t2.ID
WHERE t1.ID % 2 = 0 AND t2.ID % 3 = 0;
```

Note time elapsed.

##### Scale Database

Navigate to SQL >> Settings >> Compute + Storage and adjust settings:

1. **Max vCores**
   - Defines the upper limit of CPU resources that your database can use
   - During periods of high workload, the database can scale up to this maximum, potentially reducing query times and improving overall performance
   - Increasing max vCores is beneficial if you anticipate or experience spikes in CPU demand
2. **Min vCores**
   - Defines the baseline amount of CPU resources that are always allocated to your database
   - A higher min vCores ensures more consistent performance because you avoid waiting for the service to “warm up” or scale out when queries arrive
   - However, a higher min vCores also increases the baseline cost, as you pay for these allocated resources even when your database is idle

###### Deciding How to Adjust Min and Max vCores

- **Performance Sensitivity**: If you have critical workloads that need consistently fast response times, you might raise the **min vCores** so the database doesn’t have to ramp up from near zero.
- **Cost Optimization**: If cost is a major concern and your workload is intermittent, you can keep **min vCores** lower, allowing the database to scale down during idle times, but expect some performance delay when the load first ramps up.
- **Handling Peak Loads**: If you occasionally run CPU‑intensive queries (such as cross joins or large aggregations), raising the **max vCores** ensures your database can scale up to handle those spikes without throttling.

###### Practical Steps

1. **Monitor Current Usage**
   - Use **Azure Metrics** or **Query Performance Insight** to see your current CPU usage patterns
   - Identify whether you’re regularly hitting your max vCores or if performance suffers during spikes
2. **Adjust Gradually**
   - Increase **max vCores** by one tier at a time and observe query performance improvements
   - If you have frequent workloads that need consistent performance, raise **min vCores** to avoid scale‑up delays
3. **Test Your Workload**
   - Run your most demanding queries (like the cross join examples) to see if performance meets expectations
   - If queries still take too long or you see throttling, consider further increasing **max vCores** or raising **min vCores**

By fine‑tuning **min vCores** and **max vCores** within the Serverless model, you can strike the right balance between cost savings and performance for your specific workload.

##### Execute Query

Navigate to Query Editor and re-execute:

```sql
SELECT COUNT_BIG(*)
FROM dbo.CPU_Test t1 JOIN dbo.CPU_Test t2 ON t1.ID <> t2.ID
WHERE t1.ID % 2 = 0 AND t2.ID % 3 = 0;
```

Compare time elapsed with previous run.

------------------------- -------------------------

#### Quiz

1. A database administrator is reviewing an Azure SQL Database workload. Which factor primarily determines the CPU resources available for query processing in Azure SQL Database?  
   A. Manual configuration by the database administrator  
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

- Open SQL Server Management Studio and connect to your SQL Server instance.
- Expand Databases, right-click a specific database, and select Properties.
- Click on the Files page to view the list of file groups and their associated files.

##### Storage Pools and Striping

Storage pools and striping combine multiple disks into a single logical volume, allowing data to be distributed evenly across disks. This enhances I/O throughput by reducing bottlenecks on any single disk.
Steps to find:

- In a demo environment, simulate storage pools by creating multiple virtual disks (using different drive letters) in your operating system.
- When creating a new database in SQL Server Management Studio, assign different data files to these separate disks.
- Verify file distribution by reviewing the file paths in the New Database dialog or under the Files page in database properties.

##### Manual File Placement

Manual file placement involves intentionally placing data files and log files on separate physical disks to minimize I/O contention. This strategy ensures that the high-write operations of the log file do not interfere with data file I/O.
Steps to find:

- In SQL Server Management Studio, during the creation of a new database, click the Options or Files tab.
- Specify different file paths for data files (e.g., on one disk) and the log file (on a separate disk).
- Review and confirm the file paths in the database properties after creation.

------------------------- -------------------------

#### Exercise

Follow these step-by-step instructions to demonstrate how distributing data files across multiple disks affects I/O throughput and query performance on an on-prem SQL Server.

##### Add Disks

- In Azure Portal, navigate to your virtual machine >> Settings >> Disks
  - Click "+ Create and attach a new disk" twice {e.g., `SQLDisk1` and `SQLDisk2`}, then click "Apply"
  - Restart VM and then connect

- On the VM, open `diskmgmt.msc`... when prompted, initial new disks with partition style "GPT..."

  - The disks will now show as `"Unallocated"`... right-click unallocated space and select "New Simple Volume" from the resulting menu

  - Step through the "New Simple Volume Wizard" for each

- Open File Explorer and verify that the new drives are visible and ready to use

##### Create Databases

- Open SQL Server Management Studio, connect to your SQL Server instance, then execute:

  ```sql
  CREATE DATABASE TestDB_SingleFile
  ON PRIMARY (
      NAME = TestDB_SingleFile_Data,
      FILENAME = 'C:\SQLData\TestDB_SingleFile_Data.mdf'
  )
  LOG ON (
      NAME = TestDB_SingleFile_Log,
      FILENAME = 'C:\SQLData\TestDB_SingleFile_Log.ldf'
  );
  
  CREATE DATABASE TestDB_MultiFile
  ON PRIMARY (
      NAME = TestDB_MultiFile_Data1,
      FILENAME = 'D:\SQLData\TestDB_MultiFile_Data1.mdf'
  ),
  FILEGROUP FG2 (
      NAME = TestDB_MultiFile_Data2,
      FILENAME = 'E:\SQLData\TestDB_MultiFile_Data2.ndf'
  )
  LOG ON (
      NAME = TestDB_MultiFile_Log,
      FILENAME = 'D:\SQLData\TestDB_MultiFile_Log.ldf'
  );
  ```

##### Populate Test Tables

- In each database, execute the following to create and populate tables:

  ```sql
  USE TestDB_SingleFile;
  
  IF OBJECT_ID('dbo.IOTestTable') IS NOT NULL DROP TABLE dbo.IOTestTable;
  
  CREATE TABLE dbo.IOTestTable (
      ID INT IDENTITY(1,1) PRIMARY KEY,
      DataValue VARCHAR(100)
  );
  
  WITH X AS (
      SELECT TOP (500000) ROW_NUMBER() OVER (ORDER BY (SELECT NULL)) AS n
      FROM sys.all_objects a CROSS JOIN sys.all_objects b
  )
  INSERT INTO dbo.IOTestTable (DataValue)
  SELECT REPLICATE('X', 100)
  FROM X;
  
  USE TestDB_MultiFile;
  
  IF OBJECT_ID('dbo.IOTestTable') IS NOT NULL DROP TABLE dbo.IOTestTable;
  
  CREATE TABLE dbo.IOTestTable (
      ID INT IDENTITY(1,1) PRIMARY KEY,
      DataValue VARCHAR(100)
  ) ON FG2;
  
  WITH X AS (
      SELECT TOP (500000) ROW_NUMBER() OVER (ORDER BY (SELECT NULL)) AS n
      FROM sys.all_objects a CROSS JOIN sys.all_objects b
  )
  INSERT INTO dbo.IOTestTable (DataValue)
  SELECT REPLICATE('X', 100)
  FROM X;
  ```

##### Stress Storage

- Execute the following query separately on both databases, noting execution times from the Messages tab:

  ```sql
  SET STATISTICS TIME ON;
  SELECT COUNT(*) FROM dbo.IOTestTable WHERE DataValue LIKE '%XXX%';
  SET STATISTICS TIME OFF;
  ```

##### Compare Results

- Observe and compare query execution times between the single-file and multi-file databases, noting improvements due to reduced I/O contention from distributing files across disks

#### Quiz

1. A database administrator is troubleshooting I/O performance on an on-prem SQL Server and discovers that data and log files are stored on the same disk. Which configuration change is most likely to reduce I/O contention?  

​	A. Consolidate file groups  

​	B. Place data files and log files on separate high-performance disks  

​	C. Increase RAID striping across all disks  

​	D. Decrease file growth increments

2. Which strategy is most effective in distributing the database I/O load across multiple disks in an on-prem environment?  

​	A. Using a single large file for all database operations  

​	B. Using storage pools and striping  

​	C. Implementing dynamic data masking  

​	D. Caching query results in memory

3. To verify that file distribution is optimized in an on-prem SQL Server, which tool within SQL Server Management Studio would be most appropriate?  

​	A. The Files page under database properties in Object Explorer  

​	B. Query Performance Insight  

​	C. SQL Server Agent  

​	D. Resource Governor

-------------------------

##### Answers

1. Answer: B  
   Placing data files and log files on separate high-performance disks reduces I/O contention and improves performance.

2. Answer: B  
   Using storage pools and striping distributes the I/O load evenly across multiple disks.

3. Answer: A  
   The Files page in SQL Server Management Studio shows file distribution, allowing the database administrator to verify that files are optimally placed.

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

​	A. Performance tiers (DTU/vCore)  

​	B. Resource Governor  

​	C. Manual filegroup configuration  

​	D. SQL Server Agent

2. Which built‑in feature of Azure SQL Database helps ensure storage efficiency through high availability and automatic backups?  

​	A. Manual RAID configuration  

​	B. Automatic high availability and backups  

​	C. Custom disk partitioning  

​	D. Resource Governor

3. When scaling an Azure SQL Database to a higher tier, which action is most likely to improve storage performance?  

​	A. Enabling Resource Governor  

​	B. Configuring additional filegroups  

​	C. Increasing the service tier  

​	D. Manually redistributing data files

-------------------------

##### Answers

1. Answer: A  
   Performance tiers (DTU/vCore) automatically manage storage performance in Azure SQL Database, eliminating the need for manual filegroup configuration.

2. Answer: B  
   Automatic high availability and backups are built into Azure SQL Database, ensuring storage efficiency without manual intervention.

3. Answer: C  
   Increasing the service tier boosts resource allocation, which improves storage performance in Azure SQL Database.
