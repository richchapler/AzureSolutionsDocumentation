# SQL: Configuration Optimization

This curriculum provides hands‑on experience with optimizing SQL Server configuration settings in both On-Prem environments and Azure. It focuses on three key areas: Storage, Memory, and CPU. Each section includes practical exercises that compare and contrast On-Prem best practices with the performance tuning available in Azure

## Objectives

- Provide practical Exercises for optimizing storage, memory, and CPU configurations in SQL Server
- Compare On-Prem configuration techniques with Azure performance settings
- Reinforce best practices in instance‑level performance tuning using simple, demonstrable exercises

------------------------- ------------------------- ------------------------- -------------------------

## Memory

### On-Prem

#### Maximum Server Memory  
Controls the upper limit of memory that SQL Server can use. This setting is critical because it prevents SQL Server from consuming so much memory that the operating system and other applications become starved. Adjusting it helps ensure optimal performance and stability.  
Steps to find:  
- Open SSMS and connect to your SQL Server instance.  
- Right-click the server in Object Explorer and select Properties.  
- Go to the Memory page and locate "Maximum server memory."  

#### Minimum Server Memory  
Ensures SQL Server reserves a baseline amount of memory. This setting helps maintain a consistent performance foundation and prevents sudden drops in available memory for query processing.  
Steps to find:  
- In SSMS, open Server Properties for your SQL Server instance.  
- Navigate to the Memory page and find the "Minimum server memory" setting.  

#### Index Creation Memory  
Specifies the amount of memory allocated for index creation operations. This setting can affect the speed of index rebuilds and creations; if set too low, index operations may run slower due to insufficient memory.  
Steps to find:  
- Open SSMS and execute the following T‑SQL command:  
  ```sql
  EXEC sp_configure 'index creation memory';
  ```  
- Review the current value in the results.  

#### Minimum Memory per Query  
Determines the minimum amount of memory allocated to each query during execution. This ensures that complex queries have sufficient memory to run efficiently, thereby preventing performance degradation due to memory constraints.  
Steps to find:  
- Open SSMS and execute:  
  ```sql
  EXEC sp_configure 'minimum memory per query';
  ```  
- Review the returned configuration value.  

#### Buffer Pool Extension  
Allows the use of fast storage (usually SSDs) as an extension of the buffer pool. This can improve performance in memory-constrained environments by reducing disk I/O, although it is not a substitute for having adequate physical memory.  
Steps to find:  
- Open SSMS and execute:  
  ```sql
  EXEC sp_configure 'buffer pool extension enabled';
  ```  
- Check the configuration value; note that enabling or disabling this setting requires T‑SQL commands.  

#### In‑Memory OLTP Memory Settings  
Relate to memory‑optimized tables and indexes. Optimizing these settings can dramatically improve transaction processing and overall performance for workloads that utilize in‑memory OLTP.  
Steps to find:  
- Open the database properties in SSMS.  
- Navigate to the Files page and check for the presence of a Memory‑Optimized Filegroup.  
- Optionally, query `sys.database_files` to review file settings for memory‑optimized objects.  

#### Optimize for Ad Hoc Workloads  
Configures SQL Server to cache only plan stubs for single-use queries, thereby reducing memory usage in the plan cache. This setting can free up memory for more frequently executed queries, improving overall performance.  
Steps to find:  
- Open SSMS and run the following T‑SQL command:  
  ```sql
  EXEC sp_configure 'optimize for ad hoc workloads';
  ```  
- Review the current configuration; use sp_configure with RECONFIGURE to change the setting if necessary.  

#### Resource Governor (Memory Configuration)  
Allows you to allocate and limit memory usage for specific workloads. This ensures that high-priority tasks receive enough memory while preventing less critical workloads from consuming excessive resources.  
Steps to find:  
- Open SSMS and expand the Management folder in Object Explorer.  
- Right-click Resource Governor and select Properties to view the current resource pools.  
- Alternatively, run T‑SQL commands (e.g., CREATE RESOURCE POOL or ALTER RESOURCE POOL) to view and configure memory limits.  

#### Lock Pages in Memory  
Prevents SQL Server memory from being paged out to disk, ensuring that critical data remains in RAM. This OS-level setting is important for performance, especially on servers with heavy workloads, but it requires administrator rights and a restart of the SQL Server service to take effect.  
Steps to find:  
- Open SQL Server Configuration Manager.  
- Right-click the SQL Server service, select Properties, and then go to the Advanced tab.  
- Locate the "Lock pages in memory" setting and verify if it is enabled.

------------------------- -------------------------

#### Exercise

- Open SSMS and navigate to Server Properties → Memory  
- Set “Maximum server memory” to a lower value (for example, 512 MB) and run a memory‑intensive query (such as a large sort or hash join)  
- Record the query execution time  
- Increase “Maximum server memory” and run the query again  
- Compare the execution times to see the impact of memory allocation on performance  
- Optionally, check for memory pressure messages in the SQL Server error log or via dynamic management views

------------------------- -------------------------

#### Quiz

Question 1  
Which server property in SSMS controls the maximum amount of memory SQL Server can use  
- A. Minimum server memory  
- B. Maximum server memory  
- C. Index creation memory  
- D. Query governor cost limit

Question 2  
Reducing the maximum server memory on an On-Prem typically results in  
- A. Improved query performance  
- B. Reduced memory pressure  
- C. Increased memory pressure and slower performance  
- D. Enhanced parallelism

Question 3  
Which tool can be used to monitor memory pressure in SQL Server  
- A. Dynamic Management Views (DMVs)  
- B. SQL Server Agent  
- C. Maintenance Plans  
- D. Extended Events

-------------------------

##### Answer Key

Question 1  
Answer: B  
Explanation: Maximum server memory sets the upper limit for SQL Server's memory usage  
Correlation: Basic configuration setting as referenced in DP‑300 fundamentals

Question 2  
Answer: C  
Explanation: Lowering maximum server memory can cause memory pressure and slower performance  
Correlation: Reflects DP‑300 emphasis on proper memory configuration

Question 3  
Answer: A  
Explanation: DMVs provide insights into memory pressure and usage  
Correlation: Aligns with DP‑300 best practices for monitoring instance performance

------------------------- -------------------------

### Azure

#### Service Tier Selection  
Determines the memory allocation by automatically managing resources based on the selected performance tier. Higher tiers provide more memory, which can improve query performance.  
Steps to find:  
- Open the Azure portal and navigate to your Azure SQL Database resource  
- Select "Configure" under Compute + Storage  
- Review the current service tier and its vCore/DTU allocation, which indirectly reflects the memory available  

#### Monitoring Memory with Query Performance Insight  
Provides insights into query performance and memory usage. Monitoring these metrics helps determine if the current service tier is sufficient for your workload.  
Steps to find:  
- In the Azure portal, open your Azure SQL Database resource  
- Click on "Query Performance Insight"  
- Review the memory usage metrics and observe how memory allocation affects query duration  

#### Scaling Up for Increased Memory Allocation  
Scaling up the service tier increases CPU, memory, and I/O resources. This is an automated process that reallocates memory based on the new tier.  
Steps to find:  
- In the Azure portal, open your Azure SQL Database resource  
- Click on "Configure" under Compute + Storage  
- Increase the pricing tier to a higher level and save your changes  
- Monitor performance improvements using Query Performance Insight  

#### Automatic Memory Management  
Azure SQL Database automatically manages memory allocation based on the selected service tier and workload. No manual configuration is available, which simplifies administration.  
Steps to find:  
- In the Azure portal, navigate to the "Overview" or "Metrics" section of your Azure SQL Database  
- Review the current performance metrics to understand how memory is being utilized under the selected tier

------------------------- -------------------------

#### Exercise

- In the Azure portal, open your Basic‑tier Azure SQL Database  
- Execute a moderately large query and record its duration using Query Performance Insight  
- Scale up to a higher service tier and rerun the query  
- Compare the results to observe how the higher tier (with increased memory allocation) improves performance

------------------------- -------------------------

#### Quiz

Question 1  
How is memory allocation primarily managed in Azure  
- A. Manual configuration  
- B. Through the chosen service tier  
- C. Via SQL Server Agent  
- D. By Resource Governor

Question 2  
When you scale up an Azure SQL Database to a higher tier, what happens to the memory allocation  
- A. It remains unchanged  
- B. It decreases  
- C. It increases  
- D. It is manually adjusted

Question 3  
Which Azure portal tool helps monitor query performance and memory usage  
- A. SQL Server Management Studio  
- B. Query Performance Insight  
- C. Resource Monitor  
- D. Azure Data Studio

-------------------------

##### Answer Key

Question 1  
Answer: B  
Explanation: Memory in Azure SQL Database is automatically managed by the selected service tier  
Correlation: Reflects DP‑300 guidance on PaaS resource management

Question 2  
Answer: C  
Explanation: Scaling up to a higher service tier increases memory allocation  
Correlation: Matches DP‑300 concepts for scaling Azure resources

Question 3  
Answer: B  
Explanation: Query Performance Insight is the recommended tool for monitoring query performance and memory usage in Azure SQL Database  
Correlation: Aligns with DP‑300 best practices for monitoring in PaaS

------------------------- ------------------------- ------------------------- -------------------------

## CPU

### On-Prem

#### Processor Affinity and MAXDOP  
Controls how SQL Server utilizes CPU cores by specifying which processors are used for query execution and limiting the number of processors used in parallel processing. Proper configuration can improve query response times and prevent inefficient CPU usage.  
Steps to find:  
- Open SSMS and connect to your SQL Server instance  
- Right-click the server in Object Explorer, select Properties, and go to the "Processors" page  
- Review the "Automatically set processor affinity mask" and "Maximum degree of parallelism" (MAXDOP) settings

#### Resource Governor for CPU  
Allows you to define resource pools and workload groups to limit and allocate CPU usage among different sessions or workloads. This helps ensure that critical tasks receive sufficient CPU resources while preventing less critical tasks from over-consuming CPU.  
Steps to find:  
- Open SSMS and expand the "Management" folder in Object Explorer  
- Right-click "Resource Governor" and select "Properties" to view the current configuration  
- Alternatively, execute T‑SQL commands (CREATE RESOURCE POOL, ALTER RESOURCE POOL) to view or modify CPU limits

#### Monitoring CPU Usage  
Monitoring CPU performance is key to understanding workload demands and identifying bottlenecks. It helps you decide when to adjust CPU-related configurations or scale hardware resources.  
Steps to find:  
- In SSMS, execute dynamic management view queries such as:  
  ```sql
  SELECT * FROM sys.dm_os_performance_counters WHERE counter_name LIKE '%CPU%';
  ```  
- Alternatively, use Performance Monitor (PerfMon) on the server to track CPU metrics during query execution

#### Cost Threshold for Parallelism  
Determines the threshold at which SQL Server considers a query for parallel execution. Lowering this value can force more queries to run in parallel, while a higher value limits parallelism to only expensive queries.  
Steps to find:  
- Open SSMS and connect to your SQL Server instance  
- Right-click the server, select Properties, and go to the "Advanced" page  
- Locate the "Cost Threshold for Parallelism" setting to review or adjust its value

------------------------- -------------------------

#### Exercise

- Run a parallelizable query on your On-Prem database and record its execution time  
- Execute the following T-SQL to set `max degree of parallelism` (MAXDOP) to 1:
  ```sql
  sp_configure 'max degree of parallelism', 1;
  RECONFIGURE;
  ```
- Rerun the query and compare the performance  
- Additionally, create a Resource Governor pool with a classifier function to throttle CPU usage for a specific login, then run a workload under that login and observe CPU usage differences

------------------------- -------------------------

#### Quiz

Question 1  
Which configuration setting controls the maximum degree of parallelism in SQL Server  
- A. MAXDOP  
- B. Resource Governor  
- C. Processor affinity  
- D. Cost Threshold for Parallelism

Question 2  
Reducing MAXDOP to 1 in an On-Prem environment typically results in  
- A. Improved performance for parallel queries  
- B. Reduced parallelism and potentially slower execution  
- C. Increased memory usage  
- D. Enhanced query optimization

Question 3  
What is the purpose of using Resource Governor in SQL Server  
- A. To schedule backups  
- B. To limit CPU usage for specific workloads  
- C. To adjust memory settings  
- D. To configure file placements

-------------------------

##### Answer Key

Question 1  
Answer: A  
Explanation: MAXDOP (maximum degree of parallelism) limits the number of processors used for parallel query execution  
Correlation: Based on DP‑300 concepts for CPU configuration

Question 2  
Answer: B  
Explanation: Setting MAXDOP to 1 limits parallel processing and can slow down queries that benefit from parallelism  
Correlation: Reflects DP‑300 emphasis on proper CPU configuration

Question 3  
Answer: B  
Explanation: Resource Governor is used to manage and limit CPU usage for specific workloads  
Correlation: Matches DP‑300 ideas on managing CPU resources on-premises

------------------------- -------------------------

### Azure

#### Service Tier Selection for CPU  
In Azure SQL Database (PaaS), the selected service tier (DTU/vCore model) directly determines the available CPU resources. Upgrading to a higher tier increases the number of vCores or DTUs, thereby enhancing CPU performance.  
Steps to find:  
- Open the Azure portal and navigate to your Azure SQL Database resource  
- Click on "Configure" under Compute + Storage  
- Review the current service tier and its associated CPU (vCore/DTU) allocation  

#### Monitoring CPU Usage in Azure  
Monitoring CPU performance in Azure SQL Database helps identify workload bottlenecks and validates the adequacy of the chosen service tier.  
Steps to find:  
- In the Azure portal, navigate to the "Metrics" or "Query Performance Insight" section of your Azure SQL Database  
- Select CPU-related metrics (such as CPU percentage) to observe usage trends during query execution  

#### Scaling Up for Improved CPU Performance  
Scaling up the service tier in Azure SQL Database automatically increases CPU resources, which can reduce query execution time for CPU-intensive operations.  
Steps to find:  
- In the Azure portal, open your Azure SQL Database resource and click on "Configure" under Compute + Storage  
- Choose a higher service tier and save your changes  
- Monitor performance improvements via Query Performance Insight or the Metrics section  

#### Query Optimization for CPU Efficiency  
While direct CPU settings cannot be manually configured in Azure SQL Database (PaaS), refining query design can reduce CPU usage and improve performance.  
Steps to find:  
- In the Azure portal or SSMS, run a CPU-intensive query and record its duration using Query Performance Insight  
- Review the query execution plan to identify inefficiencies  
- Optimize the query (for example, by adding an appropriate index or rewriting complex joins) and re-run the query to compare performance improvements

------------------------- -------------------------

#### Exercise

- Create a Basic‑tier Azure SQL Database and run a CPU‑intensive query (for example, one with multiple joins and aggregates)  
- Record the query duration using Query Performance Insight  
- Scale the database to a higher tier (e.g., S2 or S3) and run the query again  
- Compare the performance to see how increased CPU resources (vCores) reduce query time  
- Optionally, refine the query design (for example, by creating an index) and observe the combined effect with a higher CPU tier

------------------------- -------------------------

#### Quiz

Question 1  
In Azure, CPU resources are allocated based on which factor  
- A. Manual configuration by the DBA  
- B. The selected service tier (vCores or DTUs)  
- C. Resource Governor settings  
- D. SQL Server Agent scheduling

Question 2  
Scaling up the service tier in Azure SQL Database generally results in  
- A. Decreased CPU resources  
- B. Increased CPU resources  
- C. No change in CPU allocation  
- D. Manual adjustment of CPU settings

Question 3  
Which recommended practice helps optimize CPU performance in Azure  
- A. Configuring processor affinity  
- B. Refining query design along with selecting the appropriate service tier  
- C. Using Resource Governor  
- D. Increasing the number of data files

-------------------------

##### Answer Key

Question 1  
Answer: B  
Explanation: In Azure SQL Database, CPU resources are allocated based on the chosen service tier, which determines the number of vCores or DTUs  
Correlation: Reflects DP‑300 guidance on PaaS CPU resource allocation

Question 2  
Answer: B  
Explanation: Scaling up the service tier increases the available CPU resources  
Correlation: Matches DP‑300 concepts for scaling in Azure

Question 3  
Answer: B  
Explanation: Optimizing query design in tandem with selecting the appropriate service tier is the recommended approach to improve CPU performance in Azure SQL Database  
Correlation: Aligns with DP‑300 best practices for PaaS optimization

------------------------- ------------------------- ------------------------- -------------------------

## Storage

### On-Prem

#### File Groups  
File groups help organize database files into logical units so that data can be spread across multiple physical disks. This can balance the I/O load and improve overall performance.  
Steps to find:  
- Open SSMS and connect to your SQL Server instance  
- Expand Databases, right-click a specific database, and select Properties  
- Click on the Files page to view the list of file groups and their associated files  

#### Storage Pools and Striping  
Storage pools and striping combine multiple disks into a single logical volume, allowing data to be distributed evenly across disks. This enhances I/O throughput by reducing bottlenecks on any single disk.  
Steps to find:  
- In a demo environment, simulate storage pools by creating multiple virtual disks (using different drive letters) in your operating system  
- When creating a new database in SSMS, assign different data files to these separate disks  
- Verify file distribution by reviewing the file paths in the New Database dialog or under the Files page in database properties  

#### Manual File Placement  
Manual file placement involves intentionally placing data files and log files on separate physical disks to minimize I/O contention. This strategy ensures that the high-write operations of the log file do not interfere with data file I/O.  
Steps to find:  
- In SSMS, during the creation of a new database, click the Options or Files tab  
- Specify different file paths for data files (e.g., on one disk) and the log file (on a separate disk)  
- Review and confirm the file paths in the database properties after creation

------------------------- -------------------------

#### Exercise

- Create two virtual disks (for example, D: and E:) on your local machine or VM  
- In SSMS, create a database named `TestDB_SingleFile` with a single data file and log file on a single disk (e.g., C:)  
  - Run a loop to insert 500,000 rows into a test table and record the execution time  
- Create a second database named `TestDB_MultiFile` with two data files  
  - Place one data file on D: and the other on E:, with the log file on one of the disks  
  - Run the same insert loop and record the execution time  
- Compare the performance to see how distributing data files across disks can improve I/O throughput

------------------------- -------------------------

#### Quiz

Question 1  
Which of the following storage techniques is commonly used in an On-Prem environment to enhance I/O performance by distributing data across multiple disks  
- A. RAID and striping  
- B. SQL Server Agent  
- C. Data encryption  
- D. Backup compression

Question 2  
When configuring file placement for an On-Prem, what is the best practice to reduce I/O contention  
- A. Place both data and log files on the same physical disk  
- B. Place data files and log files on separate high‑performance disks  
- C. Consolidate data files into a single filegroup  
- D. Use only SSDs for log files

Question 3  
Which method helps distribute the database I/O load across multiple disks in an On-Prem environment  
- A. Storage pools and striping  
- B. Resource Governor  
- C. Query tuning  
- D. Dynamic data masking

-------------------------

##### Answer Key

Question 1  
Answer: A  
Explanation: RAID and striping combine multiple disks into a single logical unit to improve I/O throughput which is a common On-Prem optimization technique  
Correlation: Loosely based on DP‑300 Q3.6

Question 2  
Answer: B  
Explanation: Placing data files and log files on separate high‑performance disks minimizes I/O contention and enhances performance  
Correlation: Reflects best practices for file placement in On-Prem environments

Question 3  
Answer: A  
Explanation: Using storage pools and striping distributes the I/O load across multiple disks, improving overall performance  
Correlation: Matches the storage optimization concepts from DP‑300 Q3.6

------------------------- -------------------------

### Azure

#### Performance Tiers  
Determines the overall performance of an Azure SQL Database, including storage throughput. The chosen tier (DTU or vCore model) automatically allocates resources based on workload demands  
Steps to find:  
- Open the Azure portal and navigate to your Azure SQL Database resource  
- Click on "Configure" under the Compute + Storage section  
- Review the current service tier, which reflects the performance and storage allocation  

#### Built‑in High Availability and Automatic Backups  
Ensures that storage performance and data integrity are maintained without manual intervention. Azure SQL Database automatically manages backup schedules and high availability, reducing administrative overhead  
Steps to find:  
- In the Azure portal, open your Azure SQL Database resource  
- Click on the "Overview" or "Backup" section to review backup history and settings  
- Note how the service provides high availability by default through geo-replication and built‑in failover capabilities  

#### Automatic Storage Management  
Azure SQL Database leverages the service tier to automatically manage storage configuration, including file distribution and I/O optimization. This means no manual file placement is required, simplifying administration  
Steps to find:  
- In the Azure portal, navigate to the "Metrics" section for your Azure SQL Database  
- Monitor performance metrics such as DTU consumption and storage I/O  
- Observe how changes in workload affect these metrics, reflecting the underlying automated storage management

------------------------- -------------------------

#### Exercise

- In the Azure portal, create a Basic‑tier Azure SQL Database  
- Run a load test by inserting a large volume of data into a test table and record response times or DTU usage  
- Scale up the database to a Standard or Premium tier  
- Rerun the load test and compare performance metrics to observe how the service tier automatically manages storage performance

------------------------- -------------------------

#### Quiz

Question 1  
In an Azure SQL Database environment, which feature automatically manages storage performance  
- A. Performance tiers (DTU/vCore)  
- B. Resource Governor  
- C. Manual filegroup configuration  
- D. SQL Server Agent

Question 2  
Which built‑in feature of Azure SQL Database helps ensure storage efficiency through high availability and automatic backups  
- A. Manual RAID configuration  
- B. Automatic high availability and backups  
- C. Custom disk partitioning  
- D. Resource Governor

Question 3  
When scaling an Azure SQL Database to a higher tier, which action is most likely to improve storage performance  
- A. Configuring additional filegroups  
- B. Increasing the service tier  
- C. Enabling Resource Governor  
- D. Manually redistributing data files

-------------------------

##### Answer Key

Question 1  
Answer: A  
Explanation: Performance tiers (DTU/vCore) automatically manage storage performance in Azure SQL Database  
Correlation: Relates to DP‑300 Q3.6 concepts for automatic storage management in PaaS environments

Question 2  
Answer: B  
Explanation: Azure SQL Database provides built‑in high availability and automatic backups, ensuring efficient storage  
Correlation: Reflects the automated storage efficiency concepts in DP‑300

Question 3  
Answer: B  
Explanation: Scaling up the service tier increases available resources, thereby improving storage performance  
Correlation: Matches DP‑300 ideas on scaling for better performance

------------------------- ------------------------- ------------------------- -------------------------

