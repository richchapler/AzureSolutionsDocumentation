# SQL: Performance Optimization

## Use Case

The database team at a mid‑sized company is under pressure... **data volumes are growing fast**, **users are complaining about slow queries**, and **critical reports are missing their deadlines**. Leadership wants answers and assigns a mission: uncover what’s slowing things down, tune performance across the system, and make sure it doesn’t happen again.

The database team talks about the mission and settles on the following goals:

- Diagnose and fix slow queries before users notice
- Speed up data ingestion so pipelines finish on time
- Keep performance stable even as load increases
- Eliminate guesswork through clear, data-driven tuning
- Build skills to proactively optimize—not just react to problems

<!-- ------------------------- ------------------------- ------------------------- ------------------------- -->

## Fundamentals
In this course, we will first discuss the following fundamentals:

<!-- ------------------------- ------------------------- -->

### Query and Plan Optimization

- **Execution plan analysis**: interpreting actual and estimated plans to identify expensive operations, missing indexes, or suboptimal join strategies  
- **Query rewriting techniques**: refactoring scalar functions, nested loops, and cursors into efficient set-based logic; improving predicate shape  
- **Parameter sensitivity and sniffing**: detecting unstable plans and applying OPTIMIZE FOR UNKNOWN, recompile hints, or plan guides  
- **Plan caching and reuse**: minimizing cache bloat with forced parameterization or `OPTIMIZE_FOR_AD_HOC_WORKLOADS`  
- **Query store usage**: tracking regressed plans, comparing historical plans, and forcing stable execution plans  
- **Automatic tuning**: enabling and managing Azure features like automatic plan correction and automatic index recommendations  
- **Query live statistics**: using SSMS live query stats to troubleshoot blocking, long-running queries, and runtime bottlenecks  

<!-- ------------------------- ------------------------- -->

### Indexing and Storage Design

- **Index strategies**: designing clustered, nonclustered, filtered, and covering indexes based on workload patterns and execution plans  
- **Statistics management**: keeping column stats current to avoid bad cardinality estimates; understanding auto update behavior  
- **Columnstore and archival compression**: using columnstore indexes for large fact tables and archival compression for infrequent access  
- **Row and page compression**: reducing I/O for rowstore tables while balancing CPU tradeoffs  
- **Data type tuning**: using precise and efficient data types (e.g., `money`, `date`, `int`) to reduce storage and improve cache efficiency  

<!-- ------------------------- ------------------------- -->

### Parallelism, TempDB, and Resource Management

- **Parallelism tuning**: configuring MAXDOP and cost thresholds to optimize CPU usage without introducing excessive context switching  
- **TempDB optimization**: sizing and distributing TempDB data files to reduce contention in high-concurrency environments  
- **Memory and plan cache behavior**: identifying memory pressure caused by single-use plans, spills, or over-provisioned operators  
- **Resource governance**: isolating workloads using Resource Governor (on-prem) or selecting appropriate tiers for predictable performance  

<!-- ------------------------- ------------------------- -->

### Ingestion and Batch Optimization

- **Bulk load tuning**: enabling minimal logging, batching inserts, and optimizing transaction log throughput  
- **Data ingestion pipeline tuning**: improving throughput using Azure Data Factory, COPY INTO, and PolyBase with parallelization and compression  
- **In-memory optimization**: leveraging memory-optimized tables and natively compiled procedures for ultra-low-latency OLTP workloads  

<!-- ------------------------- ------------------------- -->

### Concurrency and Isolation Controls

- **Blocking and deadlock mitigation**: using appropriate isolation levels, query shaping, and indexing to reduce contention  
- **Read-consistency tuning**: enabling `READ_COMMITTED_SNAPSHOT` to reduce reader/writer blocking without rewriting queries  

<!-- ------------------------- ------------------------- -->

### Observability and Diagnostics

- **Monitoring and baselining**: establishing performance baselines using Query Store, Azure Monitor, and dynamic management views  
- **Alert tuning**: configuring alert rules with dynamic thresholds and low sensitivity to reduce noise while tracking real anomalies  
- **Execution history and telemetry**: using `Lightweight_Query_Profiling` and `Last_Query_Plan_Stats` to examine recent query behavior  
- **Performance Insight and metrics**: using built-in Azure dashboards to identify bottlenecks at the server, database, or elastic pool level  

<!-- ------------------------- ------------------------- -->

### Architecture and Schema Design

- **Schema and model layout**: evaluating normalization, denormalization, and surrogate key design for join efficiency and data locality  
- **Elastic pool sizing**: estimating DTU/vCore usage across databases based on peak concurrency, average CPU, and total database size  
- **Compression and storage tiering**: choosing archival vs. standard compression methods for hybrid storage-performance balance  

<!-- ------------------------- ------------------------- ------------------------- ------------------------- -->

### Quiz

1. Question  
    A) Answer #1  
    B) Answer #2  
    C) Answer #3  
    D) Answer #4 


<!-- ------------------------- ------------------------- -->

#### Answers

1. **B** – Answer #2 content  
   *Explanation why Answer #2 is correct*

<!-- ------------------------- ------------------------- ------------------------- ------------------------- -->

## On‑Prem

### Exercise

#### Prepare Resources

This exercise assumes a machine with:
* SQL Server (hybrid Windows Authentication and SQL Authentication)
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

#### Exercise #1: [Row-Level Security](https://learn.microsoft.com/en-us/sql/relational-databases/security/row-level-security?view=sql-server-ver16)
...enforces fine‑grained access control by limiting which rows in a table a given user can see or modify, restricting data at the row level based on the caller’s identity or context rather than granting or denying access to the entire table.

##### How?  
- Uses an **inline table‑valued function** that returns a result set of allowed rows
- Binds that function to a security policy  
- Then, SQL Server automatically:
  - Applies the predicate logic to SELECT, UPDATE, and DELETE operations (**filter predicates**) 
  - Enforces it on INSERT, UPDATE, and DELETE operations (**block predicates**)

##### Why functions?  
- **Modularity**: encapsulate filter logic in one place
- **Performance**: can be inlined by the query optimizer, minimizing overhead
- **Flexibility**: create multiple predicates without altering the security policy definition

<!-- ------------------------- ------------------------- -->

##### Let's get started!

Create sample table:
```sql
CREATE TABLE dbo.EmployeeData ( Id INT IDENTITY PRIMARY KEY, Name NVARCHAR(100), OwnerLogin SYSNAME );
```



##### Final Thought

Masking does not protect data from privileged users—it’s a display‑level control. To ensure masking is effective:  
- Avoid granting the **UNMASK** permission to application users  
- Combine masking with classification, auditing, and role‑based access control  
- Use it to protect casual exposure in shared environments, dashboards, and support tools

------------------------- -------------------------

### Quiz

<!-- ------------------------- ------------------------- -->

#### Answers

<!-- ------------------------- ------------------------- ------------------------- ------------------------- -->

## Azure

### Exercise

**Objective**: Build a hardened Azure SQL environment showcasing network isolation, Entra ID integration, threat protection, and centralized auditing using Azure PowerShell  

<!-- ------------------------- ------------------------- -->

#### Let’s get started!

Open Azure Portal >> Cloud Shell and switch to PowerShell.


<!-- ------------------------- ------------------------- -->

### Quiz

<!-- ------------------------- ------------------------- -->

### Answers
