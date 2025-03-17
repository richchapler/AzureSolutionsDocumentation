# SQL: Configuration Optimization

This curriculum provides hands‑on experience with optimizing SQL Server configuration settings in both on‑premises environments and Azure SQL Database (PaaS). It focuses on three key areas: Storage, Memory, and CPU. Each section includes practical exercises that compare and contrast on‑premises best practices with the performance tuning available in Azure

## Objectives

- Provide practical hands‑on exercises for optimizing storage, memory, and CPU configurations in SQL Server
- Compare on‑premises configuration techniques with Azure SQL Database (PaaS) performance settings
- Reinforce best practices in instance‑level performance tuning using simple, demonstrable exercises

------------------------- -------------------------

## Storage

### On‑Premises SQL Server: Hands‑On Exercise

- Create two virtual disks (for example, D: and E:) on your local machine or VM  
- In SSMS, create a database named `TestDB_SingleFile` with a single data file and log file on a single disk (e.g., C:)  
  - Run a loop to insert 500,000 rows into a test table and record the execution time  
- Create a second database named `TestDB_MultiFile` with two data files  
  - Place one data file on D: and the other on E:, with the log file on one of the disks  
  - Run the same insert loop and record the execution time  
- Compare the performance to see how distributing data files across disks can improve I/O throughput

Below is the quiz for just the Storage – On‑Premises section, along with the answer key  
Each question is designed to loosely correlate with DP‑300 Q3.6 and related best practices for on‑prem storage configuration

------------------------- -------------------------

### Quiz: Storage – On‑Premises

Question 1  
Which of the following storage techniques is commonly used in an on‑premises SQL Server environment to enhance I/O performance by distributing data across multiple disks  
- A. RAID and striping  
- B. SQL Server Agent  
- C. Data encryption  
- D. Backup compression

Question 2  
When configuring file placement for an on‑premises SQL Server, what is the best practice to reduce I/O contention  
- A. Place both data and log files on the same physical disk  
- B. Place data files and log files on separate high‑performance disks  
- C. Consolidate data files into a single filegroup  
- D. Use only SSDs for log files

Question 3  
Which method helps distribute the database I/O load across multiple disks in an on‑premises environment  
- A. Storage pools and striping  
- B. Resource Governor  
- C. Query tuning  
- D. Dynamic data masking

------------------------- -------------------------

### Answer Key: Storage – On‑Premises

Question 1  
Answer: A  
Explanation: RAID and striping combine multiple disks into a single logical unit to improve I/O throughput which is a common on‑premises optimization technique  
Correlation: Loosely based on DP‑300 Q3.6

Question 2  
Answer: B  
Explanation: Placing data files and log files on separate high‑performance disks minimizes I/O contention and enhances performance  
Correlation: Reflects best practices for file placement in on‑premises environments

Question 3  
Answer: A  
Explanation: Using storage pools and striping distributes the I/O load across multiple disks, improving overall performance  
Correlation: Matches the storage optimization concepts from DP‑300 Q3.6

------------------------- -------------------------

### Azure SQL Database (PaaS): Hands‑On Exercise

- In the Azure portal, create a Basic‑tier Azure SQL Database  
- Run a load test by inserting a large volume of data into a test table and record response times or DTU usage  
- Scale up the database to a Standard or Premium tier  
- Rerun the load test and compare performance metrics to observe how the service tier automatically manages storage performance

------------------------- -------------------------

### Quiz: Storage – Azure SQL Database (PaaS)

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

------------------------- -------------------------

### Answer Key: Storage – Azure SQL Database (PaaS)

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

------------------------- -------------------------

## Memory

### On‑Premises SQL Server: Hands‑On Exercise

- Open SSMS and navigate to Server Properties → Memory  
- Set “Maximum server memory” to a lower value (for example, 512 MB) and run a memory‑intensive query (such as a large sort or hash join)  
- Record the query execution time  
- Increase “Maximum server memory” and run the query again  
- Compare the execution times to see the impact of memory allocation on performance  
- Optionally, check for memory pressure messages in the SQL Server error log or via dynamic management views

------------------------- -------------------------

### Quiz: Memory – On‑Premises SQL Server

Question 1  
Which server property in SSMS controls the maximum amount of memory SQL Server can use  
- A. Minimum server memory  
- B. Maximum server memory  
- C. Index creation memory  
- D. Query governor cost limit

Question 2  
Reducing the maximum server memory on an on‑premises SQL Server typically results in  
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

------------------------- -------------------------

### Answer Key: Memory – On‑Premises SQL Server

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

### Azure SQL Database (PaaS): Hands‑On Exercise

- In the Azure portal, open your Basic‑tier Azure SQL Database  
- Execute a moderately large query and record its duration using Query Performance Insight  
- Scale up to a higher service tier and rerun the query  
- Compare the results to observe how the higher tier (with increased memory allocation) improves performance

------------------------- -------------------------

### Quiz: Memory – Azure SQL Database (PaaS)

Question 1  
How is memory allocation primarily managed in Azure SQL Database (PaaS)  
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

------------------------- -------------------------

### Answer Key: Memory – Azure SQL Database (PaaS)

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

------------------------- -------------------------

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

------------------------- -------------------------

### Quiz: CPU – On‑Premises SQL Server

Question 1  
Which configuration setting controls the maximum degree of parallelism in SQL Server  
- A. MAXDOP  
- B. Resource Governor  
- C. Processor affinity  
- D. Cost Threshold for Parallelism

Question 2  
Reducing MAXDOP to 1 in an on‑premises SQL Server environment typically results in  
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

------------------------- -------------------------

### Answer Key: CPU – On‑Premises SQL Server

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

### Azure SQL Database (PaaS): Hands‑On Exercise

- Create a Basic‑tier Azure SQL Database and run a CPU‑intensive query (for example, one with multiple joins and aggregates)  
- Record the query duration using Query Performance Insight  
- Scale the database to a higher tier (e.g., S2 or S3) and run the query again  
- Compare the performance to see how increased CPU resources (vCores) reduce query time  
- Optionally, refine the query design (for example, by creating an index) and observe the combined effect with a higher CPU tier

------------------------- -------------------------

### Quiz: CPU – Azure SQL Database (PaaS)

Question 1  
In Azure SQL Database (PaaS), CPU resources are allocated based on which factor  
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
Which recommended practice helps optimize CPU performance in Azure SQL Database (PaaS)  
- A. Configuring processor affinity  
- B. Refining query design along with selecting the appropriate service tier  
- C. Using Resource Governor  
- D. Increasing the number of data files

------------------------- -------------------------

### Answer Key: CPU – Azure SQL Database (PaaS)

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
