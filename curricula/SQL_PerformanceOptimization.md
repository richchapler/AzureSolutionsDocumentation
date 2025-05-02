# SQL: Performance Optimization

## Use Case

The database team at a mid‑sized company is under pressure... **data volumes are growing fast**, **users are complaining about slow queries**, and **critical reports are missing their deadlines**. Leadership wants answers and assigns a mission: uncover what's slowing things down, tune performance across the system, and make sure it does not happen again.

The database team talks about the mission and settles on the following goals:

- **Diagnose and fix slow queries** before users notice
- **Speed up data ingestion** so pipelines finish on time
- **Keep performance stable** even as load increases
- Eliminate guesswork through **clear, data-driven tuning**
- **Proactively optimize**, not just react to problems

<!-- ------------------------- ------------------------- ------------------------- -->

## Fundamentals

### Infrastructure

<!-- ------------------------- ------------------------- -->

#### Processor 
Control how SQL Server uses CPU cores to execute queries efficiently and consistently.

##### MAXDOP (Maximum Degree of Parallelism) 
Limit how many CPU cores SQL Server uses per query to reduce contention.

- **Default Value**: `0` (use all available cores) 
- **Suggested Value**: `4` or `8` for OLTP workloads; higher for reporting or batch systems 
- Prevents over-parallelization that can cause CPU spikes and CXPACKET waits 
- Best set at the **server level**, with **query-level overrides** for exceptions 

```sql
-- Set server-wide MAXDOP to 4
EXEC sp_configure 'show advanced options', 1; RECONFIGURE;
EXEC sp_configure 'max degree of parallelism', 4; RECONFIGURE;

-- Confirm current setting
SELECT value_in_use FROM sys.configurations WHERE name = 'max degree of parallelism';
```

<!-- ------------------------- ------------------------- -->

##### Cost Threshold for Parallelism 
Raise the bar for when SQL Server considers parallel execution.

- **Default Value**: `5` 
- **Suggested Value**: `25` to `50` on modern systems 
- Prevents lightweight queries from triggering expensive parallel plans unnecessarily 

```sql
-- Raise threshold to 25
EXEC sp_configure 'cost threshold for parallelism', 25; RECONFIGURE;

-- Confirm current value
SELECT value_in_use FROM sys.configurations WHERE name = 'cost threshold for parallelism';
```

<!-- ------------------------- ------------------------- -->

##### Query-Level MAXDOP 
Use per-query hints to override system-wide settings when needed.

- Use `OPTION (MAXDOP 1)` to force serial execution for queries that don't scale with parallelism 
- Useful for concurrency control in OLTP workloads 

```sql
SELECT ProductID, SUM(OrderQty) 
FROM Sales.SalesOrderDetail 
GROUP BY ProductID 
OPTION (MAXDOP 1);
```

<!-- ------------------------- ------------------------- -->

##### CXPACKET and CXCONSUMER Waits 
Analyze wait stats to detect CPU parallelism issues.

- **CXPACKET**: parallel threads waiting to synchronize 
- **CXCONSUMER**: successor wait type; same cause but more diagnostic clarity 
- High values may signal **over-parallelism** or **poor partitioning** 
- Use `sys.dm_os_wait_stats` and `sys.dm_exec_requests` to monitor 

```sql
-- Identify top waits including CX*
SELECT TOP 10 wait_type, wait_time_ms, signal_wait_time_ms
FROM sys.dm_os_wait_stats
WHERE wait_type LIKE 'CX%'
ORDER BY wait_time_ms DESC;
```

<!-- ------------------------- ------------------------- -->
<!-- ------------------------- ------------------------- -->

#### Memory

##### Memory and Plan Cache Behavior
...affects overall performance by controlling how SQL Server allocates memory and reuses query plans 

- **Plan Cache Bloat**: caused by many unique, single-use queries; wastes memory and slows down lookup 
- **Memory Grants**: each query requests memory before executing—overestimation can delay others, underestimation leads to spills to disk 
- **Single-Use Plan Detection**: use `sys.dm_exec_cached_plans` to find bloated caches 
- **Spill Warnings**: visible in execution plans when sorts or hashes exceed memory and write to TempDB 
- **Tuning Tools**: consider `OPTIMIZE_FOR_AD_HOC_WORKLOADS` and review execution memory in `sys.dm_exec_query_memory_grants` 

<!-- ------------------------- ------------------------- -->

##### Memory Grant Management and Spill Analysis 
...ensures queries receive the right amount of memory and avoids costly disk spills during execution 

- **Memory Grants**: queries request a specific amount of workspace memory before running; insufficient grants can queue or throttle execution 
- **Spill Indicators**: in actual execution plans, look for warnings on Sort or Hash operators that spill to TempDB when they exceed granted memory 
- **Dynamic Management Viewss for Insight**: query `sys.dm_exec_query_memory_grants` to see requested, granted, and ideal grant sizes, plus wait times 
- **Tuning Strategies**: reduce memory demands by filtering earlier, rewriting heavy operators, updating statistics for better estimates, or increasing server memory 
- **Azure SQL Notes**: memory grant behavior is tied to your service tier; monitor grants via Query Store or `sys.resource_stats` and scale up/off as needed 

<!-- ------------------------- ------------------------- -->

##### Resource Governance 
...ensures that one workload does not overwhelm others by controlling CPU, memory, and I/O allocations 

- **Resource Governor (on-premises only)**: create workload groups and assign limits based on user, query type, or application role 
- **Azure Tiers**: in Azure SQL, pick vCore or DTU tiers based on workload shape (transactional, analytical, mixed) 
- **Elastic Pools**: share resources across multiple databases with defined limits on CPU and memory per pool 
- **Throttling Symptoms**: increased latency, query timeouts, and IO waits under peak load 
- **Best Fit**: multi-tenant apps, mixed workload environments, and any system where noisy neighbors can impact critical queries 

<!-- ------------------------- ------------------------- -->
<!-- ------------------------- ------------------------- -->

#### Storage 
Optimize physical storage structures to support scale, reduce cost, and enhance query efficiency.

<!-- ------------------------- ------------------------- -->

##### Data Type Tuning 
Right-size column types to reduce memory and I/O costs.

- Use `int` instead of `bigint`, `date` instead of `datetime`, and trim `varchar`/`nvarchar` lengths 
- Avoid `nvarchar(max)` unless needed; control precision for financial/statistical data 
- Consistency avoids implicit conversions and simplifies joins

<!-- ------------------------- ------------------------- -->

##### Row and Page Compression 
Reduce the storage footprint of rowstore tables.

- **Row Compression**: shrinks fixed-length data (e.g., `char`, `int`) 
- **Page Compression**: adds prefix/dictionary compression 
- Improves I/O and buffer cache use at minor CPU cost 
- Use `sp_estimate_data_compression_savings` to preview benefit

<!-- ------------------------- ------------------------- -->

##### Columnstore and Archival Compression 
Use columnar storage for massive analytic workloads.

- **Columnstore Indexes**: highly compressed and scan-friendly 
- **Clustered**: replaces base table; **Nonclustered**: overlays rowstore 
- **Archival Compression**: maximizes storage savings on cold data 
- Ideal for telemetry, history tables, and data warehouses

<!-- ------------------------- ------------------------- -->

##### File and Filegroup Placement Strategies 
Spread data across storage for better I/O and recovery flexibility.

- Separate data, log, and TempDB across different media 
- Use filegroups to isolate workloads or improve backup strategies 
- Align table and index placement with performance goals

<!-- ------------------------- ------------------------- -->

##### TempDB Optimization 
...reduces contention and blocking in the shared temporary workspace used by all queries 

- **Multiple Data Files**: configure 1 data file per logical CPU (up to 8) to avoid page latch contention 
- **Equal Size and Growth**: ensure all TempDB files are the same size with the same autogrowth settings 
- **Location and I/O**: place TempDB on fast storage; isolate it from user databases if possible 
- **Monitoring**: track TempDB usage with Dynamic Management Viewss like `sys.dm_db_file_space_usage` and `sys.dm_db_session_space_usage` 
- **Workload Impact**: high concurrency environments (e.g., heavy reporting or temp tables) suffer most from poor TempDB setup 

<!-- ------------------------- ------------------------- ------------------------- -->

### Engine

<!-- ------------------------- ------------------------- -->
<!-- ------------------------- ------------------------- -->

#### Server-Level Configuration
Configure server-wide settings that influence how queries are compiled, parallelized, cached, and executed across all databases. These options help control memory use, CPU load, and plan cache efficiency at the instance level.

| Setting | Benefits | Tradeoffs | Why it Matters |
| :--- | :--- | :--- | :--- |
| `max degree of parallelism` | Reduces CPU contention from overly parallel queries (performance/stability) | Slower performance for large queries that benefit from full CPU use (performance) | Prevents one query from monopolizing CPU; improves throughput consistency |
| `cost threshold for parallelism` | Avoids parallelism for inexpensive queries (performance/efficiency) | May delay parallelism for moderately complex queries (performance) | Keeps CPU reserved for heavy queries that truly benefit from parallel plans |
| `optimize for adhoc workloads` | Reduces memory pressure from single-use query plans (performance/memory) | Slight delay for full plan reuse (performance) | Helps conserve plan cache in environments with many one-off queries |

<!-- ------------------------- ------------------------- -->

##### `max degree of parallelism`

When you lower the **max degree of parallelism (MAXDOP)**, SQL Server limits how many CPUs a single query can use, which helps reduce contention and CPU spikes on busy OLTP systems. 

This **prevents excessive parallelism** and helps ensure more consistent throughput, especially in workloads with many concurrent queries.

**Use When**... you want to reduce CPU pressure caused by parallel query bursts or uneven concurrency
**Default Value**: `0` (use all available processors)
SQL Server allows a query to use all available processors for parallel execution unless restricted by MAXDOP.
**Suggested Value**: `4` to `8` for OLTP workloads; higher for data warehouse or reporting servers.

```sql
EXEC show advanced options', 1;
RECONFIGURE;

EXEC max degree of parallelism', 4;
RECONFIGURE;
```

<!-- ------------------------- ------------------------- -->

##### `cost threshold for parallelism`

In most systems, this threshold is too low—causing even small queries to go parallel unnecessarily. Raising the value forces SQL Server to reserve parallelism for truly expensive queries.

This **reduces CPU pressure** and helps avoid over-parallelizing lightweight workloads.

**Use When**... you want to prevent short, low-cost queries from unnecessarily triggering parallel plans
**Default Value**: `5` (triggers parallelism on even modest queries)
SQL Server begins considering parallel plans when a query’s estimated cost exceeds this threshold.
**Suggested Value**: `20` to `50` in most production environments

```sql
EXEC show advanced options', 1;
RECONFIGURE;

EXEC cost threshold for parallelism', 25;
RECONFIGURE;
```

> ℹ️ **Tip:** `max degree of parallelism` and `cost threshold for parallelism` are often tuned together. 
> Use MAXDOP to limit how many CPUs a query can use, and Cost Threshold to decide when parallelism kicks in.

<!-- ------------------------- ------------------------- -->

##### `optimize for adhoc workloads`

When you enable **Optimize for Adhoc Workloads**, SQL Server saves just a stub initially and promotes it to a full plan only upon a subsequent execution.

This **reduces plan cache bloat** and **conserves memory** at the cost of delaying full plan caching until the query is reused.

**Use When**... you want to reduce plan cache bloat from rarely reused or one-time queries
**Default Value**: `OFF` (full plan cached on first execution)
SQL Server saves the full plan for every query—even those never reused—unless this setting is enabled.
**Suggested Value**: `ON` for workloads with high ad hoc query volume

```sql
EXEC show advanced options', 1; 
RECONFIGURE;

EXEC optimize for adhoc workloads', 1; 
RECONFIGURE;
```

<!-- ------------------------- ------------------------- -->
<!-- ------------------------- ------------------------- -->

#### Database-Level Configuration
Configure database‑level settings to control plan caching and literal parameterization.

| Setting | Benefits | Tradeoffs | Why it Matters |
| :--- | :--- | :--- | :--- |
| `QUERY_STORE` | Enables plan analysis, regression detection, and stabilization (stability) | Slight storage and capture overhead (overhead) | Provides data to detect and fix performance regressions over time |
| `COMPATIBILITY_LEVEL` | Unlocks modern optimizer behavior and better estimations (performance) | Can introduce plan regressions—requires validation (stability) | Controls optimizer model; newer levels often produce better plans |
| `PARAMETERIZATION = FORCED` | Reduces ad hoc plan cache bloat and improves reuse (performance) | May prevent optimal plan choices for certain parameter values (flexibility) | Helps normalize ad hoc query shapes for better plan reuse |
| `AUTO_CREATE_STATISTICS` | Improves plan quality with better cardinality estimates (performance) | Slight I/O and CPU impact during stats creation (overhead) | Ensures optimizer has basic stats for good selectivity predictions |
| `AUTO_UPDATE_STATISTICS_ASYNC` | Allows queries to run without blocking on stats refresh (performance/concurrency) | Risk of slightly stale stats during update window (accuracy) | Helps avoid blocking on stats updates in reporting workloads |
| `LEGACY_CARDINALITY_ESTIMATION` | Prevents regressions when upgrading older workloads (stability) | Misses improvements in newer estimation logic (performance/flexibility) | Useful safeguard during or after compatibility level upgrades |

<!-- ------------------------- ------------------------- -->

##### `QUERY_STORE` 
...stores past query plans and performance to help you troubleshoot and improve slow queries

- **Captures Execution History**: stores actual execution plans, query text, and performance statistics 
- **Detects Regressions**: identifies when a new plan performs worse than a previous one 
- **Forces Stable Plans**: allows you to pin a known‑good plan for a given query to prevent further regressions 
- **Configuration Settings**: tune key Query Store parameters—operation mode, storage limit, data flush rate—to keep performance history available 

<!-- ------------------------- -->

**Use When**... you need to spot regressions, analyze plan changes, or lock in stable plans for critical queries.
**Default Value**: `OFF` (enabled by default in Azure SQL) 
**Suggested Value**: `ON` with `READ_WRITE` mode and configured size limits 

Execute the following T-SQL to activate:
```sql
-- Turn Query Store ON in read-write mode
ALTER DATABASE [YourDatabase] SET QUERY_STORE = ON (OPERATION_MODE = READ_WRITE);
-- Check if Query Store is enabled
SELECT actual_state_desc FROM sys.database_query_store_options WHERE database_id = DB_ID();
```

<img src="..\images\SQL_PerformanceOptimization\QueryStore.png" width="800" title="Snipped April, 2025" />

<!-- ------------------------- ------------------------- -->

##### `COMPATIBILITY_LEVEL` 
...controls which version of the SQL Server engine your database uses, including how queries are optimized

Setting the **compatibility level** to a higher value unlocks new optimizer features, CE improvements, and T‑SQL enhancements. However, jumps in compatibility level can introduce plan regressions, so it’s best to test carefully when moving up. 

<!-- ------------------------- -->

**Use When**... you want to unlock modern optimizer behavior while ensuring plan stability by testing regressions before upgrading.
**Default Value**: Varies by SQL Server version (e.g. `150` for SQL Server 2019) 
**Suggested Value**: Use the latest supported level (`150` or higher), but test for regressions before upgrading 

```sql
-- Set compatibility level to SQL Server 2019 behavior
ALTER DATABASE [YourDatabase] SET COMPATIBILITY_LEVEL = 150;
-- Check current compatibility level
SELECT compatibility_level FROM sys.databases WHERE name = 'YourDatabase';
``` 

<!-- ------------------------- ------------------------- -->

##### `PARAMETERIZATION = FORCED`
...automatically turns query literals into parameters so SQL Server can reuse plans and save memory

By default, SQL Server uses **Simple Parameterization** only for very basic literals and leaves most ad hoc queries unparameterized.

When you set the database to **Forced Parameterization** every literal that can be safely parameterized is turned into a parameter.

This increases plan reuse and reduces plan cache bloat at the cost of occasionally generating less‑optimal plans.

<!-- ------------------------- -->

**Use When**... you want to reduce single-use ad hoc plans and increase plan reuse across similar queries with different literal values.
**Default Value**: `SIMPLE` (automatic only for basic queries) 
**Suggested Value**: `FORCED` in workloads with lots of ad hoc queries 

```sql
-- Enable forced parameterization for ad hoc queries
ALTER DATABASE [YourDatabase] SET PARAMETERIZATION = FORCED;
-- Check if forced parameterization is enabled
SELECT name, is_parameterization_forced FROM sys.databases WHERE name = 'YourDatabase';
``` 

**Now both executions share a single cached plan instead of two separate ones.**

<!-- ------------------------- ------------------------- -->

##### AUTO_CREATE_STATISTICS 
...automatically builds basic stats for query columns to help SQL Server estimate row counts more accurately

By default, **AUTO_CREATE_STATISTICS** is **ON**, so SQL Server will generate lightweight, single‑column stats to help the optimizer with selectivity estimates. Disabling it can reduce overhead on very write‑heavy systems but risks poor cardinality estimates and suboptimal plans. 

<!-- ------------------------- -->

**Use When**... you want the optimizer to generate basic stats automatically for better row estimates in unpredictable or dynamic workloads.
**Default Value**: `ON` 
**Suggested Value**: Keep `ON` unless tuning for extremely write-heavy workloads 

```sql
-- Enable auto-create statistics for single-column predicates
ALTER DATABASE [YourDatabase] SET AUTO_CREATE_STATISTICS ON;
-- Check if auto-create statistics is enabled
SELECT name, is_auto_create_stats_on FROM sys.databases WHERE name = 'YourDatabase';
``` 

<!-- ------------------------- ------------------------- -->

##### AUTO_UPDATE_STATISTICS_ASYNC 
...lets queries run while stats are being refreshed, so they don’t get blocked by maintenance

By default, **AUTO_UPDATE_STATISTICS_ASYNC** is **OFF**, meaning long‑running queries will wait for statistics to be refreshed before executing. Turning it **ON** can improve throughput for heavy reporting workloads but means queries might run with slightly stale statistics. 

**Use When**... you want to avoid blocking in read-heavy systems where queries might otherwise wait for stats updates.
**Default Value**: `OFF` 
**Suggested Value**: `ON` for reporting or read-heavy workloads where query blocking on stats updates is a problem 

<!-- ------------------------- -->

```sql
-- Enable async updates for statistics
ALTER DATABASE [YourDatabase] SET AUTO_UPDATE_STATISTICS_ASYNC ON;
-- Check if async stats updates are enabled
SELECT name, is_auto_update_stats_async_on FROM sys.databases WHERE name = 'YourDatabase';
``` 

<!-- ------------------------- ------------------------- -->

##### LEGACY_CARDINALITY_ESTIMATION 
...uses the old SQL Server math for estimating row counts to avoid surprises after an upgrade

By default, databases at compatibility level 150+ use the modern estimator. Enabling **LEGACY_CARDINALITY_ESTIMATION** can avoid regressions after an upgrade but may miss improvements in multi‑column correlation handling. 

<!-- ------------------------- -->

**Use When**... upgrading the compatibility level introduces plan regressions and you want to restore legacy estimation behavior temporarily.
**Default Value**: `OFF` in compatibility level 150+ 
**Suggested Value**: `ON` only to restore plan behavior after regressions in upgraded systems 

```sql
-- Enable legacy cardinality estimator
ALTER DATABASE SCOPED CONFIGURATION SET LEGACY_CARDINALITY_ESTIMATION = ON;
-- Check if legacy estimator is enabled
SELECT name, is_legacy_cardinality_estimation_on FROM sys.database_scoped_configurations;
``` 

<!-- ------------------------- ------------------------- ------------------------- -->

### Logic
...involves modifying the structure of a query to improve performance without changing the result

<!-- ------------------------- ------------------------- -->
<!-- ------------------------- ------------------------- -->

#### Basics

##### "Execution Plan"
...reveals how the SQL engine interprets and executes a query, enabling developers and database professionals to identify bottlenecks and improve performance 

- **Estimated Execution Plan**: displays **how SQL Server plans to run a query** before it is executed; helps spot inefficient join orders, missing indexes, or inaccurate estimates without actually modifying data 
- **Actual Execution Plan**: includes details **collected during query execution**, such as actual row counts and time spent on each operation; essential for finding real-world performance problems and mismatches between estimated and actual costs 
- **Plan Comparison**: allows you to compare execution plans **before and after changes** and helps confirm that: 1) a performance issue is fixed or 2) spot regressions after a change 

**Use When...** you need to pinpoint inefficient operators, missing indexes, or estimate inaccuracies **before and after changes**

<img src="..\images\SQL_PerformanceOptimization\ActualExecutionPlan.png" width="800" title="Snipped April, 2025" />

<!-- ------------------------- ------------------------- -->

##### "Live Query Statistics"
...gives a live, visual view of query progress while it's running 

- **Real-Time Insight**: shows operator-level progress and row counts during query execution 
- **Early Detection**: helps identify where a query is stalled, blocked, or slower than expected 
- **Best For**: troubleshooting long-running or interactive queries in development and test environments

**Use When...** you need to spot stalls, blocking, or unexpectedly slow operators **in real time**

<img src="..\images\SQL_PerformanceOptimization\LiveQueryStatistics.png" width="800" title="Snipped April, 2025" />

<!-- ------------------------- ------------------------- -->
<!-- ------------------------- ------------------------- -->

#### Query Design

-------------------------

##### Set‑Based Processing
Work on all rows at once instead of one at a time.

**DON'T**: Aggregate per filter item
```sql
SELECT COUNT(*) FROM Sales.SalesOrderHeader WHERE CustomerID = 1;
SELECT COUNT(*) FROM Sales.SalesOrderHeader WHERE CustomerID = 2;
``` 

**DO**: Aggregate in one pass
```sql
SELECT CustomerID, COUNT(*) FROM Sales.SalesOrderHeader GROUP BY CustomerID;
```

-------------------------

##### Eliminate Unnecessary Function Use 
Remove unnecessary functions from WHERE or JOIN so indexes can be used.

**DON'T**: Wrap column in a function 
```sql
SELECT SalesOrderID FROM Sales.SalesOrderHeader WHERE YEAR(OrderDate) = 2021;
```

**DO**: Filter by date directly 
```sql
SELECT SalesOrderID FROM Sales.SalesOrderHeader WHERE OrderDate = '2021-01-01';
```

-------------------------

##### Bake Filters into Joins 
Apply filters in the JOIN clause so less data is processed.

**DON'T**: Filter after joining all rows 
```sql
SELECT h.SalesOrderID
FROM Sales.SalesOrderHeader AS h 
 JOIN Sales.Customer AS c ON h.CustomerID = c.CustomerID 
WHERE c.Country = 'USA';
```

**DO**: Filter as you join 
```sql
SELECT h.SalesOrderID
FROM Sales.SalesOrderHeader AS h 
 JOIN Sales.Customer AS c ON h.CustomerID = c.CustomerID AND c.Country = 'USA';
```

-------------------------

##### Flatten Sub-Queries 
Replace nested queries with joins so everything runs in one pass.

**DON'T**: Run the subquery per row 
```sql
SELECT o.SalesOrderID,
 (SELECT SUM(OrderQty) FROM Sales.SalesOrderDetail WHERE SalesOrderID = o.SalesOrderID) AS TotalQty 
FROM Sales.SalesOrderHeader AS o;
```

**DO**: Aggregate with a join and GROUP BY 
```sql
SELECT o.SalesOrderID, SUM(d.OrderQty) AS TotalQty
FROM Sales.SalesOrderHeader AS o 
 JOIN Sales.SalesOrderDetail AS d ON d.SalesOrderID = o.SalesOrderID 
GROUP BY o.SalesOrderID;
```

-------------------------

##### Reduce Column Retrieval 
Select only the columns you need to reduce network traffic.

**DON'T**: Fetch every Column 
```sql
SELECT * FROM Sales.SalesOrderDetail;
```

**DO**: Retrieve Specific fields 
```sql
SELECT SalesOrderID, OrderQty FROM Sales.SalesOrderDetail;
```

-------------------------

##### "Sargable" Conditions
> Note: "Sargable" is shorthand for Search ARGument ABLE.<Br>It describes predicates that the SQL engine can turn into an index seek or range scan instead of a full table scan.

Write predicates so indexes can be used (no functions on columns).

**DON'T**: Apply a function to the column 
```sql
SELECT SalesOrderID FROM Sales.SalesOrderHeader 
WHERE YEAR(OrderDate) = 2021;
```

**DO**: Use a date range that can leverage an index 
```sql
SELECT SalesOrderID FROM Sales.SalesOrderHeader 
WHERE OrderDate >= '2021-01-01' AND OrderDate < '2022-01-01';
```

-------------------------

##### Simplify Predicates 
Combine filters into a single IN list instead of OR chains.

**DON'T**: Use OR for each value 
```sql
SELECT SalesOrderID FROM Sales.SalesOrderHeader 
WHERE Status = 'Shipped' OR Status = 'Processing';
```

**DO**: Use IN() for multiple values 
```sql
SELECT SalesOrderID FROM Sales.SalesOrderHeader WHERE Status IN ('Shipped','Processing');
```

-------------------------

##### Eliminate Redundant Joins 
Drop any table joins that do not affect your result.

**DON'T**: Add unused table joins 
```sql
SELECT o.SalesOrderID 
FROM Sales.SalesOrderHeader AS o 
 JOIN Sales.SalesOrderDetail AS d ON d.SalesOrderID = o.SalesOrderID 
 JOIN Production.Product AS p ON p.ProductID = d.ProductID;
```

**DO**: Join only the tables you need 
```sql
SELECT o.SalesOrderID 
FROM Sales.SalesOrderHeader AS o 
 JOIN Sales.SalesOrderDetail AS d ON d.SalesOrderID = o.SalesOrderID;
```

-------------------------

##### Use UNION ALL 
Choose UNION ALL when you do not need to remove duplicates to avoid extra work.

**DON'T**: Impose a duplicate check 
```sql
SELECT CustomerID FROM Sales.SalesOrderHeader WHERE OrderDate < '2020-01-01' 
UNION 
SELECT CustomerID FROM Sales.SalesOrderHeader WHERE OrderDate >= '2020-01-01';
```

**DO**: Concatenate results directly 
```sql
SELECT CustomerID FROM Sales.SalesOrderHeader WHERE OrderDate < '2020-01-01' 
UNION ALL 
SELECT CustomerID FROM Sales.SalesOrderHeader WHERE OrderDate >= '2020-01-01';
```

-------------------------

##### Proactive Monitoring 
Catch plan regressions before users notice. 

**DON'T**: wait for user complaints 
 
**DO**: query Query Store for slowest queries 
```sql
SELECT TOP 1 q.query_id, rs.avg_duration
FROM sys.query_store_runtime_stats AS rs
 JOIN sys.query_store_plan AS p ON rs.plan_id = p.plan_id
 JOIN sys.query_store_query AS q ON p.query_id = q.query_id
ORDER BY rs.avg_duration DESC;
```

<!-- ------------------------- ------------------------- -->
<!-- ------------------------- ------------------------- -->

#### Query-Level Hints
Use hints to override the optimizer when its default plan doesn’t meet your performance needs.

##### Join
Join hints override the optimizer’s default join strategy or join order when its cost‑based choice is not ideal.

| Hint | Impact |
| :--- | :--- |
| `OPTION (LOOP JOIN)` | Forces a nested‑loops join; efficient for small, highly selective inputs |
| `OPTION (HASH JOIN)` | Forces a hash join; optimal for large, unsorted equi‑joins |
| `OPTION (MERGE JOIN)` | Forces a merge join; streams sorted inputs with low memory overhead |
| `OPTION (FORCE ORDER)` | Preserves the join order you wrote; useful when optimizer’s order is suboptimal |

##### Index
Index hints override the optimizer’s default index selection to force or avoid specific indexes when its cost‑based choice leads to inefficient access paths

| Hint | Impact |
| :--- | :--- |
| `WITH (FORCESEEK(IndexName))` | Forces an index seek on the specified index for selective filters |
| `WITH (FORCESCAN)` | Forces a full scan of the clustered or specified index |
| `WITH (INDEX(index_list))` | Restricts optimizer to only the specified index(es) |

##### Parallelism
Parallelism hints override SQL Server’s default degree of parallelism when you need to balance CPU utilization and query responsiveness for specific workloads

| Hint | Impact |
| :--- | :--- |
| `OPTION (MAXDOP n)` | Limits degree‑of‑parallelism for the query, balancing CPU utilization and throughput |
| `OPTION (FAST n)` | Optimizes plan to return the first *n* rows quickly, useful for interactive or paginated queries |

##### Plan Stability
Plan stability hints override SQL Server’s plan generation and reuse policies when you require consistent or optimal execution plans across varying parameter values

| Hint | Impact |
| :--- | :--- |
| `OPTION (OPTIMIZE FOR UNKNOWN)` | Builds a generic plan ignoring the first parameter, balancing cost across inputs |
| `OPTION (RECOMPILE)` | Discards cached plan and compiles a fresh plan each execution, avoiding sniffing |

##### Plan‑Cache
Plan‑cache hints override SQL Server’s plan caching behavior to control plan retention and reduce cache bloat when eviction or pollution hurts performance

| Hint | Impact |
| :--- | :--- |
| `OPTION (OPTIMIZE FOR AD HOC WORKLOADS)` | Caches only a stub on first run of ad hoc queries, reducing cache bloat |
| `OPTION (KEEP PLAN)` | Prevents eviction under memory pressure, ensuring quick reuse |
| `OPTION (KEEPFIXED PLAN)` | Pins plan across stats/index changes or upgrades |

##### Concurrency
Concurrency hints override SQL Server’s default locking behavior to minimize blocking or deadlocks when high contention degrades throughput

| Hint | Impact |
| :--- | :--- |
| `WITH (NOLOCK)` | Reads without shared locks (READ UNCOMMITTED), reducing blocking |
| `WITH (READPAST)` | Skips locked rows to avoid blocking on heavily contended tables |
| `WITH (UPDLOCK)` | Acquires update locks on read to prevent deadlocks when planning updates |
| `WITH (ROWLOCK)` | Forces row‑level locks only, avoiding broader lock escalation |

<!-- ------------------------- ------------------------- -->
<!-- ------------------------- ------------------------- -->

#### Indexing 
Configure and maintain indexes to reduce I/O, support index seeks, and improve execution plan quality.

| Focus Area | Benefits | Tradeoffs | Why it Matters |
| :--- | :--- | :--- | :--- |
| **Clustered Indexes** | Define row order for fast range scans and ORDER BY queries | Only one per table; impacts insert performance | Drive physical layout and affect all nonclustered index performance |
| **Nonclustered Indexes** | Accelerate filters, joins, and lookups | Increase write cost and plan complexity if overused | Allow targeted access to frequently queried columns |
| **Filtered Indexes** | Improve query speed for common filtered subsets | Only useful for predictable filters | Reduce index size and maintenance for highly selective filters |
| **Covering Indexes** | Avoid lookups by including all required columns | Consume more space and increase index size | Let SQL Server satisfy queries using only the index |
| **Missing Index Dynamic Management Viewss** | Suggest indexes based on actual query patterns | May overlap or miss multi-column interactions | Help surface opportunities from real workload activity |
| **Fragmentation Management** | Rebuild/reorganize improves scan and seek performance | Maintenance overhead; needs automation | Prevents inefficient I/O and high CPU from fragmented access paths |
| **Fill Factor** | Reduces page splits on insert-heavy tables | Uses more space; may slightly slow reads | Helps balance update speed and read efficiency |

<!-- ------------------------- ------------------------- -->

##### Clustered and Nonclustered Indexes 
...define how rows are physically stored and accessed.

- **Clustered Index**: determines row order; best for range queries and sort operations 
- **Nonclustered Index**: separate structures targeting frequent filters and joins 
- Use clustered indexes for **sorting or range retrieval** 
- Use nonclustered indexes to **target narrow queries** on selective columns

**Use this when...** queries scan or sort large tables and benefit from precise data access paths.

<!-- ------------------------- ------------------------- -->

##### Filtered and Covering Indexes 
...optimize indexing for specific queries.

- **Filtered Index**: indexes only rows matching a WHERE clause (e.g. `IsActive = 1`) 
- **Covering Index**: includes all columns needed to satisfy a query without base-table access 

**Use this when...** queries target common filters or need to eliminate lookups for better performance.

<!-- ------------------------- ------------------------- -->

##### Missing Index Recommendations 
...identify indexing gaps based on actual query execution.

- Query `sys.dm_db_missing_index_details` and `sys.dm_db_missing_index_group_stats` 
- Evaluate `equality_columns`, `inequality_columns`, and `included_columns` 
- Use `avg_user_impact` and `user_seeks` to prioritize value 
- Always validate with **Query Store** or **A/B testing**

**Use this when...** you want to base indexing changes on observed workload behavior.

```sql
-- View top recommended indexes by potential benefit
SELECT TOP 5 migs.avg_user_impact, mid.equality_columns, mid.inequality_columns, mid.included_columns
FROM sys.dm_db_missing_index_details AS mid
JOIN sys.dm_db_missing_index_groups AS mig ON mid.index_handle = mig.index_handle
JOIN sys.dm_db_missing_index_group_stats AS migs ON mig.index_group_handle = migs.group_handle
ORDER BY migs.avg_user_impact DESC;
```

<!-- ------------------------- ------------------------- -->

##### Index Maintenance and Fragmentation 
...keep indexes efficient by reducing physical fragmentation.

- **Rebuild** when fragmentation > 30% 
- **Reorganize** when fragmentation is between 5–30% 
- Use `sys.dm_db_index_physical_stats` to monitor health 
- Adjust **fill factor** to reduce future page splits 
- Schedule regular maintenance via SQL Agent or Azure Automation

**Use this when...** index fragmentation is degrading seek speed and increasing CPU usage.

```sql
-- Rebuild all indexes on a table with a custom fill factor
ALTER INDEX ALL ON Sales.SalesOrderHeader
REBUILD WITH (FILLFACTOR = 90, ONLINE = ON);
```

<!-- ------------------------- ------------------------- ------------------------- -->

### Advanced 

#### Concurrency
Improve performance in high-concurrency environments by reducing contention between readers and writers. 

**Even well-tuned queries can suffer when multiple sessions compete for the same data**.

| Feature | Benefits | Tradeoffs | Why it Matters |
| :--- | :--- | :--- | :--- |
| **Short Transactions** | Reduces lock duration and risk of blocking | May require changes to how work is batched | Long transactions hold locks longer, which increases blocking and deadlock risk |
| **Proper Indexing** | Reduces locking footprint by limiting rows scanned | Adds overhead to writes and storage | Touching fewer rows lowers the chance of conflicting locks during reads/writes |
| **READ COMMITTED** (default isolation level) | Avoids dirty reads and maintains simple consistency | Readers can block writers, and vice versa | Good balance for many OLTP systems but can introduce contention |
| **READ UNCOMMITTED** | Readers never block; best concurrency | Can read uncommitted/dirty data | Useful in analytics or telemetry where freshness matters less |
| **SNAPSHOT / READ_COMMITTED_SNAPSHOT** | Readers and writers don’t block each other | Higher tempdb usage and CPU overhead | Enables high concurrency with consistent reads by using versioned row copies |
| **Deadlock Detection** | SQL Server breaks cycles automatically | One session is always chosen as a victim | Prevents full system stalls when queries wait on each other |

#### Monitoring 
**Collect and analyze telemetry** to detect bottlenecks and plan capacity 

| Feature | Benefits | Tradeoffs | Why It Matters |
| :--- | :--- | :--- | :--- |
| **Wait Statistics** | Reveals resource waits such as locks and I/O | Lacks query‑level context | Pinpoints systemic bottlenecks for targeted tuning |
| **Query Profiling** | Captures execution metrics and runtime statistics | Can incur overhead under load | Identifies slow queries and hotspots early |
| **Performance Insight** | Provides dashboard views of trends and top queries | Limited to predefined metrics | Supports capacity planning and proactive optimization |
| **Extended Events** | Records detailed events like slow queries or plan changes | Requires session setup and storage management | Enables granular analysis of intermittent issues |

#### Tuning
Leverage **built‑in automation to apply performance corrections without manual intervention** 

| Feature | Benefits | Tradeoffs | Why It Matters |
| :--- | :--- | :--- | :--- |
| **Force Last Good Plan** | Reverts to a known‑good plan on regression | May mask underlying issues | Maintains stable performance after plan changes |
| **Create Index** | Automatically adds recommended indexes | Risk of over‑indexing | Reduces manual maintenance and speeds queries |
| **Drop Index** | Removes unused indexes based on usage | Could remove future‑needed indexes | Lowers storage and maintenance overhead |
| **Plan Regression Detection** | Alerts on degraded plans via Query Store | May generate false positives | Ensures plan consistency over time |
| **Tuning Recommendations** | Centralizes index and plan actions in portal | Requires review before applying changes | Simplifies decision making and workload optimization 

<!-- ------------------------- ------------------------- ------------------------- -->

### Quiz

1. Which execution plan type shows actual row counts and time per operator after a query runs 
 A) estimated execution plan 
 B) actual execution plan 
 C) live query statistics 
 D) client statistics 

2. Which hint can you add to a query to avoid plan reuse based on initial parameter values 
 A) FORCE ORDER 
 B) OPTIMIZE FOR UNKNOWN 
 C) MAXDOP 
 D) RECOMPILE 

3. You discover Query Store switched itself to read_only. Which setting should you increase to prevent that 
 A) OPERATION_MODE 
 B) MAX_STORAGE_SIZE_MB 
 C) DATA_FLUSH_INTERVAL_SECONDS 
 D) STALE_QUERY_THRESHOLD 

4. If you frequently filter on IsActive = 1 which index type is most appropriate 
 A) clustered index 
 B) nonclustered index 
 C) filtered index 
 D) covering index 

5. After a bulk data load which action is recommended to keep optimizer estimates accurate 
 A) rebuild all indexes 
 B) reorganize fragmented indexes 
 C) update statistics 
 D) truncate log 

6. On a server with 16 logical processors how many tempdb data files are recommended 
 A) 4 
 B) 8 
 C) 12 
 D) 16 

7. Which database scoped configuration reduces memory use from single‑use plans 
 A) LEGACY_CARDINALITY_ESTIMATION 
 B) QUERY_OPTIMIZER_HOTFIXES 
 C) OPTIMIZE_FOR_SEQUENTIAL_KEY 
 D) OPTIMIZE_FOR_AD_HOC_WORKLOADS 

1. Which database‑level setting lets queries run while statistics are being refreshed, avoiding blocking? 
 A) AUTO_CREATE_STATISTICS 
 B) AUTO_UPDATE_STATISTICS 
 C) AUTO_UPDATE_STATISTICS_ASYNC 
 D) LEGACY_CARDINALITY_ESTIMATION 

1. To prevent readers blocking writers without rewriting queries which option should you enable 
 A) ALLOW_SNAPSHOT_ISOLATION 
 B) READ_COMMITTED_SNAPSHOT 
 C) READ_UNCOMMITTED 
 D) SNAPSHOT 

1. Which Dynamic Management Views shows cumulative wait times by wait type for diagnosing bottlenecks 
 A) sys.dm_exec_requests 
 B) sys.dm_os_wait_stats 
 C) sys.dm_exec_query_stats 
 D) sys.dm_db_index_physical_stats 

<!-- ------------------------- ------------------------- -->

#### Answers

1. **B** – actual execution plan includes real row counts and operator timings 
 *It captures runtime metrics that reveal where a query is spending time* 

2. **B** – OPTIMIZE FOR UNKNOWN prevents plans from being optimized for a specific parameter value 
 *It instructs the optimizer to generate a generic plan rather than one tuned to the initial input* 

3. **C** – increasing the data flush interval (`DATA_FLUSH_INTERVAL_SECONDS`) reduces how often Query Store writes data and prevents it from filling up and becoming read‑only 
 *A longer flush interval lowers I/O pressure and avoids automatic mode changes* 

4. **C** – filtered index targets only the `IsActive = 1` subset, reducing index size and improving seek performance 
 *It indexes only the rows you query most often* 

5. **C** – updating statistics after large loads refreshes cardinality estimates for the optimizer 
 *Accurate statistics ensure the optimizer chooses efficient plans* 

6. **B** – eight TempDB files for up to 16 CPUs minimizes latch contention without excessive overhead 
 *One file per two CPUs (up to eight) is the recommended balance* 

7. **D** – OPTIMIZE_FOR_AD_HOC_WORKLOADS stubs single‑use plans until reused, reducing memory bloat 
 *It stores only a small plan stub on first execution and the full plan only if reused* 

8. **C** – AUTO_UPDATE_STATISTICS_ASYNC lets queries run while statistics are being refreshed, avoiding blocking 
*Allows queries to execute with slightly stale statistics rather than waiting, improving concurrency in read‑heavy workloads*

9. **B** – READ_COMMITTED_SNAPSHOT uses row versioning to prevent reader/writer blocking without code changes 
 *It provides read consistency via tempdb versioning instead of locks* 

10. **B** – sys.dm_os_wait_stats shows cumulative wait times by wait type, key for diagnosing resource waits 
 *It identifies the most costly waits so you know which resources to tune*

<!-- ------------------------- ------------------------- ------------------------- -->

## On‑Prem

<!-- ------------------------- ------------------------- ------------------------- -->

### Exercise #1: "Missing Index" Recommendations 
Use built‑in Dynamic Management Views to identify index candidates based on actual workload patterns

<!-- ------------------------- ------------------------- -->

Enable I/O and time statistics: 
```sql
USE AdventureWorks2022;
SET STATISTICS IO, TIME ON;
```

`SET STATISTICS IO, TIME ON` tells SQL Server to report detailed I/O and timing info for each query, including logical reads, physical reads, read‑ahead reads, CPU time and elapsed time in the Messages pane of SQL Server Management Studio

Run a baseline query (and include Actual Execution Plan): 
 ```sql
SELECT SalesOrderID, ProductID, UnitPrice, OrderQty FROM Sales.SalesOrderDetail WHERE UnitPrice > 1000;
 ``` 
The Execution Plan should show a Clustered Index **Scan** and "Missing Index..." guidance.

Retrieve missing‑index recommendations: 
 ```sql
SELECT mid.equality_columns, mid.inequality_columns, mid.included_columns, migs.user_seeks, migs.avg_total_user_cost
FROM sys.dm_db_missing_index_details AS mid
	JOIN sys.dm_db_missing_index_groups AS mig ON mid.index_handle = mig.index_handle
	JOIN sys.dm_db_missing_index_group_stats AS migs ON mig.index_group_handle = migs.group_handle
WHERE mid.object_id = OBJECT_ID('Sales.SalesOrderDetail');
 ``` 

This query analyzes which predicates and columns your workload touches and then surfaces index "candidates" as rows in the result.
<br>**Each row maps directly to an index you could create**, with details in columns:

- **inequality_columns** tells you which column(s) to put in the nonclustered‑index key 
- **equality_columns** would list any exact‑match columns (NULL here since none) 
- **included_columns** shows which columns to add as INCLUDES 
- **user_seeks** / **avg_total_user_cost** quantify how often and how costly queries are without that index 

In your example, the first row suggests: 
```sql
CREATE NONCLUSTERED INDEX IX_SalesOrderDetail_UnitPrice
 ON Sales.SalesOrderDetail(UnitPrice) -- from inequality_columns
 INCLUDE (OrderQty, ProductID); -- from included_columns
``` 

...because queries frequently filter on UnitPrice and then return OrderQty and ProductID.
<br>Once created, that index will turn scans into seeks and reduce the avg_total_user_cost reported by the Dynamic Management Views.

Create the recommended nonclustered index: 
 ```sql
 CREATE NONCLUSTERED INDEX IX_SOD_UnitPrice ON Sales.SalesOrderDetail (UnitPrice) INCLUDE (SalesOrderID, ProductID, OrderQty);
 ``` 

Expected result:
```plaintext
SQL Server parse and compile time: 
 CPU time = 0 ms, elapsed time = 0 ms.

 SQL Server Execution Times:
 CPU time = 0 ms, elapsed time = 0 ms.
SQL Server parse and compile time: 
 CPU time = 0 ms, elapsed time = 8 ms.
Table 'SalesOrderDetail'. Scan count 7, logical reads 1237, physical reads 0, page server reads 0, read-ahead reads 1237, page server read-ahead reads 0, lob logical reads 0, lob physical reads 0, lob page server reads 0, lob read-ahead reads 0, lob page server read-ahead reads 0.

 SQL Server Execution Times:
 CPU time = 140 ms, elapsed time = 376 ms.
SQL Server parse and compile time: 
 CPU time = 0 ms, elapsed time = 0 ms.

 SQL Server Execution Times:
 CPU time = 0 ms, elapsed time = 0 ms.

Completion time: 2025-04-22T12:43:44.1098059-07:00
```

Re‑run the query and observe improvements: 
```sql
SELECT SalesOrderID, ProductID, UnitPrice, OrderQty FROM Sales.SalesOrderDetail WHERE UnitPrice > 1000;
```
The Execution Plan should now show a Index **Seek** and STATISTICS IO reports far fewer logical reads.

Clean up the test index: 
```sql
DROP INDEX IX_SOD_UnitPrice ON Sales.SalesOrderDetail;
``` 

#### Final Thought 
By leveraging missing‑index Dynamic Management Views, you can pinpoint exactly which columns and included fields will yield the largest performance gains. This hands‑on method ensures you add only the indexes your workload truly needs, avoiding unnecessary maintenance overhead and write penalties.

------------------------- ------------------------- -------------------------

### Exercise #2: Index Maintenance

<!-- ------------------------- ------------------------- -->

Check current fragmentation:
```sql
USE AdventureWorks2022; 
SELECT OBJECT_NAME(ips.object_id) AS TableName, i.name AS IndexName, ips.index_type_desc, ips.avg_fragmentation_in_percent
FROM sys.dm_db_index_physical_stats(DB_ID(), NULL, NULL, NULL, 'LIMITED') AS ips
 JOIN sys.indexes AS i ON ips.object_id = i.object_id AND ips.index_id = i.index_id
WHERE ips.database_id = DB_ID() AND ips.avg_fragmentation_in_percent > 5;
```

This query identifies indexes with fragmentation above 5%.

<img src="..\images\SQL_PerformanceOptimization\IndexFragmentation.png" width="800" title="Snipped April, 2025" />

Review usage statistics:
```sql
SELECT OBJECT_NAME(ius.object_id) AS TableName, i.name AS IndexName, ius.user_seeks, ius.user_scans, ius.user_updates
FROM sys.dm_db_index_usage_stats AS ius
	JOIN sys.indexes AS i ON ius.object_id = i.object_id AND ius.index_id = i.index_id
WHERE database_id = DB_ID() AND OBJECTPROPERTY(ius.object_id, 'IsUserTable') = 1;
``` 
This query shows how often each index on your user tables in the current database has been used. 

Use these metrics to identify hot indexes (high seeks/scans) and cold or unused indexes (low or zero counts) so you can focus maintenance or consider dropping unused ones.

Reorganize lightly fragmented indexes:
```sql
ALTER INDEX ALL ON Sales.SalesOrderDetail 
REORGANIZE;
``` 

Rebuild heavily fragmented indexes: 
```sql
ALTER INDEX ALL ON Sales.SalesOrderDetail 
REBUILD WITH (FILLFACTOR = 90, ONLINE = ON);
``` 

| Action | Locking Behavior | Duration and Impact | Fill Factor | Recommended When |
| :--- | :--- | :--- | :--- | :--- |
| `REORGANIZE` | acquires low‑level page locks, works online | shorter operation, minimal blocking, defragments in place | uses existing fill factor | fragmentation 5–30 percent |
| `REBUILD` | acquires schema‑modification lock, rebuilds index structure | longer operation, blocks (unless ONLINE = ON), reorder pages fully | you specify new fill factor | fragmentation above 30 percent 

Fill factor is an index‐level setting that specifies the percentage of each leaf-level page to fill with data when an index is created or rebuilt. 
- A fill factor of 90 means SQL Server leaves 10 percent of each page empty for future growth, reducing page splits on inserts and updates 
- Lowering fill factor can improve write performance at the cost of increased storage and potentially lower read density 

Verify that fragmentation has been reduced
```sql
SELECT OBJECT_NAME(ips.object_id) AS TableName, i.name AS IndexName, ips.index_type_desc, ips.avg_fragmentation_in_percent
FROM sys.dm_db_index_physical_stats(DB_ID(), NULL, NULL, NULL, 'LIMITED') AS ips
 JOIN sys.indexes AS i ON ips.object_id = i.object_id AND ips.index_id = i.index_id
WHERE OBJECT_NAME(ips.object_id)='SalesOrderDetail';
```

• `LIMITED` scans only a subset of pages per index for a fast, lightweight estimate
• `SAMPLED` examines a sample of pages for a balance of speed and accuracy
• `DETAILED` reads every page for the most accurate result but can be slow and resource‑intensive

#### Final Thought 
Regular index maintenance—using the right operation and fill factor—keeps your indexes healthy, minimizes I/O, and avoids unexpected performance drops. By combining physical stats with usage data, you focus effort where it delivers the greatest benefit.

------------------------- ------------------------- ------------------------- -------------------------

### Quiz

1. Which dynamic management view provides the list of columns your query filters on for missing‑index suggestions? 
 A) `sys.dm_exec_query_stats` 
 B) `sys.dm_db_missing_index_details` 
 C) `sys.dm_db_index_physical_stats` 
 D) `sys.dm_db_index_usage_stats` 

2. In the missing‑index Dynamic Management View output, which field shows columns that should be included to cover the query? 
 A) `equality_columns` 
 B) `inequality_columns` 
 C) `included_columns` 
 D) `avg_total_user_cost` 

3. What T‑SQL command turns on I/O and time reporting so you can compare logical reads before and after indexing? 
 A) `SET STATISTICS PROFILE ON` 
 B) `SET SHOWPLAN_ALL ON` 
 C) `SET STATISTICS IO, TIME ON` 
 D) `SET STATISTICS XML ON` 

4. After creating the recommended nonclustered index on `UnitPrice`, which change should you observe in the execution plan? 
 A) a Clustered Index Scan becomes a Table Scan 
 B) an Index Scan becomes an Index Seek 
 C) a Hash Match becomes a Nested Loops 
 D) a Merge Join becomes a Hash Join 

5. Which Dynamic Management Views helps you measure the percentage of page fragmentation for each index? 
 A) `sys.dm_db_index_usage_stats` 
 B) `sys.dm_db_index_physical_stats` 
 C) `sys.dm_exec_query_memory_grants` 
 D) `sys.dm_os_wait_stats` 

6. According to best practice, you should rebuild an index when fragmentation exceeds what threshold? 
 A) 5 percent 
 B) 10 percent 
 C) 30 percent 
 D) 50 percent 

7. Which Dynamic Management Views reveals how often an index is used for seeks, scans, and updates? 
 A) `sys.dm_db_missing_index_group_stats` 
 B) `sys.dm_db_index_physical_stats` 
 C) `sys.dm_db_index_usage_stats` 
 D) `sys.dm_exec_cached_plans` 

8. To reorganize lightly fragmented indexes (5–30 percent), which command would you run? 
 A) `ALTER INDEX ALL ON ... REBUILD` 
 B) `ALTER INDEX ALL ON ... REORGANIZE` 
 C) `DBCC INDEXDEFRAG` 
 D) `DBCC SHRINKFILE` 

<!-- ------------------------- ------------------------- -->

#### Answers

1. **B** – `sys.dm_db_missing_index_details` 
 *This Dynamic Management Views lists the columns your queries filter on for missing‑index recommendations* 

2. **C** – `included_columns` 
 *It specifies which non‑key columns to include so the index covers the query* 

3. **C** – `SET STATISTICS IO, TIME ON` 
 *This command enables detailed I/O and duration metrics for each statement* 

4. **B** – an Index Scan becomes an Index Seek 
 *Adding the index lets the optimizer seek directly to matching rows instead of scanning* 

5. **B** – `sys.dm_db_index_physical_stats` 
 *It reports fragmentation metrics like `avg_fragmentation_in_percent` for each index* 

6. **C** – 30 percent 
 *Indexes over 30 percent fragmented are best rebuilt to restore page order* 

7. **C** – `sys.dm_db_index_usage_stats` 
 *This Dynamic Management Views shows user_seeks, user_scans, and user_updates for each index* 

8. **B** – `ALTER INDEX ALL ON ... REORGANIZE` 
 *REORGANIZE defragments pages in place without rebuilding the entire index* 

<!-- ------------------------- ------------------------- ------------------------- ------------------------- -->
<!-- ------------------------- ------------------------- ------------------------- ------------------------- -->

## Azure

### Exercise: Automatic Tuning 
Leverage Azure's built‑in tuning engine to apply and monitor index and plan corrections without manual intervention.

<!-- ------------------------- ------------------------- -->

Navigate to Azure Portal >> SQL Server >> Intelligent Performance >> Automatic Tuning

<img src="..\images\SQL_PerformanceOptimization\AutomaticTuning.png" width="800" title="Snipped April, 2025" />

Review features:
- **Inheritance From**: allows you to inherit tuning settings from server‑level Azure defaults or override them at the database level 
- **FORCE PLAN**: automatically applies a known‑good execution plan when a new plan degrades performance 
- **CREATE INDEX**: generates and **implements missing‑index recommendations** based on real workload telemetry 
- **DROP INDEX**: **removes unused or low‑value indexes** to reduce storage and maintenance overhead 

Turn on all options and click "Apply".

#### Final Thought 
Automatic Tuning in Azure SQL can offload routine index and plan corrections, letting you focus on higher‑value tasks. Always review recommendations to ensure they align with your workload patterns and maintenance windows. 

<!-- ------------------------- ------------------------- ------------------------- ------------------------- -->

### Quiz

1. Which portal path leads you to the Automatic Tuning settings for a given database? 
 A) Azure Monitor → Logs 
 B) SQL Databases → Intelligent Performance → Automatic tuning 
 C) SQL Databases → Firewall settings 
 D) SQL Servers → Virtual Network configuration 

2. What does the **Force Plan** option do? 
 A) Automatically generates missing‑index recommendations 
 B) Reverts to a known‑good execution plan when a regression is detected 
 C) Drops unused indexes 
 D) Updates statistics asynchronously 

3. What is the purpose of the **Create Index** setting? 
 A) Removes indexes not used recently 
 B) Forces queries to ignore indexes 
 C) Automatically creates suggested nonclustered indexes based on real‑time telemetry 
 D) Forces execution plans to be recompiled each time 

4. After toggling Automatic Tuning options in the portal, what must you click to apply your changes? 
 A) Save 
 B) Apply 
 C) Refresh 
 D) Restart Database 

5. The **Inheritance From** setting in Automatic Tuning allows you to: 
 A) Inherit settings from a database master key 
 B) Inherit server‑level tuning defaults or override them at the database level 
 C) Inherit data from a primary database 
 D) Inherit firewall rules 

#### Answers

1. **B** – `SQL Databases → Intelligent Performance → Automatic tuning`  
 *Opens the Automatic Tuning settings for the selected database* 

1. **B** – `Force Plan` reverts to the last known good execution plan when a regression is detected  
 *Maintains query performance by automatically applying a stable plan* 

1. **C** – `Create Index` automatically creates recommended nonclustered indexes based on workload telemetry  
 *Reduces manual index maintenance by implementing high‑impact index suggestions* 

1. **B** – `Apply` commits your Automatic Tuning changes  
 *Activates the toggled settings so Automatic Tuning begins using your configuration* 

1. **B** – `Inheritance From` lets the database inherit server‑level tuning defaults or override them  
 *Allows centralized or per‑database control of Automatic Tuning options* 