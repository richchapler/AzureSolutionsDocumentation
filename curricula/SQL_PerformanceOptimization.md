# SQL: Performance Optimization

## Use Case

The database team at a mid‑sized company is under pressure... **data volumes are growing fast**, **users are complaining about slow queries**, and **critical reports are missing their deadlines**. Leadership wants answers and assigns a mission: uncover what's slowing things down, tune performance across the system, and make sure it does not happen again.

The database team talks about the mission and settles on the following goals:

- **Diagnose and fix slow queries** before users notice
- **Speed up data ingestion** so pipelines finish on time
- **Keep performance stable** even as load increases
- Eliminate guesswork through **clear, data-driven tuning**
- **Proactively optimize**, not just react to problems

<!-- ------------------------- ------------------------- ------------------------- ------------------------- -->

## Fundamentals

1. **"Execution Plan"**: read estimated and actual plans to understand how SQL Server breaks down your queries 
2. **"Live Query Statistics"**: monitor queries as they run to catch slow or blocked operations in real time 
3. **Logic Optimization**: rewrite queries for set‑based processing, eliminate unnecessary work, and leverage indexes 
4. **Query‑Level Hints**: use hints to guide the optimizer when you need more control 
5. **Query Store**: collect and compare execution plans over time to prevent regressions

<!-- ------------------------- ------------------------- -->
<!-- ------------------------- ------------------------- -->

### "Execution Plan"
...reveals how the SQL engine interprets and executes a query, enabling developers and database professionals to identify bottlenecks and improve performance 

- **Estimated Execution Plan**: displays how SQL Server plans to run a query before it is executed; helps spot inefficient join orders, missing indexes, or inaccurate estimates without actually modifying data 
- **Actual Execution Plan**: includes details collected during query execution, such as actual row counts and time spent on each operation; essential for finding real-world performance problems and mismatches between estimated and actual costs 
- **Live Query Statistics**: offers a visual, real-time look at how a query is progressing as it runs; useful for troubleshooting slow or blocked queries 
- **Execution Plan Operators**: provides insight into the individual steps of a query, such as Nested Loops, Hash Match, and Sort; recognizing these can help you understand why a query might be slow 
- **Plan Comparison**: allows you to compare execution plans **before and after changes** and helps confirm that: 1) a performance issue is fixed or 2) spot regressions after a change 

**Use this when...** you need to pinpoint inefficient operators, missing indexes, or estimate inaccuracies **before and after changes**

<img src=".\images\SQL_PerformanceOptimization\ActualExecutionPlan.png" width="800" title="Snipped April, 2025" />

<!-- ------------------------- ------------------------- -->
<!-- ------------------------- ------------------------- -->

### "Live Query Statistics"
...gives a live, visual view of query progress while it's running 

- **Real-Time Insight**: shows operator-level progress and row counts during query execution 
- **Early Detection**: helps identify where a query is stalled, blocked, or slower than expected 
- **Best For**: troubleshooting long-running or interactive queries in development and test environments

**Use this when...** you need to spot stalls, blocking, or unexpectedly slow operators **in real time**

<img src=".\images\SQL_PerformanceOptimization\LiveQueryStatistics.png" width="800" title="Snipped April, 2025" />

<!-- ------------------------- ------------------------- -->
<!-- ------------------------- ------------------------- -->

### Logic Optimization
...involves modifying the structure of a query to improve performance without changing the result

-------------------------

#### Set‑Based Processing
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

#### Eliminate Unnecessary Function Use 
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

#### Bake Filters into Joins 
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

#### Flatten Sub-Queries 
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

#### Reduce Column Retrieval 
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

#### "Sargable" Conditions
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

#### Simplify Predicates 
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

#### Eliminate Redundant Joins 
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

#### Use UNION ALL 
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

#### Proactive Monitoring 
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

#### Forced Parameterization
...is a database‑level setting that tells SQL Server to replace literal values in your ad hoc queries with parameters at compile time so they can share execution plans.

By default, SQL Server uses **Simple Parameterization** only for very basic literals and leaves most ad hoc queries unparameterized.

When you set the database to **Forced Parameterization** every literal that can be safely parameterized is turned into a parameter.

This increases plan reuse and reduces plan cache bloat at the cost of occasionally generating less‑optimal plans.

**Before**

```sql
-- each literal produces its own plan 
SELECT * 
 FROM Sales.SalesOrderHeader 
 WHERE OrderDate = '2021-01-01';

SELECT * 
 FROM Sales.SalesOrderHeader 
 WHERE OrderDate = '2021-02-01';
``` 

**After**

```sql
ALTER DATABASE [YourDatabase] SET PARAMETERIZATION = FORCED;
GO;

-- SQL Server automatically rewrites to a parameterized form 
EXEC sp_executesql 
 N'SELECT * FROM Sales.SalesOrderHeader WHERE OrderDate = @date', 
 N'@date date', 
 @date = '2021-01-01';

EXEC sp_executesql 
 N'SELECT * FROM Sales.SalesOrderHeader WHERE OrderDate = @date', 
 N'@date date', 
 @date = '2021-02-01';
``` 

**Now both executions share a single cached plan instead of two separate ones.**

-------------------------

#### Optimize for Adhoc Workloads

**By default, SQL Server caches the complete execution plan on the first run of any query, even if it never repeats.**

When you enable **Optimize for Adhoc Workloads**, SQL Server saves just a stub initially and promotes it to a full plan only upon a subsequent execution.

This **reduces plan cache bloat** and **conserves memory** at the cost of delaying full plan caching until the query is reused.

```sql
EXEC sp_configure 'show advanced options', 1; 
RECONFIGURE;

EXEC sp_configure 'optimize for adhoc workloads', 1; 
RECONFIGURE;
``` 

-------------------------

### Query-Level Hints
Use hints to override the optimizer when its default plan doesn’t meet your performance needs.

### Join

| Hint | Impact |
| :--- | :--- |
| `OPTION (LOOP JOIN)` | Forces a nested‑loops join; efficient for small, highly selective inputs |
| `OPTION (HASH JOIN)` | Forces a hash join; optimal for large, unsorted equi‑joins |
| `OPTION (MERGE JOIN)` | Forces a merge join; streams sorted inputs with low memory overhead |
| `OPTION (FORCE ORDER)` | Preserves the join order you wrote; useful when optimizer’s order is suboptimal |

### Index

| Hint | Impact |
| :--- | :--- |
| `WITH (FORCESEEK(IndexName))` | Forces an index seek on the specified index for selective filters |
| `WITH (FORCESCAN)` | Forces a full scan of the clustered or specified index |
| `WITH (INDEX(index_list))` | Restricts optimizer to only the specified index(es) |

### Parallelism

| Hint | Impact |
| :--- | :--- |
| `OPTION (MAXDOP n)` | Limits degree‑of‑parallelism for the query, balancing CPU utilization and throughput |
| `OPTION (FAST n)` | Optimizes plan to return the first *n* rows quickly, useful for interactive or paginated queries |

### Plan Stability

| Hint | Impact |
| :--- | :--- |
| `OPTION (OPTIMIZE FOR UNKNOWN)` | Builds a generic plan ignoring the first parameter, balancing cost across inputs |
| `OPTION (RECOMPILE)` | Discards cached plan and compiles a fresh plan each execution, avoiding sniffing |

### Plan‑Cache

| Hint | Impact |
| :--- | :--- |
| `OPTION (OPTIMIZE FOR AD HOC WORKLOADS)` | Caches only a stub on first run of ad hoc queries, reducing cache bloat |
| `OPTION (KEEP PLAN)` | Prevents eviction under memory pressure, ensuring quick reuse |
| `OPTION (KEEPFIXED PLAN)` | Pins plan across stats/index changes or upgrades |

### Concurrency

| Hint | Impact |
| :--- | :--- |
| `WITH (NOLOCK)` | Reads without shared locks (READ UNCOMMITTED), reducing blocking |
| `WITH (READPAST)` | Skips locked rows to avoid blocking on heavily contended tables |
| `WITH (UPDLOCK)` | Acquires update locks on read to prevent deadlocks when planning updates |
| `WITH (ROWLOCK)` | Forces row‑level locks only, avoiding broader lock escalation |

<!-- ------------------------- ------------------------- -->
<!-- ------------------------- ------------------------- -->

### Query Store Usage 
...lets you track, compare, and manage query performance over time 

- **Captures Execution History**: stores actual execution plans, query text, and performance statistics 
- **Detects Regressions**: identifies when a new plan performs worse than a previous one 
- **Forces Stable Plans**: allows you to pin a known‑good plan for a given query to prevent further regressions 
- **Configuration Settings**: tune key Query Store parameters—operation mode (`READ_WRITE` vs. `READ_ONLY`), maximum size (`MAX_STORAGE_SIZE_MB`), data flush interval, and statistics collection interval—to maintain continuous capture and avoid automatic read‑only transitions 
- **Useful in Azure and On‑Premises**: enabled by default in Azure SQL; optional but highly recommended in SQL Server 

Execute the following T-SQL to activate:
```sql
ALTER DATABASE AdventureWorks2022 SET QUERY_STORE = ON (OPERATION_MODE = READ_WRITE, MAX_STORAGE_SIZE_MB = 100, INTERVAL_LENGTH_MINUTES = 30)
```

<img src=".\images\SQL_PerformanceOptimization\QueryStore.png" width="800" title="Snipped April, 2025" />

<!-- ------------------------- ------------------------- -->
<!-- ------------------------- ------------------------- -->
<!-- ------------------------- ------------------------- -->
<!-- ------------------------- ------------------------- -->
<!-- ------------------------- ------------------------- -->
<!-- ------------------------- ------------------------- -->
<!-- ------------------------- ------------------------- -->
<!-- ------------------------- ------------------------- -->

### Automatic Tuning 
...automates some performance tuning decisions based on collected telemetry 

- **Automatic Plan Correction**: reverts to a previously good plan if a new one degrades performance 
- **Automatic Indexing**: in Azure SQL, evaluates and applies index create/drop recommendations 
- **Customizable Options**: can be configured to suggest changes, apply them automatically, or require manual approval 
- **Azure Advantage**: only available in Azure SQL; not present in on-prem SQL Server 

<!-- ------------------------- ------------------------- -->
<!-- ------------------------- ------------------------- -->

### Indexing and Storage Design

#### Index Strategies 
...improve query performance by allowing faster data access and more efficient execution plans 

- **Clustered Indexes**: determine the physical order of rows in a table; best for range queries and columns frequently used in ORDER BY 
- **Nonclustered Indexes**: separate from the base table; ideal for columns frequently filtered or joined on 
- **Filtered Indexes**: index only a subset of rows; useful when a query targets a predictable WHERE clause (e.g., `IsActive = 1`) 
- **Covering Indexes**: include all columns needed by a query, eliminating the need to access the base table (avoids lookups) 
- **Design Tip**: avoid over-indexing—each index has a write penalty; index based on workload, not guesswork 

<!-- ------------------------- ------------------------- -->
<!-- ------------------------- ------------------------- -->

#### Data Type Tuning 
...ensures that column definitions align with actual data usage to reduce memory and storage costs 

- **Right-Sizing Types**: use `int` instead of `bigint`, `date` instead of `datetime`, and `money` instead of `decimal(18,4)` when appropriate 
- **Impact on Performance**: smaller data types mean smaller indexes, more rows per page, and less memory usage in query execution 
- **String Optimization**: avoid using `nvarchar(max)` or `varchar(max)` unless necessary; define appropriate lengths 
- **Precision Control**: for financial or statistical data, ensure the scale and precision match business requirements—**DON'T** default to overly wide types 
- **Schema Discipline**: enforce consistent type usage across tables to simplify query writing and avoid implicit conversions 

<!-- ------------------------- ------------------------- -->
<!-- ------------------------- ------------------------- -->

#### Statistics Management 
...provides the optimizer with data distribution estimates to make better plan decisions 

- **Column Statistics**: track the distribution of values within indexed and non-indexed columns 
- **Why It Matters**: outdated statistics can cause the optimizer to misjudge row counts and pick inefficient plans 
- **Auto Update**: SQL Server updates statistics automatically, but large changes in data or complex queries may require manual refresh 
- **Manual Update**: use `UPDATE STATISTICS` or `sp_updatestats` after bulk loads or major data changes 
- **Best Practice**: monitor for queries with poor estimates (e.g., large discrepancies between estimated and actual rows) 

<!-- ------------------------- ------------------------- -->
<!-- ------------------------- ------------------------- -->

#### Histograms and Cardinality Estimation 
...ensures the optimizer can accurately predict row counts so it picks efficient plans 

- **Histogram Buckets**: up to 200 equal‑height steps per column showing value distribution; used to estimate how many rows match a filter 
- **Density Vectors**: stored multi‑column statistics that help estimate selectivity of combined predicates 
- **Cardinality Estimator Versions**: SQL Server compatibility levels 120+ use the modern estimator with better handling of multi‑column correlations; legacy estimator still available via database compatibility setting 
- **Sampling and Full Scan**: control accuracy with `FULLSCAN` for precise stats or sampled scans for speed; check last update with `STATS_DATE` 
- **Impact on Plans**: mismatches between estimated and actual row counts lead to poor join choices and inappropriate memory grants 
- **Azure SQL**: always uses the latest estimator by default, but you can alter behavior via the database compatibility level if you need to match on‑prem behavior 

<!-- ------------------------- ------------------------- -->
<!-- ------------------------- ------------------------- -->

#### Missing Index Recommendations 
...leverages built‑in dynamic management views to identify index candidates based on workload patterns 

- **Key DMVs**: 
 - `sys.dm_db_missing_index_details` shows which indexes could help specific queries 
 - `sys.dm_db_missing_index_groups` and `sys.dm_db_missing_index_group_stats` group related suggestions and provide impact metrics 
- **Important Fields**: 
 - `equality_columns` and `inequality_columns` for filter predicates 
 - `included_columns` for covering queries 
 - `avg_user_impact` and `user_seeks`/`user_scans` to prioritize high‑value recommendations 
- **Validation**: load test or use Query Store to confirm expected benefit before creating a new index 
- **Limitations**: DMV suggestions can overlap or not account for multi‑column correlations; always review schema and workload 
- **Platform Notes**: 
 - **SQL Server (On‑Premises)**: query the DMVs directly in SSMS or include in maintenance scripts 
 - **Azure SQL**: Performance Recommendations in the portal also surface missing‑index advice, backed by Query Store data 

<!-- ------------------------- ------------------------- -->
<!-- ------------------------- ------------------------- -->

#### Index Maintenance and Fragmentation 
...ensures your indexes remain efficient over time by monitoring and correcting physical fragmentation 

- **Fragmentation Metrics**: track fragmentation levels with `sys.dm_db_index_physical_stats` 
- **Rebuild vs. Reorganize**: rebuild for > 30 percent fragmentation, reorganize for 5–30 percent 
- **Online vs. Offline**: choose online rebuilds for minimal blocking in Enterprise/Azure tiers, offline when maintenance win**DO**ws allow 
- **Fill Factor and Page Splits**: 
 - **Fill Factor**: sets the percentage of page fullness on rebuild (e.g. 80–90 percent) to leave free space for growth 
 - **Page Splits**: happen when a page is full and must split to accommodate inserts, causing extra I/O and fragmentation 
 - **Trade‑Offs**: lower fill factor reduces splits but increases storage and may slightly slow reads; adjust based on update frequency 
- **Maintenance Scheduling**: automate via SQL Agent or Azure Automation to keep fragmentation and splits in check without manual intervention 

<!-- ------------------------- ------------------------- -->
<!-- ------------------------- ------------------------- -->

#### Row and Page Compression 
...reduces the storage footprint of rowstore tables without altering schema or indexing 

- **Row Compression**: eliminates unnecessary space in fixed-length data types (e.g., `char`, `int`) 
- **Page Compression**: combines row compression with prefix and dictionary compression within data pages 
- **Performance Tradeoff**: reduces I/O but adds slight CPU cost for compression and decompression 
- **Best Fit**: transactional systems where storage and I/O savings outweigh minor CPU overhead 
- **Usage Tip**: use `sp_estimate_data_compression_savings` to preview potential gains before enabling 

<!-- ------------------------- ------------------------- -->
<!-- ------------------------- ------------------------- -->

#### Columnstore and Archival Compression 
...compress data vertically across columns for analytic workloads that scan large data sets 

- **Columnstore Indexes**: ideal for fact tables with billions of rows; optimized for aggregation and scanning, not row-by-row access 
- **Clustered vs. Nonclustered**: clustered columnstore replaces the base table; nonclustered overlays a rowstore 
- **Archival Compression**: adds an extra layer of compression for cold data, saving space at the cost of CPU during decompression 
- **Azure Benefit**: columnstore and archival compression are fully supported and often recommended for storage-optimized Azure SQL configurations 
- **When to Use**: data warehouse scenarios, telemetry, and history tables with high read-to-write ratios 

<!-- ------------------------- ------------------------- -->
<!-- ------------------------- ------------------------- -->

#### File and Filegroup Placement Strategies 
...ensures optimal I/O throughput and manageability by distributing data and log files across storage volumes 

- **Filegroups**: group related tables and indexes into named containers to control where data lives and simplify backups 
- **Data File Placement**: spread data files across separate disks or storage accounts to balance I/O and avoid hotspots 
- **Log File Isolation**: place transaction log files on dedicated, low‑latency storage to speed commits and reduce contention 
- **TempDB Separation**: use its own filegroup or even separate storage to keep temporary object I/O from impacting user databases 
- **Design Tip**: align filegroup placement with workload characteristics (e.g. put large, read‑heavy tables on faster media) 

<!-- ------------------------- ------------------------- -->
<!-- ------------------------- ------------------------- -->

### Parallelism, TempDB, and Resource Management

#### Parallelism Tuning 
...controls how many CPUs the SQL engine can use to execute a query in parallel 

- **MAXDOP (Maximum Degree of Parallelism)**: limits the number of CPUs used for a single query; too high can cause CPU thrashing, too low can underutilize the system 
- **Cost Threshold for Parallelism**: determines when a query qualifies for parallel execution; default is often too low for modern systems 
- **Query-Level Overrides**: use hints like `OPTION (MAXDOP 1)` for queries that **DON'T** benefit from parallelism 
- **Symptoms of Poor Tuning**: high CXPACKET waits, unpredictable CPU spikes, and excessive context switching 
- **Best Practice**: balance system-wide settings with per-query overrides; monitor with `sys.dm_exec_requests` and `sys.dm_os_wait_stats` 

<!-- ------------------------- ------------------------- -->
<!-- ------------------------- ------------------------- -->

#### TempDB Optimization 
...reduces contention and blocking in the shared temporary workspace used by all queries 

- **Multiple Data Files**: configure 1 data file per logical CPU (up to 8) to avoid page latch contention 
- **Equal Size and Growth**: ensure all TempDB files are the same size with the same autogrowth settings 
- **Location and I/O**: place TempDB on fast storage; isolate it from user databases if possible 
- **Monitoring**: track TempDB usage with DMVs like `sys.dm_db_file_space_usage` and `sys.dm_db_session_space_usage` 
- **Workload Impact**: high concurrency environments (e.g., heavy reporting or temp tables) suffer most from poor TempDB setup 

<!-- ------------------------- ------------------------- -->
<!-- ------------------------- ------------------------- -->

#### Memory and Plan Cache Behavior 
...affects overall performance by controlling how SQL Server allocates memory and reuses query plans 

- **Plan Cache Bloat**: caused by many unique, single-use queries; wastes memory and slows down lookup 
- **Memory Grants**: each query requests memory before executing—overestimation can delay others, underestimation leads to spills to disk 
- **Single-Use Plan Detection**: use `sys.dm_exec_cached_plans` to find bloated caches 
- **Spill Warnings**: visible in execution plans when sorts or hashes exceed memory and write to TempDB 
- **Tuning Tools**: consider `OPTIMIZE_FOR_AD_HOC_WORKLOADS` and review execution memory in `sys.dm_exec_query_memory_grants` 

<!-- ------------------------- ------------------------- -->
<!-- ------------------------- ------------------------- -->

#### Memory Grant Management and Spill Analysis 
...ensures queries receive the right amount of memory and avoids costly disk spills during execution 

- **Memory Grants**: queries request a specific amount of workspace memory before running; insufficient grants can queue or throttle execution 
- **Spill Indicators**: in actual execution plans, look for warnings on Sort or Hash operators that spill to TempDB when they exceed granted memory 
- **DMVs for Insight**: query `sys.dm_exec_query_memory_grants` to see requested, granted, and ideal grant sizes, plus wait times 
- **Tuning Strategies**: reduce memory demands by filtering earlier, rewriting heavy operators, updating statistics for better estimates, or increasing server memory 
- **Azure SQL Notes**: memory grant behavior is tied to your service tier; monitor grants via Query Store or `sys.resource_stats` and scale up/off as needed 

<!-- ------------------------- ------------------------- -->
<!-- ------------------------- ------------------------- -->

#### Resource Governance 
...ensures that one workload does not overwhelm others by controlling CPU, memory, and I/O allocations 

- **Resource Governor (on-premises only)**: create workload groups and assign limits based on user, query type, or application role 
- **Azure Tiers**: in Azure SQL, pick vCore or DTU tiers based on workload shape (transactional, analytical, mixed) 
- **Elastic Pools**: share resources across multiple databases with defined limits on CPU and memory per pool 
- **Throttling Symptoms**: increased latency, query timeouts, and IO waits under peak load 
- **Best Fit**: multi-tenant apps, mixed workload environments, and any system where noisy neighbors can impact critical queries 

<!-- ------------------------- ------------------------- -->
<!-- ------------------------- ------------------------- -->

### Ingestion and Batch Optimization

#### Bulk Load Tuning 
...improves the speed and efficiency of large data insert operations by minimizing overhead 

- **Minimal Logging**: enabled by using `TABLOCK` and bulk insert operations in `SIMPLE` or `BULK_LOGGED` recovery models to reduce transaction log usage 
- **Batching**: break large inserts into smaller chunks to avoid excessive memory usage and reduce locking 
- **Table Hints**: apply options like `TABLOCK`, `CHECK_CONSTRAINTS OFF`, and `FIRE_TRIGGERS OFF` (where appropriate) to reduce runtime 
- **Transaction Log Throughput**: monitor write speed and ensure the log file is pre-sized and on fast storage 
- **Use Case**: nightly ETL jobs, historical data migrations, and staged imports where speed outweighs transactional guarantees 

<!-- ------------------------- ------------------------- -->
<!-- ------------------------- ------------------------- -->

#### Data Ingestion Pipeline Tuning 
...focuses on optimizing external data movement into SQL using scalable, cloud-native tools 

- **Azure Data Factory (ADF)**: adjust parallelism settings, use `COPY INTO`, and enable compression for faster ingestion 
- **PolyBase**: enables parallel imports from flat files or external sources; best used for structured files (CSV, Parquet) 
- **COPY INTO (Azure SQL)**: optimized for bulk inserts in cloud environments, especially when staging via Azure Blob Storage 
- **Performance Tactics**: push computation to source where possible, avoid row-by-row inserts, and monitor throughput metrics 
- **Best Fit**: cloud-native ingestion from data lakes, warehouses, or external systems feeding Azure SQL 

<!-- ------------------------- ------------------------- -->
<!-- ------------------------- ------------------------- -->

#### In-Memory Optimization 
...uses special table types and compiled procedures to remove traditional engine bottlenecks in high-performance OLTP systems 

- **Memory-Optimized Tables**: store rows in memory rather than disk; eliminate locking, latching, and logging overhead 
- **Natively Compiled Procedures**: execute with minimal CPU cycles and no interpretation; ideal for highly repeatable, high-volume logic 
- **Durability Options**: choose between schema-only (non-durable) and schema-and-data (durable) for trade-offs between speed and persistence 
- **Deployment Constraints**: only available in certain SQL Server editions and Azure SQL Database configurations 
- **Use Case**: extreme transaction throughput requirements, such as financial systems, telemetry, and gaming platforms 

<!-- ------------------------- ------------------------- -->
<!-- ------------------------- ------------------------- -->

### Concurrency and Isolation Controls

#### Blocking and Deadlock Mitigation 
...prevents sessions from blocking each other or getting stuck in deadlocks by controlling resource access and query behavior 

- **Blocking**: occurs when one query holds a lock that another query needs; common with long-running transactions or unindexed lookups 
- **Deadlocks**: two or more queries wait on each other in a cycle—SQL Server will automatically terminate one to break the cycle 
- **Query Shaping**: reduce lock duration by keeping transactions short, avoiding user interaction inside transactions, and indexing filter columns 
- **Isolation Levels**: use `READ COMMITTED` for default behavior, or lower levels like `READ UNCOMMITTED` or `SNAPSHOT` to reduce contention where data staleness is acceptable 
- **Indexing for Concurrency**: properly placed indexes reduce the number of rows touched, which reduces locking footprint 

<!-- ------------------------- ------------------------- -->

#### Read-Consistency Tuning 
...ensures that readers **DON'T** block writers and vice versa, improving concurrency without rewriting application logic 

- **READ_COMMITTED_SNAPSHOT**: enables version-based row reads using tempdb so readers **DON'T** block updates and updates **DON'T** block readers 
- **How It Works**: when enabled, SQL Server keeps a copy of the original row in tempdb so SELECT queries can read consistent data even if a write is in progress 
- **Impact**: greatly improves concurrency in read-heavy systems without needing to change isolation level on every query 
- **Tradeoffs**: uses more tempdb space and slightly more CPU but avoids reader/writer contention in most OLTP workloads 
- **Best Fit**: high-concurrency applications with mixed read/write activity, like customer portals or transactional APIs 

<!-- ------------------------- ------------------------- -->
<!-- ------------------------- ------------------------- -->

### Observability and Diagnostics

#### Monitoring and Baselining 
...builds a reference point for expected system behavior so performance issues can be detected early and diagnosed accurately 

- **Query Store**: tracks query text, plans, and runtime statistics over time—ideal for identifying regressions and tuning opportunities 
- **Azure Monitor**: provides platform-level telemetry across CPU, memory, I/O, and query duration; integrates with alerting and diagnostics 
- **Dynamic Management Views (DMVs)**: expose real-time engine internals like memory usage, wait statistics, and plan cache contents 
- **Baselining Strategy**: monitor key metrics during normal operations so future anomalies can be quickly identified as outliers 
- **Best Practice**: establish time-based and workload-based baselines for both user activity and system resource consumption 

<!-- ------------------------- ------------------------- -->

#### Wait Statistics Analysis 
...uses built‑in wait metrics to pinpoint where queries spend time waiting for resources 

- **sys.dm_os_wait_stats**: view cumulative wait times by wait type to identify the most costly waits 
- **Key Wait Types**: recognize common waits like `PAGEIOLATCH_*` (I/O), `CXPACKET` (parallelism), and `LCK_M_*` (locking) 
- **Wait Time Breakdown**: correlate wait stats with workload patterns and time‑of‑day to target tuning efforts 
- **Filtering and Baselines**: ignore known benign waits (e.g. `SQLTRACE_BUFFER_FLUSH`) and compare against historical baselines 
- **Actionable Insights**: use waits to decide if you need more memory, faster storage, index changes, or isolation‑level adjustments 

<!-- ------------------------- ------------------------- -->

#### Lock and Latch Wait Analysis 
...distinguishes between user‑visible lock contention and internal latch blocking, including NUMA and parallelism effects 

- **Lock Waits (`LCK_M_*`)**: indicate sessions blocked waiting for locks on rows, pages, or tables; investigate blockers via `sys.dm_tran_locks` and `sys.dm_exec_requests` 
- **Latch Waits (`PAGELATCH_*`, `BUFFERIOPENDING`)**: occur when internal engine structures (like Buffer Pool pages) are contended; often seen in TempDB under heavy load 
- **NUMA Node Impact**: on‑premises servers with multiple NUMA nodes can suffer cross‑node memory access waits; monitor with `sys.dm_os_ring_buffers` or Win**DO**ws Performance Counters 
- **Parallelism Waits (`CXPACKET`, `CXCONSUMER`)**: reflect parallel thread coordination overhead; correlate with MAXDOP settings to balance CPU use and minimize skew 
- **Mitigation Strategies**: add or refine indexes, tune isolation levels, configure appropriate MAXDOP, isolate workloads with Resource Governor (on‑prem), and optimize TempDB file layout 
- **Platform Notes**: 
 - **SQL Server On‑Premises**: full control over NUMA affinity, Resource Governor, and low‑level storage settings 
 - **Azure SQL**: abstracts NUMA; tune MAXDOP via database scoped configurations and rely on service tiers for resource governance 

<!-- ------------------------- ------------------------- -->

#### Extended Events and Trace Sessions 
...captures detailed, event‑driven diagnostics for deep performance and behavioral analysis 

- **Extended Events**: lightweight, scalable event framework replacing SQL Profiler; use `CREATE EVENT SESSION` to capture events like `query_post_execution_showplan`, `wait_info`, and `deadlock_graph` 
- **Targets**: write to ring buffer for quick inspection or file targets for long‑term storage and offline analysis 
- **Filtering Early**: apply predicates in your session definition to limit overhead (for example, filter by database_id or duration) 
- **SQL Trace / Profiler**: legacy tracing tool—support being deprecated in favor of Extended Events; use only when compatibility requires it 
- **Use Cases**: capture execution plans on demand, track custom wait types, record parameter values, and diagnose rare deadlocks 

- **SQL Server (On-Premises)**: define and start sessions via T-SQL or SSMS UI; manage retention and cleanup with built‑in procedures 
- **Azure SQL**: use Azure Monitor diagnostic settings to stream Extended Events to Log Analytics or storage; limited SSMS session support but queryable via DMVs like `sys.fn_xe_file_target_read_file` 

<!-- ------------------------- ------------------------- -->

#### Alert Tuning 
...filters meaningful events from noise so that alerts signal real performance problems 

- **Static vs. Dynamic Thresholds**: dynamic thresholds adapt to baseline behavior; better for environments with variable workloads 
- **Threshold Sensitivity**: lower sensitivity reduces false positives but may miss subtle trends; tune based on system criticality 
- **Metric Selection**: prioritize alerts on CPU, DTU/vCore usage, query duration, and failed logins over less actionable events 
- **Aggregation Win**DO**ws**: configure alert frequency to avoid alert storms during short-lived spikes 
- **Azure Integration**: use Azure Monitor and Log Analytics to centralize alert rules, actions, and historical alert trends 

<!-- ------------------------- ------------------------- -->

#### Execution History and Telemetry 
...reveals recent query behavior and performance without needing to capture a live session 

- **Lightweight Query Profiling**: enables low-overhead tracking of operator-level performance metrics during query execution 
- **Last Query Plan Stats**: exposes the plan and statistics from the most recent execution of a query 
- **When to Use**: investigate recent regressions or spikes in resource usage without enabling full trace or extended events 
- **System Views**: access via `sys.dm_exec_query_stats`, `sys.dm_exec_requests`, and `sys.dm_exec_query_profiles` 
- **Best Fit**: adhoc troubleshooting, post-mortem analysis, and interactive workloads where full profiling is too heavy 

<!-- ------------------------- ------------------------- -->

#### Workload Classification 
...categorizes queries so you can apply targeted tuning and resource allocations 

- **adhoc Queries**: one‑off or unpredictable queries; optimize individually and control plan cache impact with `OPTIMIZE_FOR_AD_HOC_WORKLOADS` 
- **Prepared/Parameterized**: repeated queries with different parameters; benefit from plan reuse and require parameter‑sniffing strategies 
- **Batch and ETL Jobs**: scheduled, high‑volume operations; tune using bulk‑load techniques, minimize logging, and isolate via Resource Governor or separate compute 
- **Reporting and Analytical**: long‑running scans and aggregations; optimize with columnstore indexes, compression, and appropriate distribution of compute 
- **Resource Allocation**: assign each class to its own workload group (on‑premises Resource Governor) or elastic pool/SLO tier (Azure SQL) 

- **Platform Notes**: 
 - **SQL Server On‑Premises**: use Resource Governor to create workload groups, classify sessions, and enforce caps 
 - **Azure SQL**: leverage elastic pools, database‑scoped resource classes, and service tiers to isolate and prioritize workload classes 

<!-- ------------------------- ------------------------- -->

#### Performance Insight and Metrics 
...provides visual dashboards for identifying trends, bottlenecks, and misconfigured resources 

- **Query Performance Insight (Azure SQL)**: breaks down top resource-consuming queries by CPU, duration, or execution count 
- **Elastic Pool Metrics**: monitor utilization across pooled databases to spot over-provisioning or unexpected spikes 
- **Server-Level Insights**: includes tempdb contention, long wait times, or high I/O via Azure metrics or Extended Events 
- **Historical Resource Metrics (`sys.resource_stats`)**: captures five‑minute snapshots of CPU, storage, and I/O usage; query this view to analyze usage over periods like “last week” for capacity planning and trend analysis 
- **Visualization Tools**: use Azure portal, Log Analytics, and Power BI for long-term trend analysis and executive dashboards 
- **When to Use**: capacity planning, performance reviews, and SLA compliance tracking 

<!-- ------------------------- ------------------------- -->
<!-- ------------------------- ------------------------- -->

### Architecture and Schema Design

#### Schema and Model Layout 
...influences how efficiently queries execute by shaping how data is organized and related 

- **Normalization**: reduces redundancy and ensures data integrity by organizing data into related tables; improves write efficiency 
- **Denormalization**: adds redundancy to reduce joins for read-heavy workloads; improves performance at the cost of storage and complexity 
- **Surrogate Keys**: use system-generated IDs (like `INT` or `GUID`) as primary keys instead of natural keys to simplify joins and indexing 
- **Join Efficiency**: design schema so related data can be joined on indexed columns with predictable cardinality 
- **Data Locality**: group frequently joined or co-accessed data to improve cache efficiency and reduce page reads 

<!-- ------------------------- ------------------------- -->

#### Elastic Pool Sizing 
...ensures that shared resources are properly allocated across multiple Azure SQL databases 

- **Peak Concurrency**: consider how many databases hit high CPU or I/O usage at the same time 
- **CPU Patterns**: analyze average and max CPU usage per database over time 
- **Storage Considerations**: total database size affects DTU or vCore selection, especially when tempdb and transaction logs are active 
- **Metrics to Monitor**: DTU usage, eDTUs per second, and storage percent; use Azure Monitor or Query Performance Insight 
- **Right-Sizing Strategy**: avoid over-provisioning by grouping databases with staggered workloads or low simultaneous usage 

<!-- ------------------------- ------------------------- -->

#### Compression and Storage Tiering 
...balances cost and performance by placing data in the right format and storage class 

- **Row/Page Compression**: apply to OLTP workloads to reduce I/O and storage, with minimal CPU overhead 
- **Columnstore Compression**: ideal for analytic tables; improves scan speed and reduces storage for large datasets 
- **Archival Compression**: used with columnstore indexes for cold or infrequently accessed data; saves space but increases CPU cost 
- **Azure Storage Tiers**: match table/index use with storage performance tiers (e.g., Premium SSD for high IOPS, Standard HDD for archival) 
- **Tradeoffs**: higher compression reduces cost and I/O but may increase CPU; monitor workload profile before applying across the board 

<!-- ------------------------- ------------------------- ------------------------- ------------------------- -->

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

8. Which recovery model and hint combination enables minimal logging for bulk loads 
 A) full model with TABLOCK 
 B) bulk_logged model with TABLOCK 
 C) simple model without TABLOCK 
 D) simple model with FORCESEEK 

9. To prevent readers blocking writers without rewriting queries which option should you enable 
 A) ALLOW_SNAPSHOT_ISOLATION 
 B) READ_COMMITTED_SNAPSHOT 
 C) READ_UNCOMMITTED 
 D) SNAPSHOT 

10. Which DMV shows cumulative wait times by wait type for diagnosing bottlenecks 
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

8. **B** – bulk_logged recovery model with TABLOCK enables minimal logging for large batch imports 
 *This combination reduces log usage while allowing bulk operations* 

9. **B** – READ_COMMITTED_SNAPSHOT uses row versioning to prevent reader/writer blocking without code changes 
 *It provides read consistency via tempdb versioning instead of locks* 

10. **B** – sys.dm_os_wait_stats shows cumulative wait times by wait type, key for diagnosing resource waits 
 *It identifies the most costly waits so you know which resources to tune*

<!-- ------------------------- ------------------------- ------------------------- ------------------------- -->

## On‑Prem

### Exercise

#### Prepare Resources

This exercise assumes a machine with:
* SQL Server (hybrid Win**DO**ws Authentication and SQL Authentication)
* SQL Server Management Studio

Open SQL Server Configuration Manager >> SQL Server Services and confirm that the SQL Server (MSSQLSERVER) service is running.

Launch SQL Server Management Studio, connect to localhost, and then click "New Query". 

Create a demonstration database:
```sql
CREATE DATABASE SecurityDemo;
```

Switch to the demonstration database:
```sql
USE SecurityDemo;
```
<!-- ------------------------- ------------------------- ------------------------- ------------------------- -->

#### Exercise #1: [Missing Index Recommendations via DMVs](https://learn.microsoft.com/sql/relational-databases/system-dynamic-management-views/sys-dm-db-missing-index-details) 
...uses built‑in dynamic management views to identify index candidates based on actual workload patterns 

##### How? 
- Queries `sys.dm_db_missing_index_details`, `sys.dm_db_missing_index_groups`, and `sys.dm_db_missing_index_group_stats` for columns your queries filter or join on 
- Reviews `equality_columns`, `inequality_columns`, and `included_columns` to script a new nonclustered index 
- Validates impact by comparing execution plans and I/O before and after index creation 

##### Why DMVs? 
- **Workload‑Driven**: suggestions reflect real query usage rather than guesswork 
- **Prioritization**: stats like `user_seeks` and `avg_total_user_cost` help you pick the highest‑impact index 
- **Safety**: you can script and test indexes in a sandbox before rolling out to production 

<!-- ------------------------- ------------------------- -->

##### Let's get started!

Enable I/O and time statistics: 
```sql
USE AdventureWorks2019;
SET STATISTICS IO, TIME ON;
```

1. Run the baseline query (likely a scan): 
 ```sql
 SELECT SalesOrderID, ProductID, UnitPrice, OrderQty
 FROM Sales.SalesOrderDetail
 WHERE UnitPrice > 1000;
 ``` 
 *Note the execution plan shows an Index Scan and STATISTICS IO reports logical reads* 

2. Retrieve missing‑index recommendations: 
 ```sql
 SELECT
 mid.equality_columns,
 mid.inequality_columns,
 mid.included_columns,
 migs.user_seeks,
 migs.avg_total_user_cost
 FROM sys.dm_db_missing_index_details AS mid
 JOIN sys.dm_db_missing_index_groups AS mig
 ON mid.index_handle = mig.index_handle
 JOIN sys.dm_db_missing_index_group_stats AS migs
 ON mig.index_group_handle = migs.group_handle
 WHERE mid.object_id = OBJECT_ID('Sales.SalesOrderDetail');
 ``` 
 *Look for `UnitPrice` in `equality_columns` and other key columns in `included_columns`* 

3. Create the recommended nonclustered index: 
 ```sql
 CREATE NONCLUSTERED INDEX IX_SOD_UnitPrice
 ON Sales.SalesOrderDetail (UnitPrice)
 INCLUDE (SalesOrderID, ProductID, OrderQty);
 ``` 

4. Re‑run the query and observe improvements: 
 ```sql
 SELECT SalesOrderID, ProductID, UnitPrice, OrderQty
 FROM Sales.SalesOrderDetail
 WHERE UnitPrice > 1000;
 ``` 
 *Execution plan now shows an Index Seek on `IX_SOD_UnitPrice` and STATISTICS IO reports far fewer logical reads* 

5. Clean up the test index: 
 ```sql
 DROP INDEX IX_SOD_UnitPrice
 ON Sales.SalesOrderDetail;
 ``` 

##### Final Thought 
By leveraging missing‑index DMVs, you can pinpoint exactly which columns and included fields will yield the largest performance gains. This hands‑on method ensures you add only the indexes your workload truly needs, avoiding unnecessary maintenance overhead and write penalties.

<!-- ------------------------- ------------------------- -->
<!-- ------------------------- ------------------------- -->

#### Exercise #2: [Index Maintenance and Fragmentation](https://learn.microsoft.com/sql/relational-databases/system-dynamic-management-views/sys-dm-db-index-physical-stats) 
...demonstrates how to measure and correct index fragmentation to keep seeks and scans efficient 

##### How? 
- Uses `sys.dm_db_index_physical_stats` to quantify fragmentation levels by index 
- References `sys.dm_db_index_usage_stats` to confirm which indexes see reads and writes 
- Applies `ALTER INDEX ... REORGANIZE` or `ALTER INDEX ... REBUILD` with an appropriate `FILLFACTOR` 
- Verifies fragmentation is reduced and discusses trade‑offs 

##### Why maintain fragmentation? 
- **I/O Efficiency**: fragmented pages force extra reads and slow scans/seeks 
- **Avoid Page Splits**: proper fill factor leaves free space for inserts, reducing rebuild frequency 
- **Sustained Performance**: regular maintenance prevents sudden performance degradation 

<!-- ------------------------- ------------------------- -->

##### Let's get started!

1. **Check current fragmentation** 
 ```sql
 USE AdventureWorks2019; 
 SELECT 
 OBJECT_NAME(ips.object_id) AS TableName, 
 i.name AS IndexName, 
 ips.index_type_desc, 
 ips.avg_fragmentation_in_percent 
 FROM sys.dm_db_index_physical_stats(
 DB_ID(), NULL, NULL, NULL, 'LIMITED'
 ) AS ips 
 JOIN sys.indexes AS i 
 ON ips.object_id = i.object_id 
 AND ips.index_id = i.index_id 
 WHERE ips.database_id = DB_ID()
 AND ips.avg_fragmentation_in_percent > 5;
 ``` 
 *Identify indexes with fragmentation above 5 percent*

2. **Review usage stats** 
 ```sql
 SELECT 
 OBJECT_NAME(ius.object_id) AS TableName, 
 i.name AS IndexName, 
 ius.user_seeks, 
 ius.user_scans, 
 ius.user_updates 
 FROM sys.dm_db_index_usage_stats AS ius 
 JOIN sys.indexes AS i 
 ON ius.object_id = i.object_id 
 AND ius.index_id = i.index_id 
 WHERE database_id = DB_ID()
 AND OBJECTPROPERTY(ius.object_id, 'IsUserTable') = 1;
 ``` 
 *Focus maintenance on indexes with both reads and writes*

3. **Reorganize lightly fragmented indexes (5–30 %)** 
 ```sql
 ALTER INDEX ALL 
 ON Sales.SalesOrderDetail 
 REORGANIZE;
 ``` 

4. **Rebuild heavily fragmented indexes (> 30 %) with FILLFACTOR** 
 ```sql
 ALTER INDEX ALL 
 ON Sales.SalesOrderDetail 
 REBUILD 
 WITH (FILLFACTOR = 90, ONLINE = ON);
 ``` 

5. **Verify fragmentation is reduced** 
 ```sql
 -- Rerun the initial DM_PHYSICAL_STATS query 
 ``` 
 *Confirm avg_fragmentation_in_percent falls into the acceptable range (< 5 %)* 

##### Final Thought 
Regular index maintenance—using the right operation and fill factor—keeps your indexes healthy, minimizes I/O, and avoids unexpected performance drops. By combining physical stats with usage data, you focus effort where it delivers the greatest benefit.

<!-- ------------------------- ------------------------- -->
<!-- ------------------------- ------------------------- -->

#### Exercise #3: [Plan Cache and Memory Grant Analysis](https://learn.microsoft.com/sql/relational-databases/system-dynamic-management-views/sys-dm-exec-query-memory-grants) 
...uses dynamic management views to surface memory‑grant issues and plan‑cache bloat, then tunes a database‑scoped setting to improve stability 

##### How? 
- Queries `sys.dm_exec_query_memory_grants` to inspect requested vs. granted memory and any waits 
- Uses `sys.dm_exec_cached_plans` joined with `sys.dm_exec_sql_text` to identify single‑use (“adhoc”) plans filling the cache 
- Applies the `OPTIMIZE_FOR_AD_HOC_WORKLOADS` setting at the database scope to convert full plans into lightweight stubs on first execution 

##### Let's get started!

1. **Check current memory grants** 
 ```sql
 USE AdventureWorks2019; 
 SELECT 
 session_id, 
 requested_memory_kb, 
 granted_memory_kb, 
 grant_time, 
 requested_memory_kb - granted_memory_kb AS shortfall_kb, 
 sql_text.text 
 FROM sys.dm_exec_query_memory_grants AS mg 
 CROSS APPLY sys.dm_exec_sql_text(mg.plan_handle) AS sql_text 
 WHERE mg.requested_memory_kb > 0;
 ``` 
 *Review any queries waiting for memory or receiving less than requested* 

2. **Identify ad‑hoc plan bloat** 
 ```sql
 SELECT 
 cp.cacheobjtype, 
 cp.objtype, 
 COUNT(*) AS plan_count, 
 SUM(cp.size_in_bytes)/1024 AS total_kb 
 FROM sys.dm_exec_cached_plans AS cp 
 WHERE cp.objtype = 'Adhoc' 
 GROUP BY cp.cacheobjtype, cp.objtype; 
 ``` 
 *High counts mean many single‑use plans are consuming memory* 

3. **Enable optimize_for_adhoc_workloads** 
 ```sql
 ALTER DATABASE SCOPED CONFIGURATION 
 SET OPTIMIZE_FOR_AD_HOC_WORKLOADS = ON; 
 ``` 

4. **Rerun a representative workload** 
 ```sql
 -- Execute some ad-hoc queries, e.g.:
 SELECT COUNT(*) FROM Person.Person WHERE LastName LIKE 'A%';
 SELECT TOP(5) * FROM Sales.SalesOrderHeader WHERE OrderDate > '2013-01-01';
 -- Repeat a few times with slight variations
 ``` 

5. **Re‑check plan cache bloat** 
 ```sql
 SELECT 
 cp.cacheobjtype, 
 cp.objtype, 
 COUNT(*) AS plan_count, 
 SUM(cp.size_in_bytes)/1024 AS total_kb 
 FROM sys.dm_exec_cached_plans AS cp 
 WHERE cp.objtype = 'Adhoc' 
 GROUP BY cp.cacheobjtype, cp.objtype; 
 ``` 
 *You should see fewer full‑plan entries and more “stub” entries, lowering memory use* 

6. **Verify improved memory‑grant behavior** 
 ```sql
 -- Rerun step 1 and confirm fewer or no memory‑wait rows 
 ``` 

##### Final Thought 
By combining memory‑grant and plan‑cache insights with the `OPTIMIZE_FOR_AD_HOC_WORKLOADS` setting, you can dramatically reduce memory bloat from ad‑hoc queries, leading to more consistent performance and reduced compile‑time pressure.

<!-- ------------------------- ------------------------- ------------------------- ------------------------- -->

### Quiz

1. Which dynamic management view provides the list of columns your query filters on for missing‑index suggestions? 
 A) `sys.dm_exec_query_stats` 
 B) `sys.dm_db_missing_index_details` 
 C) `sys.dm_db_index_physical_stats` 
 D) `sys.dm_db_index_usage_stats` 

2. In the missing‑index DMV output, which field shows columns that should be included to cover the query? 
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

5. Which DMV helps you measure the percentage of page fragmentation for each index? 
 A) `sys.dm_db_index_usage_stats` 
 B) `sys.dm_db_index_physical_stats` 
 C) `sys.dm_exec_query_memory_grants` 
 D) `sys.dm_os_wait_stats` 

6. According to best practice, you should rebuild an index when fragmentation exceeds what threshold? 
 A) 5 percent 
 B) 10 percent 
 C) 30 percent 
 D) 50 percent 

7. Which DMV reveals how often an index is used for seeks, scans, and updates? 
 A) `sys.dm_db_missing_index_group_stats` 
 B) `sys.dm_db_index_physical_stats` 
 C) `sys.dm_db_index_usage_stats` 
 D) `sys.dm_exec_cached_plans` 

8. To reorganize lightly fragmented indexes (5–30 percent), which command would you run? 
 A) `ALTER INDEX ALL ON ... REBUILD` 
 B) `ALTER INDEX ALL ON ... REORGANIZE` 
 C) `DBCC INDEXDEFRAG` 
 D) `DBCC SHRINKFILE` 

9. Which DMV shows queries that requested more memory than they were granted? 
 A) `sys.dm_exec_query_stats` 
 B) `sys.dm_exec_query_memory_grants` 
 C) `sys.dm_os_wait_stats` 
 D) `sys.dm_exec_cached_plans` 

10. Which database‑scoped configuration converts single‑use plans into lightweight stubs to reduce plan‑cache bloat? 
 A) `LEGACY_CARDINALITY_ESTIMATION` 
 B) `QUERY_OPTIMIZER_HOTFIXES` 
 C) `OPTIMIZE_FOR_AD_HOC_WORKLOADS` 
 D) `OPTIMIZE_FOR_SEQUENTIAL_KEY` 

<!-- ------------------------- ------------------------- -->

#### Answers

1. **B** – `sys.dm_db_missing_index_details` 
 *This DMV lists the columns your queries filter on for missing‑index recommendations* 

2. **C** – `included_columns` 
 *It specifies which non‑key columns to include so the index covers the query* 

3. **C** – `SET STATISTICS IO, TIME ON` 
 *This command enables detailed I/O and duration metrics for each statement* 

4. **B** – an Index Scan becomes an Index Seek 
 *Adding the index lets the optimizer seek directly to matching rows instead of scanning* 

5. **B** – `sys.dm_db_index_physical_stats` 
 *It reports fragmentation metrics like `avg_fragmentation_in_percent` for each index* 

6. **C** – 30 percent 
 *Indexes over 30 percent fragmented are best rebuilt to restore page order* 

7. **C** – `sys.dm_db_index_usage_stats` 
 *This DMV shows user_seeks, user_scans, and user_updates for each index* 

8. **B** – `ALTER INDEX ALL ON ... REORGANIZE` 
 *REORGANIZE defragments pages in place without rebuilding the entire index* 

9. **B** – `sys.dm_exec_query_memory_grants` 
 *It shows requested vs. granted memory and any wait times for memory grants* 

10. **C** – `OPTIMIZE_FOR_AD_HOC_WORKLOADS` 
 *This setting stores only a stub on first execution, converting to a full plan only upon reuse*

<!-- ------------------------- ------------------------- ------------------------- ------------------------- -->

## Azure

### Exercises

#### Exercise #1: [Automatic Tuning in Azure SQL](https://learn.microsoft.com/azure/azure-sql/database/automatic-tuning-overview) 
...leverages Azure's built‑in tuning engine to apply and monitor index and plan corrections without manual intervention 

##### How? 
- Enables automatic plan correction and automatic index create/drop on your database 
- Generates tuning recommendations based on Query Store telemetry 
- Applies, monitors, or rolls back changes via the Azure portal or T‑SQL 

##### Let's get started! 
1. **Enable automatic tuning** 
 ```sql
 ALTER DATABASE CURRENT 
 SET AUTOMATIC_TUNING = 
 ( FORCE_LAST_GOOD_PLAN = ON, 
 CREATE_INDEX = ON, 
 DROP_INDEX = ON );
 ``` 

2. **Review recommendations in the portal** 
 - Open your Azure SQL database in the Azure portal 
 - Under **Intelligent Performance**, select **Automatic tuning** 
 - Observe any **automatic tuning actions** (index create/drop or plan corrections) and their status 

3. **Trigger a recommendation** 
 - Run a sample query that would benefit from an index (e.g. a scan on SalesOrderDetail as in Exercise #1) 
 - Wait a few minutes for Query Store to collect telemetry and Automatic Tuning to propose an index 

4. **Inspect and override** 
 - In the portal, under **Performance recommendations**, locate the suggested index 
 - Click **Apply** to have Azure create it automatically, or **Disable** to reject 

5. **Validate impact** 
 - Re‑run your query and compare execution plans (SSMS) or use **Query Performance Insight** 
 - Optionally, roll back the automatic index via 
 ```sql
 EXEC sp_delete_database_automatic_tuning_recommendation 
 @resource_group_name = N'<rg>', 
 @server_name = N'<server>', 
 @database_name = N'<db>', 
 @recommendation_id = '<recommendation_guid>';
 ``` 

##### Final Thought 
Automatic Tuning in Azure SQL can offload routine index and plan corrections, letting you focus on higher‑value tasks. Always review recommendations to ensure they align with your workload patterns and maintenance win**DO**ws. 

<!-- ------------------------- ------------------------- -->

#### Exercise #2: [Query Performance Insight](https://learn.microsoft.com/azure/azure-sql/database/query-performance-insight-overview) 
...provides a built‑in dashboard showing top resource‑consuming queries over time and helps prioritize tuning efforts 

##### How? 
- Uses Query Store data to visualize CPU, duration, and execution count trends 
- Breaks down individual query performance and correlates plan changes 

##### Let's get started! 
1. **Open Query Performance Insight** 
 - In the Azure portal, navigate to your Azure SQL database 
 - Select **Query Performance Insight** under **Intelligent Performance** 

2. **Identify hot queries** 
 - Observe the top queries by CPU, duration, or count 
 - Click a query to view its **performance over time** and **plan comparison** 

3. **Drill into details** 
 - View the execution plan history side‑by‑side to spot regressions 
 - Note any recommendations (e.g., missing indexes) surfaced 

4. **Apply a tuning action** 
 - For a given query, click **Force Plan** to pin the last known good plan 
 - Alternatively, script a missing index from the recommendation pane 

5. **Validate improvement** 
 - Refresh the dashboard after rerunning the workload 
 - Confirm a drop in CPU or duration for the tuned query 

##### Final Thought 
Query Performance Insight empowers you to focus on your real‑world workload, surfacing the queries that matter most and giving you actionable insights without manual DMV queries. 

<!-- ------------------------- ------------------------- ------------------------- ------------------------- -->

### Quiz

1. Which T‑SQL statement correctly enables automatic plan correction, index creation, and index dropping on an Azure SQL database? 
 A) `ALTER DATABASE CURRENT SET QUERY_STORE = ON;` 
 B) `ALTER SERVER CONFIGURATION SET AUTO_TUNING = ALL;` 
 C) 
 `ALTER DATABASE CURRENT 
 SET AUTOMATIC_TUNING = 
 ( FORCE_LAST_GOOD_PLAN = ON, 
 CREATE_INDEX = ON, 
 DROP_INDEX = ON );` 
 D) `EXEC sp_auto_tuning_start;` 

2. What **DO**es the Automatic Tuning option CREATE_INDEX **DO**? 
 A) Forces the last known good plan when a regression occurs 
 B) Automatically creates recommended missing indexes 
 C) Drops indexes that have not been used recently 
 D) Updates stale statistics automatically 

3. In the Azure portal, where **DO** you navigate to view and configure Automatic Tuning? 
 A) Azure Monitor → Alerts 
 B) SQL Databases → Activity Log 
 C) Intelligent Performance → Automatic tuning 
 D) Advisor Recommendations → Performance 

4. Which button in the Automatic Tuning blade disables a specific tuning recommendation? 
 A) Apply 
 B) Force Plan 
 C) Disable 
 D) Refresh 

5. Which stored procedure **DO** you use to remove a specific automatic tuning recommendation? 
 A) `sp_remove_tuning_recommendation` 
 B) `sp_drop_plan_guide` 
 C) `sp_delete_database_automatic_tuning_recommendation` 
 D) `sp_delete_index_suggestion` 

6. Which Azure portal feature shows the top resource‑consuming queries over time? 
 A) Automatic Tuning 
 B) Query Performance Insight 
 C) SQL Analytics 
 D) Activity Log 

7. In Query Performance Insight, which view lets you compare execution plans before and after a change? 
 A) CPU Analysis 
 B) Plan Comparison 
 C) Index Recommendations 
 D) Diagnostic Logs 

8. To pin a stable plan for a query in Query Performance Insight, you click: 
 A) Apply 
 B) Force Plan 
 C) Pin Plan 
 D) Hold Plan 

9. After applying a tuning action (plan force or index create), what is the recommended way to validate its impact? 
 A) Run `DBCC CHECKDB` 
 B) Compare pre‑ and post‑tuning execution plans or observe CPU/duration drops in Query Performance Insight 
 C) Query `sys.dm_db_missing_index_details` again 
 D) Restart the database 

10. What underlying feature must be enabled for Query Performance Insight to display historical query data? 
 A) Automatic Tuning 
 B) Advanced Data Security 
 C) Query Store 
 D) Dynamic Data Masking 

<!-- ------------------------- ------------------------- -->

#### Answers

### Answers

1. **C** – `ALTER DATABASE CURRENT SET AUTOMATIC_TUNING = (FORCE_LAST_GOOD_PLAN = ON, CREATE_INDEX = ON, DROP_INDEX = ON);` 
 *Turns on all three Automatic Tuning options (plan correction, index create, and index drop) in one command.* 

2. **B** – `CREATE_INDEX` 
 *Automatically implements missing‑index recommendations based on Query Store telemetry.* 

3. **A** – `FORCE_LAST_GOOD_PLAN` 
 *Reverts to the last known good execution plan when a new plan degrades performance.* 

4. **C** – SQL Database → Intelligent Performance → Automatic tuning 
 *The portal path under your database settings where you view and configure Automatic Tuning.* 

5. **C** – `Disable` 
 *Rejects the specific tuning recommendation so it will not be applied.* 

6. **C** – `sp_delete_database_automatic_tuning_recommendation` 
 *Deletes the specified automatic tuning suggestion from the database.* 

7. **A** – **Query Performance Insight** 
 *Surfaces the top resource‑consuming queries over time using Query Store data.* 

8. **C** – **Execution count** 
 *Shows how many times each query has run, highlighting the most frequently executed.* 

9. **B** – **Force Plan** 
 *Pins the selected execution plan so subsequent runs use that known‑good plan.* 

10. **C** – **Query Store** 
 *Must be enabled to capture and store historical query and execution plan information.*