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

#### Configuration Settings

-------------------------

##### Server Memory

SQL Server dynamically manages memory between its own processes and the operating system, but you can control its behavior by configuring two key settings:

- Maximum Server Memory:
  This setting controls the upper limit of memory SQL Server can use. It’s critical for preventing SQL Server from consuming so much memory that the operating system and other applications are starved of resources.
- Minimum Server Memory:
  This setting ensures SQL Server reserves a baseline amount of memory. It helps maintain consistent performance by guaranteeing that a certain amount of memory remains allocated even during periods of low activity.

###### SQL Server Management Studio

- Open SQL Server Management Studio and connect to your SQL Server instance
- Right-click the server in Object Explorer, select "Properties", and navigate to the "Memory" page
- Locate and adjust the values for both "Maximum server memory" and "Minimum server memory"

###### Server-Level

Configure using the following T‑SQL:

```sql
-- Set Maximum Server Memory to 6GB (6144 MB)
EXEC sp_configure 'show advanced options', 1;
RECONFIGURE;
EXEC sp_configure 'max server memory', 6144;
RECONFIGURE;

-- Set Minimum Server Memory to 1GB (1024 MB)
EXEC sp_configure 'min server memory', 1024;
RECONFIGURE;
```

##### Query-Level

Instead of changing server-level settings, you can control the memory allocation for a specific query using query hints available in SQL Server 2019 and later. Two useful hints are:

- MIN_MEMORY_GRANT_PERCENT: Specifies the minimum percentage of the computed memory grant to use
- MAX_MEMORY_GRANT_PERCENT: Specifies the maximum percentage of the computed memory grant to use

For example, to force a query to use only 10% of its computed memory grant, you can use:

```sql
SELECT * FROM dbo.LargeTestTable 
OPTION (MIN_MEMORY_GRANT_PERCENT = 10, MAX_MEMORY_GRANT_PERCENT = 10);
```

This query hint limits the memory grant for that specific query to a fixed percentage (10% in this example) of the computed memory grant. Adjust the percentage as needed based on your testing and workload requirements.

-------------------------

##### Index Creation Memory

Specifies the amount of memory allocated for index creation operations. If set too low, index rebuilds and creations may run slower due to insufficient memory.

###### SQL Server Management Studio

- Open SQL Server Management Studio and connect to your SQL Server instance
- Right-click the server in Object Explorer, select "Properties", and navigate to the "Memory" page
- Locate and adjust the value for "Index creation memory (in KB, 0 = dynamic memory)"

###### Server-Level

Configure using the following T‑SQL:

```sql
EXEC sp_configure 'show advanced options', 1;
RECONFIGURE;
EXEC sp_configure 'index creation memory', 2048; -- in KB
RECONFIGURE;
```

###### Query-Level

Not possible — this setting must be configured at the server level.

-------------------------

##### Minimum Memory per Query

Changes not recommended because:

1. It's misleadingly named: Despite its name, the `minimum memory per query` setting doesn't guarantee a fixed memory grant per query. Instead, it sets the *minimum* size of a memory grant request that the system will consider. Most queries request more than this, so the setting is rarely a limiting factor.

2. It’s rarely adjusted: The default value (1024 KB) is almost always appropriate. It's very uncommon to need to raise or lower this unless you're doing deep internals tuning or working in a highly constrained environment.

3. Changing it can harm performance: Raising it too high can cause small queries to be over-allocated memory unnecessarily. Lowering it too much may cause under-allocation, leading to tempdb spills and performance hits.

4. It doesn't apply to all workloads: Many modern memory grant behaviors (especially in SQL Server 2019+) are influenced more by query plan operators, hints, and adaptive memory mechanisms than by this static threshold.

-------------------------

##### Buffer Pool Extension

Allows SQL Server to use fast storage (typically SSDs) as an extension of the buffer pool. This helps improve read performance in memory-constrained environments by offloading less-frequently used pages to disk. However, it is not a replacement for adequate physical memory and is supported only in certain SQL Server editions.

###### SQL Server Management Studio

Not possible — this setting must be configured at the server level.

###### Server-Level

Configure using the following T‑SQL:

```sql
-- Enable buffer pool extension and define file path and size
ALTER SERVER CONFIGURATION
SET BUFFER POOL EXTENSION ON
(FILENAME = 'C:\Temp\BPECache.bpe', SIZE = 4 GB);

-- Disable buffer pool extension
ALTER SERVER CONFIGURATION
SET BUFFER POOL EXTENSION OFF;
```

###### Query-Level

Not possible — this setting must be configured at the server level.

------

##### Optimize for Ad Hoc Workloads

This setting controls whether SQL Server caches full execution plans for single-use queries. When enabled, only a small plan stub is stored for queries executed once, reducing memory usage in the plan cache. If the same query is executed again, a full plan is compiled and cached. This helps reduce memory pressure from ad hoc or dynamic workloads.

###### SQL Server Management Studio

Not possible — this setting must be configured at the server level.

###### Server-Level

Use the following T‑SQL to configure the setting:

```sql
-- View current setting
EXEC sp_configure 'show advanced options', 1;
RECONFIGURE;
EXEC sp_configure 'optimize for ad hoc workloads';

-- Enable the setting
EXEC sp_configure 'optimize for ad hoc workloads', 1;
RECONFIGURE;
```

- `0` = Disabled
- `1` = Enabled (recommended for systems with many unique ad hoc queries)

###### Query-Level

Not possible — this setting must be configured at the server level.

------

##### Resource Governor

> Note: This section applies only to Enterprise edition

Resource Governor allows you to control how SQL Server allocates memory to specific workloads by defining resource pools and workload groups. This helps ensure that high-priority workloads receive guaranteed resources, while limiting memory consumption from less critical or unpredictable sessions.

###### SQL Server Management Studio

- Open SQL Server Management Studio and connect to your SQL Server instance
- Expand "Management" in Object Explorer, right-click "Resource Governor", and select "Properties"
- In the "Resource Governor Properties: dialog:
  - Check "Enable Resource Governor"
  - "Resource pools": Configure the `Minimum memory %` and `Maximum memory %` for each pool; these define the guaranteed and capped percentage of overall SQL Server memory assigned to that pool
  - "Workload groups...": Optionally set `Memory Grant %` and other workload characteristics for sessions tied to that pool
  - "External resource pools": Apply only to R, Python, or other external scripts run via SQL Server Machine Learning Services; they do not affect regular T-SQL workloads
  - Click "OK" to save changes and then right-click Resource Governor and select "Reconfigure" to apply the new settings

###### Server-Level

Configure using the following T‑SQL:

```sql
-- Create a new resource pool with memory limits
CREATE RESOURCE POOL PoolReporting
WITH (
    MIN_MEMORY_PERCENT = 10,
    MAX_MEMORY_PERCENT = 30
);

-- Create a workload group that uses this pool
CREATE WORKLOAD GROUP GroupReporting
USING PoolReporting;

-- Apply configuration
ALTER RESOURCE GOVERNOR RECONFIGURE;
```

To modify an existing resource pool:

```sql
ALTER RESOURCE POOL PoolReporting
WITH (
    MAX_MEMORY_PERCENT = 40
);
ALTER RESOURCE GOVERNOR RECONFIGURE;
```

###### Query-Level

Not possible — this setting must be configured at the server level.

------

##### Lock Pages in Memory (Windows Policy Setting)

The Lock Pages in Memory privilege prevents SQL Server memory from being paged out to disk by the operating system, helping maintain consistent performance under memory pressure. This is especially beneficial for systems with high memory utilization or predictable memory workloads.

> Note: This setting is configured at the Windows OS level and not through SQL Server Management Studio or T‑SQL.

###### Operating System Configuration

1. Grant the Privilege

   - Open the Local Security Policy console (`secpol.msc`)
   - Navigate to: `Local Policies` → `User Rights Assignment`
   - Locate Lock pages in memory
   - Add the SQL Server service account to the policy

2. Restart the SQL Server Service

   - Changes take effect only after restarting the SQL Server service

3. Verification

   - Use Windows Resource Monitor or sys.dm_os_process_memory to confirm that paging is minimized and SQL Server memory is retained in physical RAM:

     ```sql
     SELECT sql_memory_model_desc
     FROM sys.dm_os_sys_info;
     ```

     - If the setting is active, `sql_memory_model_desc` will return: `LOCK_PAGES`
     - If not, it will return: `CONVENTIONAL` or `LARGE_PAGES` (if that feature is used)

###### SQL Server Management Studio

Not possible — this is a Windows security setting and cannot be configured within SQL Server Management Studio.

###### Server-Level

Not possible — this setting cannot be changed via SQL Server commands or configuration.

###### Query-Level

Not possible — memory locking is a system-level privilege and cannot be controlled or scoped to individual queries.

------------------------- -------------------------

#### Exercise

Follow these step-by-step instructions to demonstrate how adjusting the Maximum Server Memory setting affects query performance and memory usage on an on-prem SQL Server.

##### Prepare Sample Data

- Open SQL Server Management Studio and connect to your SQL Server instance

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

##### Decrease Maximum Server Memory

- Execute the following T‑SQL commands to set Maximum Server Memory to 512 MB:

  ```sql
  EXEC sp_configure 'show advanced options', 1;
  RECONFIGURE;
  EXEC sp_configure 'max server memory', 512;
  RECONFIGURE;
  ```

  >  Note: SQL Server service restart not required

##### Stress Memory

- Click "Include Actual Execution Plan" and execute the following T-SQL:

  ```sql
  SELECT * FROM dbo.LargeTestTable ORDER BY DataValue;
  ```

- Note execution time and review execution plan


##### Increase Maximum Server Memory

- Execute the following T‑SQL commands to set Maximum Server Memory to 2,147,483,647 MB:

  ```sql
  EXEC sp_configure 'show advanced options', 1;
  RECONFIGURE;
  EXEC sp_configure 'max server memory', 2147483647;
  RECONFIGURE;
  ```

  _Note: SQL Server service restart not required_

##### Re-Stress Memory

- Execute the following T‑SQL:

  ```sql
  SELECT * FROM dbo.LargeTestTable ORDER BY DataValue;
  ```

- Compare execution time and execution plan

###### Review Statistics

- Execute the following T‑SQL:

  ```sql
  SELECT TOP 1 qt.query_sql_text, r.max_query_max_used_memory, r.last_execution_time
  FROM sys.query_store_query_text AS qt
  JOIN sys.query_store_query AS q ON qt.query_text_id = q.query_text_id
  JOIN sys.query_store_plan AS p ON q.query_id = p.query_id
  JOIN sys.query_store_runtime_stats AS r ON p.plan_id = r.plan_id
  WHERE qt.query_sql_text LIKE '%SELECT * FROM dbo.LargeTestTable ORDER BY DataValue%'
  ORDER BY r.last_execution_time DESC;
  ```

  > Note: System tables used in the query above (and more) are detailed in the Appendix

-------------------------

#### Recap

When SQL Server is installed by default, it’s set to use as much memory as it can because it’s designed to run on dedicated servers. However, on a VM with 8GB of memory, you need to leave enough memory for the operating system and any other applications. Here are some points and guidelines:

- Default Setting: By default, SQL Server’s "max server memory" is set very high (2,147,483,647 MB) to allow it to use as much available memory as possible on a dedicated machine. This isn’t ideal for a VM where the OS also needs memory.

- Rule of Thumb: A common approach is to reserve about 20–25% of total memory for the OS and other background processes. On an 8GB VM, that might mean allocating around 6GB for SQL Server (i.e. 75% of 8GB).

- Calculations:  

  - Total Memory: 8GB  
  - Reserved for OS and other services: ~2GB (this can vary based on what else is running)  
  - Memory for SQL Server: 8GB − 2GB = 6GB  
    This 6GB is a starting point; you may adjust based on your workload.

- Absolute Minimum: While SQL Server can run on very low memory (1GB or even less for small, light workloads), settings as low as 32MB are far below what is necessary for even minimal functionality. For production or even testing environments, at least 1–2GB is usually needed, though more is recommended for better performance.

- Practical Steps:  

  - Monitor your current usage: Use Performance Monitor or DMVs (like `sys.dm_os_memory_clerks`) to see how much memory SQL Server is actually using.  

  - Gradual adjustment: Instead of drastically lowering the memory, adjust it incrementally while observing performance and stability.  

  - Test different settings: In your case, you might try setting "Maximum server memory" to 6GB and see how the system behaves compared to higher or lower settings.

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
   ‘Minimum memory per query’ defines a floor for each query’s memory grant, providing a small baseline allocation even under heavy workloads

------------------------- ------------------------- -------------------------

### Azure

#### Configuration Settings

Azure SQL Database is a fully managed platform-as-a-service (PaaS) offering. As such, server-level and query-level memory configurations are not supported. You cannot modify memory-related settings using `sp_configure`, apply Resource Governor policies, or use memory grant hints. All memory management is handled internally by the Azure platform, and tuning is done by adjusting the service tier.

-------------------------

##### Service Tier Selection

Determines memory allocation by automatically managing resources based on the selected performance tier. Higher tiers provide more memory, which can improve query performance.

###### Azure Portal

- Open the Azure Portal and navigate to your Azure SQL Database resource  
- Select "Configure" under Compute + Storage  
- Review the current service tier (vCore or DTU), which determines available memory, CPU, and I/O resources

-------------------------

##### Monitoring Memory with Query Performance Insight

Query Performance Insight provides visibility into query behavior. While direct memory metrics are not exposed, query duration and CPU time can help infer memory limitations.

###### Azure Portal

- Navigate to your Azure SQL Database resource  
- Select "Query Performance Insight"  
- Review top queries, execution count, CPU usage, and durations  
- Identify patterns that suggest memory-related performance constraints

-------------------------

##### Scaling Up for Increased Memory Allocation

The primary way to increase memory capacity is by scaling the database to a higher performance tier.

###### Azure Portal

- Navigate to your Azure SQL Database resource  
- Select "Configure" under Compute + Storage  
- Choose a higher tier (e.g., more vCores or DTUs)  
- Save the configuration to apply the change  
- Monitor the performance impact using Query Performance Insight

-------------------------

##### Automatic Memory Management

Azure SQL Database automatically manages memory based on service tier and workload patterns. There is no manual tuning or configuration required.

###### Azure Portal

- Navigate to the "Overview" or "Metrics" blade for your Azure SQL Database  
- Review indicators such as DTU usage, CPU percentage, and IO metrics  
- Use these to evaluate memory-related behavior indirectly under your current tier

------------------------- -------------------------

#### Exercise

In Azure SQL Database, it's not possible to directly configure memory settings like you can on-prem (e.g., "Maximum Server Memory"). Azure manages memory automatically based on the chosen service tier and compute resources (vCores/DTUs).

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

#### Configuration Settings

-------------------------

##### Maximum Degree of Parallelism (MAXDOP)

MAXDOP controls the number of processors SQL Server uses to execute a single query in parallel. Tuning this setting can improve performance and reduce contention—especially in OLTP environments or systems with many concurrent queries.

###### SQL Server Management Studio

- Open SQL Server Management Studio and connect to your SQL Server instance
- Right-click the server in Object Explorer, select "Properties", and navigate to the "Advanced" page
- Adjust the setting based on your server’s workload characteristics:
  - Set MAXDOP = 1 for OLTP workloads to reduce contention and prevent parallel plan overhead
  - Set MAXDOP = 0 to allow SQL Server to use all available CPUs for large analytical queries (default behavior)
  - For mixed workloads, consider values like 2, 4, or 8 depending on the number of logical processors and concurrency needs
  - As a general rule:
    - Use 1 for highly concurrent transactional systems
    - Use (number of cores per NUMA node) or ≤ 8 for most general-purpose workloads
    - Avoid setting MAXDOP to a value higher than the number of available logical processors
    - For large servers with many cores, also consider combining this with the Cost Threshold for Parallelism setting to control which queries are eligible for parallel execution.

###### Server-Level

Configure using the following T‑SQL:

```sql
EXEC sp_configure 'show advanced options', 1;
RECONFIGURE;
EXEC sp_configure 'max degree of parallelism', 4;
RECONFIGURE;
```

###### Query-Level

Override MAXDOP on a per-query basis using the `OPTION (MAXDOP n)` query hint:

```sql
SELECT * FROM dbo.LargeCPUTable OPTION (MAXDOP 1);  -- run this query on a single CPU
```

This is helpful when:

- You want to avoid parallelism overhead for simple queries
- You’re running heavy batch queries that would otherwise consume excessive resources
- You want to isolate workloads in mixed environments

-------------------------

##### Cost Threshold for Parallelism

Controls the minimum estimated query cost (in arbitrary units) that SQL Server uses to decide whether a query should be executed using parallelism. A lower value may allow even simple queries to run in parallel, increasing CPU usage. A higher value restricts parallelism to only more expensive queries, reducing CPU pressure on busy systems.

> This setting works in conjunction with Maximum Degree of Parallelism (MAXDOP) to control parallel query behavior.

###### SQL Server Management Studio

- Open SQL Server Management Studio and connect to your SQL Server instance
- Right-click the server in Object Explorer, select "Properties", and navigate to the "Advanced" page
- Locate "Cost Threshold for Parallelism" under the Parallelism section
- Adjust the setting based on your server’s workload characteristics:
  - Default: `5` (often too low for most production workloads)
  - Common values: `25`, `50`, or higher to restrict parallelism to truly expensive queries

###### Server-Level

Use the following T‑SQL to adjust the setting programmatically:

```sql
EXEC sp_configure 'show advanced options', 1;
RECONFIGURE;
EXEC sp_configure 'cost threshold for parallelism', 50;  -- example: raise from default 5 to 50
RECONFIGURE;
```

###### Query-Level

Not possible — this setting must be configured at the server level.

-------------------------

##### Processor Affinity

Processor Affinity determines which specific CPUs SQL Server is allowed to use. By default, SQL Server dynamically schedules tasks across all available CPUs, which is optimal in most scenarios. Manual processor affinity settings are advanced and typically used only in constrained or highly specialized environments.

###### SQL Server Management Studio

- Open SQL Server Management Studio and connect to your SQL Server instance
- Right-click the server in Object Explorer, select "Properties", and navigate to the "Processors" page
- Adjust the setting based on your server’s workload characteristics:
  - Automatically set processor affinity mask for all processors
    - Checked by default — allows SQL Server to dynamically schedule across all available CPUs
    - Leave this checked unless you need to isolate SQL Server to specific cores (e.g., for licensing or performance reasons)
    - Uncheck to manually assign processor affinity in the table below
  - Automatically set I/O affinity mask for all processors
    - Typically left checked — controls which CPUs handle disk I/O operations
    - Change only in advanced tuning scenarios
  - Processor Affinity (table below)
    - When the "Automatically set" box is unchecked, you can manually assign which CPUs SQL Server can use
    - Select specific processors by checking boxes in the Processor Affinity column
    - Avoid assigning too few cores or splitting across NUMA nodes unless you have a specific, tested reason
    - Never leave all boxes unchecked — SQL Server will fail to start without any processor assigned
  - Maximum worker threads
    - Leave at `0` to let SQL Server manage threading automatically
    - Override only for specialized tuning (e.g., known under-subscription of CPU resources)

###### Server-Level

Processor affinity can be configured using T‑SQL, but only for process affinity — not I/O affinity.

```sql
-- Assign CPUs 0 through 3 for SQL Server processing
ALTER SERVER CONFIGURATION SET PROCESS AFFINITY CPU = 0, 1, 2, 3;

-- To reset to automatic processor affinity
ALTER SERVER CONFIGURATION SET PROCESS AFFINITY CPU = AUTO;
```

> Note: SQL Server does not support setting I/O affinity using T‑SQL. To configure I/O affinity, use SQL Server Management Studio under the Processors page.

> Caution: Misconfiguring processor affinity may reduce CPU availability for SQL Server or cause uneven scheduling. Always test changes in a non-production environment.

###### Query-Level

Not possible — this setting must be configured at the server level.

-------------------------

##### Resource Governor

> Note: This section applies only to Enterprise edition

Resource Governor allows you to control how SQL Server allocates CPU to specific workloads by defining resource pools and workload groups. This helps ensure that high-priority workloads receive guaranteed processor access, while limiting CPU consumption from less critical or unpredictable sessions.

###### SQL Server Management Studio

- Open SQL Server Management Studio and connect to your SQL Server instance
- Expand "Management" in Object Explorer, right-click "Resource Governor", and select "Properties"
- In the "Resource Governor Properties" dialog:
  - Check "Enable Resource Governor"
  - Resource pools: Configure the `Minimum CPU %` and `Maximum CPU %` for each pool; these define the guaranteed and capped percentage of overall CPU resources assigned to that pool
  - Workload groups...: Optionally adjust other workload characteristics for sessions tied to that pool
  - External resource pools: Apply only to R, Python, or other external scripts run via SQL Server Machine Learning Services; they do not affect regular T-SQL workloads
  - Click "OK" to save changes and then right-click Resource Governor and select "Reconfigure" to apply the new settings

###### Server-Level

Configure using the following T‑SQL:

```sql
-- Create a new resource pool with CPU limits
CREATE RESOURCE POOL PoolReporting
WITH (
    MIN_CPU_PERCENT = 10,
    MAX_CPU_PERCENT = 30
);

-- Create a workload group that uses this pool
CREATE WORKLOAD GROUP GroupReporting
USING PoolReporting;

-- Apply configuration
ALTER RESOURCE GOVERNOR RECONFIGURE;
```

To modify an existing resource pool:

```sql
ALTER RESOURCE POOL PoolReporting
WITH (
    MAX_CPU_PERCENT = 40
);
ALTER RESOURCE GOVERNOR RECONFIGURE;
```

###### Query-Level

Not possible — this setting must be configured at the server level.

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

  - show advanced options:
    - Possible values: 0 (disabled) or 1 (enabled)
  - max degree of parallelism:
    - Possible values: 0 (use all available CPUs) or any positive integer (typically 1 up to the number of logical processors, with an upper bound of 32767)
  - cost threshold for parallelism:
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

#### Configuration Settings

Azure SQL Database is a fully managed platform-as-a-service (PaaS) offering. As such, server-level and query-level CPU configurations are not supported. You cannot modify CPU-related settings using `sp_configure`, set processor affinity, or apply Resource Governor policies. All CPU allocation is handled internally by the Azure platform and scales automatically based on the selected service tier.

------

##### Service Tier Selection

CPU resources are provisioned according to the selected performance tier (DTU or vCore model). Higher tiers allocate more vCores, improving throughput for CPU-intensive workloads.

###### Azure Portal

- Open the Azure Portal and navigate to your Azure SQL Database resource
- Select "Configure" under Compute + Storage
- Review the current service tier and its associated vCore or DTU allocation

------

##### Monitoring CPU Usage

Monitoring CPU usage helps identify under-provisioned workloads and validate whether the selected tier provides sufficient compute resources.

###### Azure Portal

- Navigate to your Azure SQL Database resource
- Open the "Metrics" blade or Query Performance Insight
- Select CPU Percentage or other relevant metrics to observe trends over time
- Analyze high-CPU queries in Query Performance Insight to correlate usage with specific workloads

------

##### Scaling Up for Improved CPU Performance

The primary way to increase CPU resources is by scaling the database to a higher service tier.

###### Azure Portal

- Navigate to your Azure SQL Database resource
- Select "Configure" under Compute + Storage
- Choose a higher tier (e.g., more vCores or DTUs)
- Apply the change and monitor performance improvements using Metrics or Query Performance Insight

------

##### Query Optimization

Although CPU settings cannot be manually adjusted, CPU usage can be reduced through proper query tuning and indexing strategies.

###### Azure Portal

- Use Query Performance Insight to identify high-CPU queries
- Review execution plans and identify missing indexes, expensive operations, or inefficient joins
- Optimize the query and re-measure performance using execution duration and CPU usage metrics

------------------------- -------------------------

#### Exercise

Follow these step-by-step instructions using the Query Editor in the Azure Portal to demonstrate how scaling an Azure SQL Database can improve performance for CPU‑intensive queries. Note that while Azure SQL Database manages memory automatically, scaling up increases both CPU resources and available memory.

##### Connect

- In the Azure portal, navigate to your existing SQL Database resource.
- Click on Query Editor (preview) from the left-hand menu.
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

- Check Service Tier and Edition
  This query displays the current service tier and edition for your database:

  ```sql
  SELECT 
      DATABASEPROPERTYEX(DB_NAME(), 'Edition') AS Edition,
      DATABASEPROPERTYEX(DB_NAME(), 'ServiceObjective') AS ServiceObjective;
  ```

  These values tell you the selected tier (for example, Standard S0, S2, or S3) and provide an indication of the CPU and memory resources available based on your chosen tier.

- Review System Information
  Although Azure SQL Database is a managed service and doesn’t expose detailed hardware specs, you can get approximate resource details by running:

  ```sql
  SELECT * FROM sys.dm_os_sys_info;
  ```

  Note values:

  - cpu_ticks: Cumulative CPU ticks since SQL Server started
  - ms_ticks: Cumulative time in milliseconds since SQL Server started
  - cpu_count: Number of logical CPUs available
  - hyperthread_ratio: Ratio indicating the hyper-threading multiplier

##### Baseline Performance

```sql
SELECT COUNT_BIG(*)
FROM dbo.CPU_Test t1 JOIN dbo.CPU_Test t2 ON t1.ID <> t2.ID
WHERE t1.ID % 2 = 0 AND t2.ID % 3 = 0;
```

Note time elapsed.

##### Scale Database

Navigate to SQL >> Settings >> Compute + Storage and adjust settings:

1. Max vCores
   - Defines the upper limit of CPU resources that your database can use
   - During periods of high workload, the database can scale up to this maximum, potentially reducing query times and improving overall performance
   - Increasing max vCores is beneficial if you anticipate or experience spikes in CPU demand
2. Min vCores
   - Defines the baseline amount of CPU resources that are always allocated to your database
   - A higher min vCores ensures more consistent performance because you avoid waiting for the service to “warm up” or scale out when queries arrive
   - However, a higher min vCores also increases the baseline cost, as you pay for these allocated resources even when your database is idle

###### Deciding How to Adjust Min and Max vCores

- Performance Sensitivity: If you have critical workloads that need consistently fast response times, you might raise the min vCores so the database doesn’t have to ramp up from near zero.
- Cost Optimization: If cost is a major concern and your workload is intermittent, you can keep min vCores lower, allowing the database to scale down during idle times, but expect some performance delay when the load first ramps up.
- Handling Peak Loads: If you occasionally run CPU‑intensive queries (such as cross joins or large aggregations), raising the max vCores ensures your database can scale up to handle those spikes without throttling.

###### Practical Steps

1. Monitor Current Usage
   - Use Azure Metrics or Query Performance Insight to see your current CPU usage patterns
   - Identify whether you’re regularly hitting your max vCores or if performance suffers during spikes
2. Adjust Gradually
   - Increase max vCores by one tier at a time and observe query performance improvements
   - If you have frequent workloads that need consistent performance, raise min vCores to avoid scale‑up delays
3. Test Your Workload
   - Run your most demanding queries (like the cross join examples) to see if performance meets expectations
   - If queries still take too long or you see throttling, consider further increasing max vCores or raising min vCores

By fine‑tuning min vCores and max vCores within the Serverless model, you can strike the right balance between cost savings and performance for your specific workload.

##### Compare Performance

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

#### Configuration Settings

-------------------------

##### Filegroups

Filegroups are logical groupings of database files that allow data to be spread across multiple physical disks. This can improve I/O performance, simplify maintenance, and isolate workloads. While the default filegroup is typically sufficient, custom filegroups provide greater flexibility for performance tuning and file placement strategies.

###### SQL Server Management Studio

- Open SQL Server Management Studio and connect to your SQL Server instance
- Expand Databases, right-click a specific database, and select "Properties"
- Navigate to the "Files" page to view the list of filegroups and associated files
- To add or modify filegroups:
  - Go to the "Filegroups" page (visible when editing database properties)
  - Define a new filegroup and associate one or more data files with it via the "Files" page

###### Server-Level

Not applicable — filegroups are defined at the database level, not at the server level.

###### Query-Level

Not possible — filegroup configuration cannot be controlled at the query level.

-------------------------

##### Storage Pools and Striping

Storage striping improves I/O throughput by distributing database files across multiple physical disks. This reduces contention and enhances performance in high-read/write scenarios. On Windows, this can be implemented using storage spaces or RAID configurations.

###### SQL Server Management Studio

- Open SSMS and begin creating a new database
- In the "New Database" dialog, navigate to the "Files" page
- Add multiple data files and assign each one to a different drive or volume (e.g., D:, E:, F:)
- After creation, view the file distribution under Database Properties > Files

###### Server-Level

Not applicable — striping is implemented through the operating system or storage hardware, not SQL Server configuration.

###### Query-Level

Not possible — striping behavior is transparent to query execution and cannot be adjusted at the query level.

-------------------------

##### Manual File Placement

Manual file placement is the practice of assigning data files and transaction log files to different physical disks to reduce I/O contention. Separating write-heavy log operations from data access improves concurrency and durability.

###### SQL Server Management Studio

- During new database creation in SSMS, click on the "Files" or "Options" tab
- Modify the file path for the data file (typically .mdf) to one physical drive
- Assign the log file (typically .ldf) to a different drive
- After creation, review these paths under Database Properties > Files

###### Server-Level

Not applicable — file placement is handled per database and must be configured individually.

###### Query-Level

Not possible — file placement is not accessible or changeable from within query execution.

------------------------- -------------------------

#### Exercise

Follow these step-by-step instructions to demonstrate how distributing data files across multiple disks affects I/O throughput and query performance on an on-prem SQL Server.

##### Add Disks

- In Azure Portal, navigate to your virtual machine → Settings → Disks
  - Click `+ Create and attach a new disk` twice (e.g., `SQLDisk1` and `SQLDisk2`), then click `"Apply"`
  - Restart VM and then connect
- On the VM, open `diskmgmt.msc`… when prompted, initialize new disks with partition style "GPT"
  - The disks will now show as "Unallocated"… right-click unallocated space and select "New Simple Volume" from the resulting menu
  - Step through the New Simple Volume Wizard for each
- Open File Explorer and verify that the new drives are visible and ready to use
- Manually create folder `SQLData` on both of the new drives (`E:\SQLData` and `I:\SQLData`)

##### Single-File

###### Create Database

```sql
DECLARE @Drive1 VARCHAR(2) = 'E:';

DECLARE @sql NVARCHAR(MAX) = '
CREATE DATABASE trainingdb_singlefile
ON PRIMARY (
    NAME = trainingdb_singlefile_Data,
    FILENAME = ''' + @Drive1 + '\SQLData\trainingdb_singlefile_Data.mdf''
)
LOG ON (
    NAME = trainingdb_singlefile_Log,
    FILENAME = ''' + @Drive1 + '\SQLData\trainingdb_singlefile_Log.ldf''
);';

EXEC sp_executesql @sql;
```

###### Populate Table

```sql
USE trainingdb_singlefile;

IF OBJECT_ID('dbo.IOTestTable') IS NOT NULL DROP TABLE dbo.IOTestTable;

CREATE TABLE dbo.IOTestTable (
    ID INT IDENTITY(1,1) PRIMARY KEY,
    DataValue VARCHAR(100)
);

WITH X AS (
    SELECT TOP (5000) ROW_NUMBER() OVER (ORDER BY (SELECT NULL)) AS n
    FROM sys.all_objects a CROSS JOIN sys.all_objects b
)
INSERT INTO dbo.IOTestTable (DataValue)
SELECT REPLICATE('X', 100)
FROM X;
```

###### Optimize Settings

```sql
-- Enable advanced options to allow configuration changes
EXEC sp_configure 'show advanced options', 1;
RECONFIGURE;

-- Force SQL Server to execute queries on a single CPU core (disable parallelism)
EXEC sp_configure 'max degree of parallelism', 1;  
RECONFIGURE;

-- Prevent queries from running in parallel by increasing the cost threshold
EXEC sp_configure 'cost threshold for parallelism', 100;  
RECONFIGURE;

-- Reduce SQL Server's available memory to force more physical disk reads
EXEC sp_configure 'max server memory', 2048;  -- Adjust as needed
RECONFIGURE;

-- Restrict SQL Server I/O processing to a single CPU core
EXEC sp_configure 'affinity I/O mask', 1;  
RECONFIGURE;

-- Keep all indexes in the PRIMARY filegroup to prevent distributed disk access
IF NOT EXISTS (
    SELECT * FROM sys.indexes WHERE name = 'IX_IOTestTable_DataValue' 
    AND object_id = OBJECT_ID('dbo.IOTestTable')
)
BEGIN
    CREATE NONCLUSTERED INDEX IX_IOTestTable_DataValue 
    ON dbo.IOTestTable (DataValue);
END
```

###### Execute Query

```sql
SELECT COUNT_BIG(*), AVG(CAST(LEN(t1.DataValue) + LEN(t2.DataValue) AS BIGINT))
FROM dbo.IOTestTable AS t1 CROSS JOIN dbo.IOTestTable AS t2
WHERE LEFT(t1.DataValue, 3) = LEFT(t2.DataValue, 3) AND RIGHT(t1.DataValue, 3) = RIGHT(t2.DataValue, 3);
```

Note execution time and plan.

-------------------------

##### Multi-File

###### Create Database

```sql
DECLARE @Drive1 VARCHAR(2) = 'E:';
DECLARE @Drive2 VARCHAR(2) = 'I:';

DECLARE @sql NVARCHAR(MAX) = '
CREATE DATABASE trainingdb_multifile
ON PRIMARY (
    NAME = trainingdb_multifile_Data1,
    FILENAME = ''' + @Drive1 + '\SQLData\trainingdb_multifile_Data1.mdf''
),
FILEGROUP FG2 (
    NAME = trainingdb_multifile_Data2,
    FILENAME = ''' + @Drive2 + '\SQLData\trainingdb_multifile_Data2.ndf''
)
LOG ON (
    NAME = trainingdb_multifile_Log,
    FILENAME = ''' + @Drive1 + '\SQLData\trainingdb_multifile_Log.ldf''
);';

EXEC sp_executesql @sql;
```

###### Populate Table

```sql
USE trainingdb_multifile;

IF OBJECT_ID('dbo.IOTestTable') IS NOT NULL DROP TABLE dbo.IOTestTable;

-- Create the table on the PRIMARY filegroup (drive E:)
CREATE TABLE dbo.IOTestTable (
    ID INT IDENTITY(1,1) PRIMARY KEY,
    DataValue VARCHAR(100)
) ON [PRIMARY];
```

###### Optimize Settings

```sql
-- Enable advanced options to allow configuration changes
EXEC sp_configure 'show advanced options', 1;
RECONFIGURE;

-- Allow SQL Server to use all available CPUs for parallel processing
EXEC sp_configure 'max degree of parallelism', 0;  
RECONFIGURE;

-- Lower the cost threshold to encourage more queries to run in parallel
EXEC sp_configure 'cost threshold for parallelism', 5;  
RECONFIGURE;

-- Enable Buffer Pool Extension to improve read-ahead optimization
-- Ensure the file path exists before running this command
ALTER SERVER CONFIGURATION SET BUFFER POOL EXTENSION ON (FILENAME = 'E:\SQLData\BPE.bpe', SIZE = 2GB);

-- Create a nonclustered index on FG2 (drive I:) to distribute I/O across both disks
IF NOT EXISTS (
    SELECT * FROM sys.indexes WHERE name = 'IX_IOTestTable_DataValue' 
    AND object_id = OBJECT_ID('dbo.IOTestTable')
)
BEGIN
    CREATE NONCLUSTERED INDEX IX_IOTestTable_DataValue 
    ON dbo.IOTestTable (DataValue) ON FG2;
END

-- Configure SQL Server to use multiple CPU cores for disk I/O processing
EXEC sp_configure 'affinity I/O mask', 3;  
RECONFIGURE;
```

###### Execute Query

```sql
SELECT COUNT_BIG(*), AVG(CAST(LEN(t1.DataValue) + LEN(t2.DataValue) AS BIGINT))
FROM dbo.IOTestTable AS t1 CROSS JOIN dbo.IOTestTable AS t2
WHERE LEFT(t1.DataValue, 3) = LEFT(t2.DataValue, 3) AND RIGHT(t1.DataValue, 3) = RIGHT(t2.DataValue, 3);
```

Compare execution times and plans.

------------------------- -------------------------

##### Recap

| Metric               | TestDB_SingleFile           | TestDB_MultiFile           |
| -------------------- | --------------------------- | -------------------------- |
| Execution Plan       | Fewer parallelism operators | More parallelism operators |
| Logical Reads        | Higher                      | Lower                      |
| Physical Reads       | Higher                      | Lower                      |
| Disk Contention      | Higher                      | Lower                      |
| Query Execution Time | Longer                      | Shorter                    |

------------------------- -------------------------

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

Follow these step-by-step instructions to demonstrate how scaling an Azure SQL Database affects storage performance by comparing query execution times before and after scaling.

##### Connect

- In the Azure portal, navigate to your existing SQL Database resource.
- Click on Query Editor (preview) from the left-hand menu.
- Log in using your database credentials.

##### Prepare Sample Data

```sql
IF OBJECT_ID('dbo.IOTestTable') IS NOT NULL DROP TABLE dbo.IOTestTable;

CREATE TABLE dbo.IOTestTable (
    ID INT IDENTITY(1,1) PRIMARY KEY,
    DataValue VARCHAR(100)
);

WITH X AS (
    SELECT TOP (5000) ROW_NUMBER() OVER (ORDER BY (SELECT NULL)) AS n
    FROM sys.all_objects a CROSS JOIN sys.all_objects b
)
INSERT INTO dbo.IOTestTable (DataValue)
SELECT REPLICATE('X', 100)
FROM X;
```

------

##### Baseline Performance

```sql
SELECT COUNT_BIG(*), AVG(CAST(LEN(t1.DataValue) + LEN(t2.DataValue) AS BIGINT))
FROM dbo.IOTestTable AS t1 CROSS JOIN dbo.IOTestTable AS t2
WHERE LEFT(t1.DataValue, 3) = LEFT(t2.DataValue, 3) AND RIGHT(t1.DataValue, 3) = RIGHT(t2.DataValue, 3);
```

Note time elapsed.

------

##### Scale Database

Navigate to SQL Database → Settings → Compute + Storage in the Azure portal and adjust the storage tier for your serverless General Purpose Azure SQL Database.

------

###### Understanding Storage Scaling in Azure SQL Database

1. Storage Size
   - Defines the total amount of storage allocated to the database.
   - Increasing storage can improve performance by reducing contention for space and I/O operations.
   - More storage allows larger workloads and prevents slowdowns caused by running out of available space.
2. IOPS (Input/Output Operations per Second)
   - Increases automatically as storage size grows.
   - Higher IOPS means faster query performance for read/write-heavy workloads.
   - Scaling up reduces latency for transactions that involve frequent disk access.

------

###### Choosing the Right Storage Settings

- Performance Sensitivity → If queries involve large data reads/writes, increasing storage size helps improve disk I/O speed.
- Cost Optimization → If storage costs are a concern, scale only when metrics show high I/O latency or slow response times due to disk contention.
- Handling Growth → If database size is near its limit, increase storage before hitting capacity to avoid performance degradation.

------

###### Scaling Steps

1. Monitor Storage Usage
   - In the Azure portal, go to Metrics → Storage Consumption to check current storage usage.
   - Review IOPS and Log Write Performance to see if storage is causing bottlenecks.
2. Increase Storage Capacity
   - Under Compute + Storage, increase the allocated storage size.
   - Storage scaling in General Purpose (serverless) mode is online and does not require downtime.
3. Test Query Performance
   - After scaling, re-run the stress query and compare execution times.
   - Check if DTU consumption and IOPS have improved, indicating better disk performance.

------

##### Compare Performance

```sql
SELECT COUNT_BIG(*), AVG(CAST(LEN(t1.DataValue) + LEN(t2.DataValue) AS BIGINT))
FROM dbo.IOTestTable AS t1 
CROSS JOIN dbo.IOTestTable AS t2
WHERE LEFT(t1.DataValue, 3) = LEFT(t2.DataValue, 3) 
  AND RIGHT(t1.DataValue, 3) = RIGHT(t2.DataValue, 3);
```

Compare time elapsed with previous run.

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

3. Which action is most likely to improve storage performance in an Azure SQL Database (General Purpose serverless)?

​	A. Enabling Resource Governor
​	B. Configuring additional filegroups
​	C. Increasing Data Max Size
​	D. Manually redistributing data files

-------------------------

##### Answers

1. Answer: A  
   Performance tiers (DTU/vCore) automatically manage storage performance in Azure SQL Database, eliminating the need for manual filegroup configuration.

2. Answer: B  
   Automatic high availability and backups are built into Azure SQL Database, ensuring storage efficiency without manual intervention.

3. Answer: C  
   Increasing Data Max Size improves storage performance by increasing IOPS (Input/Output Operations per Second) and log throughput, which directly impacts database read/write performance in General Purpose (serverless).


------------------------- ------------------------- ------------------------- -------------------------

## Appendix

### sys.query_store_query

| Column                      | Description                                                  | Related to Memory Optimization |
| :-------------------------- | :----------------------------------------------------------- | :----------------------------- |
| query_id                    | Unique identifier for the query in Query Store               |                                |
| query_text_id               | References the query text stored in sys.query_store_query_text |                                |
| query_hash                  | Hash value for the query text                                |                                |
| query_plan_hash             | Hash value for the query plan                                |                                |
| last_execution_time         | Timestamp of the most recent execution of this query         |                                |
| query_parameterization_type | Indicates how the query was parameterized (e.g., forced parameterization) |                                |
| is_internal_query           | Indicates if the query is an internal SQL Server query       |                                |
| object_id                   | References the object (e.g., a stored procedure) if applicable |                                |

### sys.query_store_query_text

| Column               | Description                                                  | Related to Memory Optimization |
| :------------------- | :----------------------------------------------------------- | :----------------------------- |
| query_text_id        | Unique identifier for the text row in Query Store            |                                |
| query_sql_text       | Actual text of the query                                     |                                |
| is_internal_query    | Indicates if the query text belongs to an internal SQL Server query |                                |
| statement_sql_handle | Identifies the statement text within SQL Server              |                                |

### sys.query_store_plan

| Column                            | Description                                                  | Related to Memory Optimization |
| :-------------------------------- | :----------------------------------------------------------- | :----------------------------- |
| plan_id                           | Unique identifier for the plan in Query Store                |                                |
| query_id                          | References the query in sys.query_store_query                |                                |
| engine_version                    | Indicates the version of the SQL Server engine               |                                |
| last_execution_time               | Timestamp of the most recent execution for this plan         |                                |
| plan_handle                       | Handle for the query plan                                    |                                |
| query_plan_hash                   | Hash value for the query plan                                |                                |
| query_plan                        | XML representation of the execution plan                     |                                |
| forcing_status                    | Indicates if plan forcing is enabled, disabled, or not applicable |                                |
| forcing_policy                    | Additional detail on the plan forcing policy (if used)       |                                |
| is_optimized_plan_forcing_enabled | Shows whether plan forcing is optimized for this plan        |                                |
| compatibility_level               | Database compatibility level at the time the plan was compiled |                                |

### sys.query_store_runtime_stats

| Column                      | Description                                                  | Related to Memory Optimization |
| :-------------------------- | :----------------------------------------------------------- | :----------------------------- |
| runtime_stats_id            | Unique identifier for this row of runtime stats              |                                |
| plan_id                     | References sys.query_store_plan                              |                                |
| interval_id                 | References sys.query_store_interval, which groups runtime stats by time intervals |                                |
| execution_type              | Numeric code indicating the execution type (e.g., regular, forced plan, etc.) |                                |
| execution_type_desc         | Descriptive text for execution_type                          |                                |
| count_executions            | Number of times the plan was executed in the given interval  |                                |
| sum_duration                | Total duration (in microseconds) of all executions of this plan in the interval |                                |
| sum_cpu_time                | Total CPU time (in microseconds) used by all executions in the interval |                                |
| sum_physical_io_reads       | Total physical I/O reads for all executions                  |                                |
| sum_logical_io_reads        | Total logical I/O reads for all executions                   |                                |
| sum_clr_time                | Total CLR time used by all executions                        |                                |
| sum_dop                     | Sum of degrees of parallelism across all executions          |                                |
| sum_row_count               | Total rows processed across all executions                   |                                |
| sum_warnings                | Total count of warnings across all executions                |                                |
| sum_tempdb_allocations      | Total number of tempdb allocations made by this query plan in the interval |                                |
| sum_tempdb_current          | Current tempdb usage for this plan at the time stats were collected |                                |
| sum_memory_grant_usage      | Total memory grant usage across all executions               | X                              |
| sum_degree_of_parallelism   | Sum of actual parallel threads used across all executions    |                                |
| avg_query_max_used_memory   | Average maximum memory (in KB) used by each execution        | X                              |
| min_query_max_used_memory   | Minimum of the maximum memory (in KB) used by any single execution | X                              |
| max_query_max_used_memory   | Maximum of the maximum memory (in KB) used by any single execution | X                              |
| last_query_max_used_memory  | Maximum memory (in KB) used by the most recent execution in this interval | X                              |
| stdev_query_max_used_memory | Standard deviation of the maximum memory usage across all executions | X                              |
| first_execution_time        | Time of the earliest execution in this interval              |                                |
| last_execution_time         | Time of the most recent execution in this interval           |                                |
