Below is a set of simple, hands‑on exercise ideas for each bullet in the “Configuration Optimization” matrix. These suggestions assume you have only basic environments to work with (for example, a local SQL Server instance or a single Azure SQL Database). Each exercise is short, realistic, and highlights the principle behind the bullet point.

--------------------------------------------------------------------------------
STORAGE

On‑Premises SQL Server
• Configure file groups, RAID, storage pools, and striping to optimize I/O
  - Exercise: Create two virtual disks (for example, D: and E:) in your local VM or machine. In SSMS, create a database with at least two data files—one on each disk. Insert a large volume of rows (for example, a loop of 500,000 inserts). Record the time it takes. Compare to a single‑disk configuration to see if distributing files improves throughput.
• Manually plan data and log file placement on high‑performance disks
  - Exercise: Create a small database where the data file is on a “fast” virtual disk (D:) and the log file is on another disk (E:). Run an insert loop, then check performance stats or use SET STATISTICS IO to see read/write activity. Compare the result to having both data and log on the same disk.

Azure SQL Database (PaaS)
• Leverage performance tiers (DTU/vCore) to automatically manage storage performance
  - Exercise: Create a Basic‑tier Azure SQL Database. Run a simple load test by inserting large amounts of data. Note the response time or average DTU usage in the portal. Then scale up to Standard or Premium and repeat the test. Observe how storage performance automatically improves without manual disk configuration.
• Rely on built‑in high availability and automatic backups for storage efficiency
  - Exercise: In the Azure portal, open your Azure SQL Database. Locate the Backup/Restore or Point‑in‑Time Restore settings. (Optionally, drop a small table to simulate data loss.) Perform a restore to a point in time just before the table was dropped. This demonstrates how backups are automatic and readily available.

--------------------------------------------------------------------------------
MEMORY

On‑Premises SQL Server
• Adjust maximum and minimum server memory settings via SSMS or scripts
  - Exercise: In SSMS, go to Server Properties > Memory. Set “Maximum server memory” to a low value (for example, 512 MB if your machine has 2 GB or more). Run a memory‑intensive query (like a large sort or hash join). Observe performance. Then increase “Maximum server memory” and rerun the query. Compare execution times.
• Optimize buffer pool allocation and manage memory pressure manually
  - Exercise: Again, artificially lower “Maximum server memory,” then run a loop of insert statements or big selects. Watch the SQL Server error log or sys.dm_os_ring_buffers for “memory pressure” messages. Increase memory and see how quickly the pressure subsides. This demonstrates manual control over buffer pool allocation.

Azure SQL Database (PaaS)
• Memory allocation is managed by the chosen service tier
  - Exercise: Create a Basic‑tier Azure SQL Database. Run a moderately large query (for example, multiple joins on a sample table). Track query duration in the Query Performance Insight or by timing it in SSMS. Scale the database to a higher tier (Standard or Premium) and rerun the same query. Notice how the database effectively “has more memory” at higher tiers without you configuring anything.
• Monitor performance and select the appropriate tier in the Azure portal
  - Exercise: In the Azure portal, open the Metrics blade for your database. Observe DTU usage or vCore utilization. If usage spikes above 80% during tests, scale up one tier and monitor again. Discuss how this is the primary method for managing memory in Azure SQL Database.

--------------------------------------------------------------------------------
CPU

On‑Premises SQL Server
• Configure processor affinity and set maximum degree of parallelism (MAXDOP)
  - Exercise: Run a parallelizable query (for example, a large aggregate on a table). Record the time. Then use T-SQL:  
    sp_configure 'max degree of parallelism', 1;  
    RECONFIGURE;  
    Run the query again and compare times. This shows how limiting parallelism affects performance.  
• Use Resource Governor to fine‑tune CPU usage for specific workloads
  - Exercise: Create a Resource Governor pool with a classifier function that routes a specific login to that pool with a CPU cap. Run a workload under that login, observe it is throttled. Then run the same workload under a different login without a cap. Compare CPU usage and query times.

Azure SQL Database (PaaS)
• CPU resources are allocated based on the selected performance level (vCores or DTUs)
  - Exercise: Create a Basic (5 DTU) Azure SQL Database. Run a CPU‑intensive query (e.g., multiple joins, large aggregates). Observe the query duration or CPU usage in Query Performance Insight. Scale to a higher tier (S2 or S3) and rerun the same query. Note the faster completion time and higher CPU availability.
• Optimize query performance by choosing the appropriate service tier and refining query design
  - Exercise: Start in a lower tier (Basic or S0). Run a query that involves multiple table joins. Time it. Next, create an index or refine the query to reduce logical reads. Rerun it. Finally, scale up to a higher tier and run the refined query again. Compare each step’s performance to see the combined impact of both query tuning and a higher CPU tier.

--------------------------------------------------------------------------------

These hands‑on examples are kept simple so you can demonstrate the core principle with minimal setup. Even on a single machine or a free Azure trial, you can observe differences in performance and behavior when changing each setting or configuration.
