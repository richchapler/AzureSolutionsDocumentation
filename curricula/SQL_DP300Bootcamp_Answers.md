# SQL: DP-300 Bootcamp, Answer Key

> Copilot-generated answers may contain inaccuracies

## Question 1

**Correct Answer**: D) Database Migration Assistant

**Why this is correct**
- Database Migration Assistant scans SQL Server databases for deprecated features, breaking changes, and compatibility issues.  
- It provides clear guidance to fix problems before migration.

**Why the others are incorrect**
- **A) Azure Migrate**:
  Azure Migrate assesses servers and virtual machines but does not check SQL Server features.
- **B) SQLPackage**:
  SQLPackage moves database schemas but does not analyze feature issues or performance blockers.
- **C) Azure Database Migration Service**:
  Azure Database Migration Service performs the migration but depends on tools like DMA for compatibility checks.

## Question 2

**Correct Answer**: C) Full and transaction log backups only

**Why this is correct**
- Azure Database Migration Service requires a full backup and a continuous chain of transaction log backups to keep the source and target in sync during an online migration.  
- This preserves data integrity without downtime.

**Why the others are incorrect**
- **A) Full backup only**:
  A full backup alone cannot capture ongoing changes during the migration.
- **B) Transaction log backups only**:
  Transaction log backups need a full backup as a base to be usable.
- **D) Differential backup only**:
  A differential backup captures changes since the last full backup but cannot maintain the transaction log chain needed for online migration.

## Question 3

**Correct Answer**: C) WITH CHECKSUM

**Why this is correct**
- WITH CHECKSUM verifies the integrity of each backup by detecting any corruption during the backup process.

**Why the others are incorrect**
- **A) WITH NOINIT**:
  WITH NOINIT appends to an existing backup set but does not validate backup integrity.
- **B) WITH UNLOAD**:
  WITH UNLOAD is used for tape devices and does not perform validation.
- **D) WITH COPY_ONLY**:
  WITH COPY_ONLY creates an independent backup without affecting the backup sequence but does not check for corruption.

## Question 4

**Correct Answer**:
A) Export the Azure Resource Manager templates from A1Dev  
B) Change the server and database names (and related variables) in the templates  
C) Deploy the updated templates to A1Test  
D) Publish the database schema and permissions from the SQL Server Data Tools database project

**Why this is correct**
- Exporting ARM templates captures the source server and database configuration.
- Editing the templates updates them for the new target resource group and database names.
- Deploying the templates creates the target resources.
- Publishing from SQL Server Data Tools applies the schema and permissions to the new database.

**Why the others are incorrect**
- **E) Add client IP addresses to the firewall on TestS1**:
  Firewall rules are needed to connect but are not part of replicating configuration and schema.
- **F) Configure the Azure Active Directory administrator on TestS1**:
  Setting an Active Directory admin helps manage access but is not required for replicating the server and database structure.

## Question 5

**Correct Answer**: D) Columnstore archival compression

**Why this is correct**
- Columnstore archival compression offers the highest compression ratio, especially for large, infrequently accessed fact tables.

**Why the others are incorrect**
- **A) Row compression**:
  Row compression reduces storage but far less than columnstore methods.
- **B) Page compression**:
  Page compression provides better savings than row compression but still less than columnstore compression.
- **C) Columnstore compression**:
  Columnstore compression saves space but archival compression saves even more.

## Question 6

**Correct Answer**: B) PolyBase

**Why this is correct**
- PolyBase is not supported in Azure SQL Database and must be removed or replaced before migration.

**Why the others are incorrect**
- **A) Clustered columnstore indexes**:
  Clustered columnstore indexes are fully supported in Azure SQL Database.
- **C) Change tracking**:
  Change tracking is supported in Azure SQL Database.
- **D) Automatic tuning**:
  Automatic tuning is supported and even enhanced in Azure SQL Database.

## Question 7

**Correct Answer**: A) Enable Azure SQL auditing and send logs to an Azure Storage account  
C) Turn on Advanced Data Security (Microsoft Defender for SQL) on the server  
D) Apply a sensitivity label of "Highly Confidential" to the column

**Why this is correct**
- Auditing records access events, including reads of sensitive columns.
- Advanced Data Security enhances monitoring and alerting for data access.
- Sensitivity labels help prioritize monitoring and protection of confidential columns.

**Why the others are incorrect**
- **B) Add extended properties to the column**:
  Extended properties store metadata but do not audit access.
- **E) Enable Azure Advanced Threat Protection (Microsoft Defender for Cloud Recommendations)**:
  Defender for Cloud gives security recommendations but does not specifically audit data access.

## Question 8

**Correct Answer**: B) Delete a row from Table1

**Why this is correct**
- Members of the db_datawriter role can insert, update, and delete rows but cannot alter tables.

**Why the others are incorrect**
- **A) Add a column to Table1**:
  Adding columns requires ALTER permissions, not provided by db_datawriter.
- **C) Delete Table1**:
  Dropping a table requires ALTER or CONTROL permissions, not db_datawriter.

## Question 9

**Correct Answer**: A) db_datareader

**Why this is correct**
- The db_datareader role grants SELECT permissions on all tables and views in the database.

**Why the others are incorrect**
- **B) db_ddladmin**:
  db_ddladmin grants rights to run DDL commands, not SELECT data.
- **C) db_denydatareader**:
  db_denydatareader explicitly denies read access.
- **D) db_denydatawriter**:
  db_denydatawriter explicitly denies write access, not related to reading.

## Question 10

**Correct Answer**: A) Set the Active Directory admin for AzSQL1  
C) Connect to DB1 by using the Active Directory admin account  
D) Create a user in DB1 using the FROM EXTERNAL PROVIDER clause

**Why this is correct**
- Setting an Active Directory admin allows AAD authentication.
- Connecting with the Active Directory admin is required to create users based on AAD identities.
- FROM EXTERNAL PROVIDER creates a contained database user authenticated by Azure AD.

**Why the others are incorrect**
- **B) Connect to DB1 by using the server administrator account**:
  The server admin cannot create AAD users without setting up an AD admin first.
- **E) From the Azure portal, assign the SQL DB Contributor role to the user**:
  Assigning Azure roles does not create database users.
- **F) Create a login in the master database**:
  Contained users do not require server-level logins.

## Question 11

**Correct Answer**: A) 0

**Why this is correct**

* Setting the Exposed Prefix to 0 ensures that no characters at the very start (digits or hyphens) remain unmasked, so the first six positions are fully replaced by the padding string.

**Why the others are incorrect**

* **B) 1**: Setting 1 would leave the very first character (“5”) visible.
* **C) 3**: Setting 3 would leave the first three characters (“555”) visible.
* **D) 5**: Setting 5 would leave the first five characters (“555-5”) visible.


## Question 12

**Correct Answer**: B) xxx-xxx

**Why this is correct**

* Using the padding string “xxx-xxx” inserts three mask characters, a hyphen, then three mask characters in place of the first six digits. When paired with an Exposed Prefix of 0 and an Exposed Suffix of 4, it yields “xxx-xxx-0173,” preserving both hyphens.

**Why the others are incorrect**

* **A) xxxxxx**: Masks six characters but drops both hyphens.
* **C) x[3]-x[3]**: Not valid syntax for the padding parameter.
* **D) xxxx-xxxx**: Inserts eight mask characters and does not match the six-digit mask requirement.

## Question 13

**Correct Answer**: D) 4

**Why this is correct**

* The `suffix` parameter specifies how many characters at the very end remain unmasked. Setting `suffix = 4` ensures that exactly the last four digits (“0173”) are visible and everything before is replaced by the padding string.

**Why the others are incorrect**

* **A) 0**: Would mask the entire string, leaving no characters visible.
* **B) 1**: Would leave only the final single character (“3”) visible.
* **C) 3**: Would leave only the last three characters (“173”) visible.

## Question 14a

**Correct Answer**: A) Create a column master key  
B) Create a column encryption key  
D) Encrypt the Salary column by using the randomized encryption type

**Why this is correct**
- A column master key protects the encryption keys.
- A column encryption key is used to encrypt the data.
- Randomized encryption provides the highest security but prevents joins or searches.

**Why the others are incorrect**
- **C) Encrypt the Salary column by using the deterministic encryption type**:
  Deterministic encryption is less secure because it allows equality searches and pattern analysis.
- **E) Enable Transparent Data Encryption**:
  Transparent Data Encryption protects data at rest, not individual column values.
- **F) Apply a dynamic data mask to the Salary column**:
  Dynamic masking hides data from queries but does not encrypt it.

## Question 14b

**Correct Answer**: A) Column encryption key

**Why this is correct**
- The column encryption key protects the Salary column both in use and in transit.

**Why the others are incorrect**
- **B) Database encryption key**:
  The database encryption key protects the entire database at rest, not at the column level.
- **C) Service master key**:
  The service master key protects other keys but is not used directly to encrypt data columns.

## Question 14c

**Correct Answer**: A) Deterministic

**Why this is correct**
- Deterministic encryption allows equality comparisons and joins on encrypted columns.

**Why the others are incorrect**
- **B) Randomized**:
  Randomized encryption prevents joins and comparisons.
- **C) Transparent Data Encryption**:
  Transparent Data Encryption applies at the database level and does not allow joins on encrypted values.

## Question 15

**Correct Answer**:
<br>B) Create users in each database
<br>E) Create logins in the master database

**Why this is correct**

* A login in the `master` database authenticates a customer at the server-level, and a corresponding user in their specific database authorizes them only there. By mapping each login to a user in exactly one database, you prevent that login from accessing any other database on the server.

**Why the others are incorrect**

* **A) Implement row-level security (RLS)**: RLS controls which rows within a table are visible, but does nothing to restrict access to entire databases.
* **C) Configure the database firewall**: While database-level firewall rules can block or allow IP ranges per database, they do not enforce per-login database access. They’re a network control, not an authorization mechanism.
* **D) Configure the server firewall**: Server-level firewall rules apply to all databases on the server and cannot isolate individual databases per customer.
* **F) Implement Always Encrypted**: Always Encrypted protects sensitive data at rest and in transit, but does not control who can connect to or select from a database.

## Question 16

**Correct Answer**:
<br>A) On the master database, run `CREATE LOGIN [A1] WITH PASSWORD = '<password>';`
<br>C) On D1, run `CREATE USER [A1] FROM LOGIN [A1];`
<br>E) On D1, run `ALTER ROLE db_datareader ADD MEMBER [A1];`

**Why this is correct**
- Creating the login in the master database enables server-level authentication.
- Creating the user in D1 maps the login to the database.
- Adding the user to db_datareader grants SELECT permissions.

**Why the others are incorrect**
- **B) CREATE LOGIN FROM EXTERNAL PROVIDER**:
  This is used for Azure Active Directory, not SQL authentication.
- **D) CREATE USER WITH PASSWORD**:
  You create users from logins, not with a new password at the database level.

## Question 17

**Correct Answer**: B) Private Link

**Why this is correct**
- Private Link connects the virtual network directly to Azure SQL Database over the Azure backbone without exposing traffic to the internet.

**Why the others are incorrect**
- **A) Azure SQL service endpoint**:
  Service endpoints secure traffic but still allow access to all servers in the subscription.
- **C) ExpressRoute gateway**:
  ExpressRoute is used for on-premises to Azure connectivity, not needed here.
- **D) VPN gateway**:
  VPN gateway connects on-premises networks, not needed between VM and Azure SQL.

## Question 18

**Correct Answer**: A) SQL Server Audit  
B) Database Audit Specification

**Why this is correct**
- SQL Server Audit captures and logs audit events at the server level.
- Database Audit Specification targets specific events like SELECT operations on sensitive tables.

**Why the others are incorrect**
- **C) Server Audit Specification**:
  Server Audit Specification audits server-level events, not specific table reads.
- **D) Extended Events Session**:
  Extended Events can monitor activity but are not designed for auditing compliance.

## Question 19

**Correct Answer**: A) Register the virtual machine with the Microsoft.SqlVirtualMachine resource provider

**Why this is correct**
- Registering the VM enables management features like automated backups, patching, and maintenance.

**Why the others are incorrect**
- **B) Enable a system-assigned managed identity**:
  This supports Azure services authentication but does not configure SQL-specific features.
- **C) Install Desired State Configuration extension**:
  This enforces configuration but does not enable SQL management features.
- **D) Register with Microsoft.Sql resource provider**:
  This provider manages Azure SQL Database services, not SQL Server on VMs.

## Question 20

**Correct Answer**: C) Set Threshold Sensitivity to Low  
E) Set the alert logic threshold to Dynamic

**Why this is correct**
- Setting sensitivity to Low reduces false positives from small fluctuations.
- Dynamic thresholds adapt automatically to baseline usage patterns.

**Why the others are incorrect**
- **A) Set the alert logic threshold to Static**:
  Static thresholds do not adjust based on normal fluctuations.
- **B) Enable Force Plan**:
  Force Plan controls execution plans, not alerting.
- **D) Set Threshold Sensitivity to High**:
  High sensitivity would increase false alerts for minor variations.

## Question 21

**Correct Answer**: B) DBCC SHRINKDATABASE

**Why this is correct**
- DBCC SHRINKDATABASE reclaims unused space from all database files at once.

**Why the others are incorrect**
- **A) DBCC SHRINKFILE**:
  DBCC SHRINKFILE reclaims space from a single file, not the whole database.
- **C) sp_clean_db_free_space**:
  This procedure zeros out free space for security but does not reclaim it.
- **D) sp_clean_db_file_free_space**:
  This zeros out free space in a specific file, not across the database.

## Question 22

**Correct Answer**: B) Dynamic management view queries

**Why this is correct**
- Dynamic management views (DMVs) expose real-time details about TempDB usage and contention.

**Why the others are incorrect**
- **A) Query Performance Insight**:
  Query Performance Insight shows workload patterns, not TempDB internals.
- **C) SQL Server Profiler traces**:
  Profiler can capture queries but is not efficient for TempDB analysis.
- **D) Query Store-based queries**:
  Query Store tracks query performance, not TempDB behavior.

## Question 23

**Correct Answer**: A) A storage pool  
D) A virtual disk that uses the stripe layout  
E) A volume

**Why this is correct**
- A storage pool aggregates the disks.
- A striped virtual disk combines performance across multiple disks.
- A volume formats and presents the virtual disk to the OS.

**Why the others are incorrect**
- **B) A virtual disk that uses the simple layout**:
  Simple layout uses only one disk and cannot combine IOPS.
- **C) A virtual disk that uses the mirror layout**:
  Mirror layout duplicates data, reducing usable performance.

## Question 24

**Correct Answer**: C) Double the Query Store statistics collection interval

**Why this is correct**
- Doubling the statistics collection interval reduces the frequency of data collection, lowering the volume of data stored.

**Why the others are incorrect**
- **A) Halve the Query Store data flush interval**:
  Halving flush intervals increases the frequency of writes, not helpful here.
- **B) Double the Query Store data flush interval**:
  Flush intervals control when data is persisted, not how much is collected.
- **D) Halve the Query Store statistics collection interval**:
  Halving it would capture more data, making the storage problem worse.

## Question 25

**Correct Answer**: A) 8

**Why this is correct**
- Microsoft recommends starting with one TempDB data file per 4 vCPUs and adjusting based on contention, so 16 vCPUs suggests 4 files initially, but with heavy contention or high concurrency, scaling to 8 is standard.

**Why the others are incorrect**
- **B) 64**: 64 TempDB files would be excessive for 16 vCPUs
- **C) 2**: 2 files are too few and would likely cause contention
- **D) 4**: 4 files might work initially, but 8 gives better load distribution under high workload

## Question 26

**Correct Answer**: D) sys.resource\_stats

**Why this is correct**

- The `sys.resource_stats` view in the master database captures CPU, I/O and storage metrics every five minutes and retains approximately 14 days of history, making it ideal for querying the past week’s resource usage

**Why the others are incorrect**

- **A) sys.dm\_db\_resource\_stats**: returns the same metrics but only maintains about one hour of history
- **B) sys.dm\_exec\_requests**: shows details about currently executing queries, not historical resource metrics
- **C) sys.dm\_user\_db\_resource\_governance**: reports the configuration and capacity settings of resource governance, not past usage data

## Question 27

**Correct Answer**: A) DATEADD(day, -7, GETDATE())

**Why this is correct**
- DATEADD subtracts seven days from the current date to get the correct range.

**Why the others are incorrect**
- **B) DATEDIFF(day, -7, GETDATE())**:
  DATEDIFF returns the number of days between two dates, not a new date.
- **C) DATEPART(day, -7, GETDATE())**:
  DATEPART extracts parts of a date, not adjusts the date.
- **D) TODATETIMEOFFSET(day, -7, GETDATE())**:
  TODATETIMEOFFSET adjusts time zone offsets, not date arithmetic.

## Question 28

**Correct Answer**: D) Create an activity log alert rule

**Why this is correct**
- Activity log alerts capture management events like configuration changes across Azure resources.

**Why the others are incorrect**
- **A) Create a metric alert rule**:
  Metric alerts monitor performance counters, not configuration changes.
- **B) Configure a diagnostic setting for advanced metrics**:
  Diagnostic settings export metrics but do not generate alerts by themselves.
- **C) Forward security logs via a diagnostic setting**:
  Forwarding security logs is unrelated to monitoring configuration changes.

## Question 29

**Correct Answer**: C) SQL Servers

**Why this is correct**
- Scoping the alert to the SQL Server resource automatically includes all current and future databases under it.

**Why the others are incorrect**
- **A) SQL Databases**:
  Scoping to individual databases would require manually updating the alert when new databases are added.
- **B) Resource Groups**:
  Scoping to a resource group could capture unrelated resources.
- **D) SQL Virtual Machines**:
  SQL Virtual Machines host SQL Server instances but are not related to Azure SQL Database services.

## Question 30

**Correct Answer**: A) Yes

**Why this is correct**
- Forcing the plan eliminates re-evaluation by the optimizer, reducing I/O and improving execution time because the plan already uses an efficient Index Seek and Key Lookup.

**Why the others are incorrect**
- **B) No**:
  Not forcing the plan could allow the optimizer to pick worse alternatives, increasing I/O.

## Question 31

**Correct Answer**: B) No

**Why this is correct**
- The new nonclustered index eliminates the need for a Key Lookup, reducing I/O and execution time compared to the original plan.

**Why the others are incorrect**
- **A) Yes**:
  I/O and execution time decrease, not increase, when the Key Lookup is eliminated.

## Question 32

**Correct Answer**: A) Yes

**Why this is correct**
- Including necessary columns in the clustered index allows the query to be satisfied directly without additional lookups, reducing I/O and execution time.

**Why the others are incorrect**
- **B) No**:
  Modifying the index improves performance, not worsens it.

## Question 33

**Correct Answer**: A) Change Salary to the money data type  
E) Change LastHireDate to date

**Why this is correct**
- Using money reduces storage and provides correct precision for salaries.
- Using date instead of datetime saves space and matches the actual data need.

**Why the others are incorrect**
- **B) Change PhoneNumber to float**:
  Float is unsuitable for phone numbers and would introduce rounding errors.
- **C) Change LastHireDate to datetime2(7)**:
  datetime2(7) is more precise but larger; date is a better fit.
- **D) Change PhoneNumber to bigint**:
  Bigint would remove hyphens and complicate formatting.

## Question 34

**Correct Answer**: C) Live Query Statistics

**Why this is correct**
- Live Query Statistics provides real-time operator-level progress during query execution.

**Why the others are incorrect**
- **A) Estimated execution plan**:
  Estimated plans show potential execution paths, not real-time progress.
- **B) Actual execution plan**:
  Actual plans are available only after query completion.
- **D) Client Statistics**:
  Client Statistics shows query timing but not detailed operator progress.

## Question 35

**Correct Answer**: D) Query Performance Insight

**Why this is correct**
- Query Performance Insight identifies resource-intensive queries on Azure Database for MySQL.

**Why the others are incorrect**
- **A) Query Store**:
  Query Store is not available in Azure Database for MySQL.
- **B) Azure Monitor Metrics**:
  Metrics track server-level performance, not individual query performance.
- **C) Alerts**:
  Alerts notify about threshold breaches but do not analyze queries.

## Question 36

**Correct Answer**: C) Enable Last_Query_Plan_Stats in DB1  
B) Enable Lightweight_Query_Profiling in DB1

**Why this is correct**
- Last_Query_Plan_Stats captures execution plan details including parameter values.
- Lightweight_Query_Profiling enables lightweight real-time query profiling without major overhead.

**Why the others are incorrect**
- **A) Enable Last_Query_Plan_Stats in the master database**:
  Enabling it in master does not capture stats for DB1.
- **D) Enable Lightweight_Query_Profiling in the master database**:
  Needs to be enabled in the target database (DB1), not master.
- **E) Enable PARAMETER_SNIFFING in DB1**:
  Parameter sniffing affects plan reuse but does not capture parameter values.

## Question 37

**Correct Answer**: C) ON (OPERATION_MODE = READ_WRITE)

**Why this is correct**
- Setting Query Store to ON with READ_WRITE mode allows it to capture and persist query performance data.

**Why the others are incorrect**
- **A) OFF**:
  Turns off Query Store entirely.
- **B) ON (OPERATION_MODE = READ_ONLY)**:
  Allows reading data but not capturing new query performance data.
- **D) AUTO**:
  AUTO is not a valid value for OPERATION_MODE.

## Question 38

**Correct Answer**: B) FORCE_LAST_GOOD_PLAN = ON

**Why this is correct**
- Enabling FORCE_LAST_GOOD_PLAN automatically reverts queries to their last good execution plan if regression is detected.

**Why the others are incorrect**
- **A) FORCE_LAST_GOOD_PLAN = OFF**:
  Disables automatic plan correction.
- **C) AUTO**:
  AUTO is not a valid setting for this configuration.
- **D) OFF**:
  OFF disables the feature.

## Question 39

**Correct Answer**: D) Set READ_COMMITTED_SNAPSHOT to ON

**Why this is correct**
- READ_COMMITTED_SNAPSHOT uses row versioning to prevent readers from blocking writers and vice versa.

**Why the others are incorrect**
- **A) Set PARAMETERIZATION to FORCED**:
  PARAMETERIZATION controls query plan reuse, not concurrency.
- **B) Set Delayed Durability to FORCED**:
  Delayed durability affects transaction commit behavior, not blocking.
- **C) Set PARAMETERIZATION to SIMPLE**:
  SIMPLE is not a valid value for PARAMETERIZATION.

## Question 40

**Correct Answer**: C) OPTIMIZE_FOR_AD_HOC_WORKLOADS

**Why this is correct**
- Enabling OPTIMIZE_FOR_AD_HOC_WORKLOADS stores a lightweight plan stub for first-time queries, reducing memory pressure.

**Why the others are incorrect**
- **A) LEGACY_CARDINALITY_ESTIMATION**:
  Affects query plans, not memory use for ad-hoc queries.
- **B) QUERY_OPTIMIZER_HOTFIXES**:
  Enables fixes but does not reduce memory usage for ad-hoc plans.
- **D) ACCELERATED_PLAN_FORCING**:
  Helps force efficient plans but not related to ad-hoc workload optimization.

## Question 41

**Correct Answer**: A) Force Plan 1221065

**Why this is correct**
- Forcing the faster plan (average duration ~5 ms) ensures the query consistently uses the most efficient execution plan.

**Why the others are incorrect**
- **B) Run DBCC FREEPROCCACHE**:
  Clearing the plan cache could remove good plans and cause instability.
- **C) Force Plan 1220917**:
  This forces the slower plan (~230 ms), worsening performance.
- **D) Disable parameter sniffing**:
  Disabling parameter sniffing changes optimization behavior but does not guarantee using the faster plan.

## Question 42a

**Correct Answer**: A) Yes

**Why this is correct**
- With Create Index set to ON, Azure SQL Database can automatically create nonclustered indexes to improve performance.

## Question 42b

**Correct Answer**: B) No

**Why this is correct**
- Azure SQL Automatic Tuning creates new indexes but does not modify existing indexes by adding columns.

## Question 42c

**Correct Answer**: A) Yes

**Why this is correct**
- Force Plan = ON allows Azure SQL Database to revert to a last known good plan automatically if a regression is detected.

## Question 43

**Correct Answer**: C) Save the actual execution plan

**Why this is correct**
- Saving the initial actual execution plan allows you to compare it against the Live Query Statistics after changes.

**Why the others are incorrect**
- **A) Enable Query Store and set QUERY_CAPTURE_MODE to ALL**:
  Useful for long-term capture but not for immediate plan comparison.
- **B) Run SET SHOWPLAN_ALL ON**:
  Only shows estimated plans and does not capture the actual execution.
- **D) Enable Query Store for the database**:
  Helps track historical plans but does not save specific execution plan files for comparison.

## Question 44

**Correct Answer**: C) Query Store in SQL Server Management Studio

**Why this is correct**
- Query Store provides an easy way to view and compare previous and current execution plans with minimal effort.

**Why the others are incorrect**
- **A) Extended Events in SQL Server Management Studio**:
  Extended Events are more complex and require custom session setup.
- **B) Performance Recommendations in the Azure portal**:
  Gives suggestions but does not show previous execution plans.
- **D) Query Performance Insight in the Azure portal**:
  Shows workload patterns, not detailed plan comparisons.

## Question 45

**Correct Answer**: B) Queries will read fewer data pages  
E) Queries will consume more CPU resources

**Why this is correct**
- Archival compression reduces storage size so fewer pages are read.
- Decompressing archived data on query increases CPU usage.

**Why the others are incorrect**
- **A) Each query will perform more disk I/O**:
  Disk I/O is reduced because there are fewer pages.
- **C) The index will occupy more space on disk**:
  Archival compression shrinks storage, not expands it.
- **D) The index will use more memory at runtime**:
  Compression affects CPU, not memory usage directly.

## Question 46

**Correct Answer**: A) Enable Database Mail  
B) Enable the email settings for SQL Server Agent  
D) Create a job notification

**Why this is correct**
- Database Mail must be enabled to send emails.
- SQL Server Agent must be configured to use Database Mail.
- A job notification must be set up to send an email when the job fails.

**Why the others are incorrect**
- **C) Create a job alert**:
  Alerts trigger on specific error conditions, but you need a notification tied to the job outcome.
- **E) Create a job target**:
  Job targets define job scope, not notifications.

## Question 47

**Correct Answer**: B) Create an Azure Policy initiative that groups the 20 definitions  
C) Create an Azure Policy initiative assignment at the subscription scope  
D) Run Azure Policy remediation tasks for existing non-compliant resources

**Why this is correct**
- Grouping the policies into an initiative makes management easier.
- Assigning at subscription scope applies policies to all current and future resources.
- Remediation tasks fix non-compliant resources automatically.

**Why the others are incorrect**
- **A) Duplicate the built-in policy definitions**:
  Duplication is unnecessary unless customization is needed.
- **E) Create an Azure Blueprints assignment using the initiative**:
  Azure Blueprints is a broader governance tool, not required for simple policy enforcement.

## Question 48

**Correct Answer**: C) Create a Database Mail profile in SQL Server Management Studio

**Why this is correct**
- SQL Server Agent uses a Database Mail profile to send job completion emails.

**Why the others are incorrect**
- **A) Enable SQL Server Agent in Configuration Manager**:
  Necessary to run jobs, but not related to email notifications.
- **B) Run sp_set_sqlagent_properties**:
  This can configure agent properties but does not create a mail profile.
- **D) Create an Azure Monitor action group with email/SMS/Push/Voice notifications**:
  Azure Monitor action groups are used for Azure resources, not for SQL Server Agent jobs.

## Question 49

**Correct Answer**: B) No

**Why this is correct**
- Simply deleting the database and restoring from another server's backup is not supported directly in the Azure portal; geo-restore, point-in-time restore, or other managed options must be used.

## Question 50a

**Correct Answer**: A) Yes

**Why this is correct**
- Using Remove-AzSqlDatabase followed by Restore-AzSqlDatabase from a different server instance correctly replaces the database.

## Question 50b

**Correct Answer**: B) No

**Why this is correct**
- RESTORE DATABASE FROM URL is not supported for Azure SQL Database — it is only available for SQL Server in a VM or Managed Instance.

## Question 50c

**Correct Answer**: A) Yes

**Why this is correct**
- Renaming the existing DB1, restoring DB1 from the source, and then deleting the old database properly replaces the database.

## Question 51

**Correct Answer**: C) Use the COPY_ONLY option

**Why this is correct**
- COPY_ONLY allows a backup from a secondary replica without disrupting the backup chain on the primary.

**Why the others are incorrect**
- **A) Use the DIFFERENTIAL option**:
  Differential backups depend on full backups and do not avoid impacting the backup chain.
- **B) Use the NOINIT option**:
  NOINIT controls whether backups are appended but does not affect backup chain integrity.
- **D) Use the FILE_SNAPSHOT option**:
  FILE_SNAPSHOT is specific to Azure Blob Storage backups, not Always On replicas.

## Question 52

**Correct Answer**: B) Azure SQL Database elastic pool

**Why this is correct**
- Elastic pools automatically scale resources across databases based on workload demand with minimal downtime.

**Why the others are incorrect**
- **A) Azure SQL Managed Instance**:
  Managed Instances are suited for lift-and-shift scenarios but do not autoscale like elastic pools.
- **C) Single Azure SQL databases**:
  Single databases scale individually but not as a pooled resource.
- **D) SQL Server on Azure virtual machines**:
  SQL Server VMs require manual scaling.

## Question 53

**Correct Answer**: C) Azure Backup

**Why this is correct**
- Azure Backup can capture SQL native backups at the database level and restore them individually.

**Why the others are incorrect**
- **A) Azure Site Recovery**:
  Site Recovery replicates VMs for disaster recovery, not for database backups.
- **B) SQL Server Agent jobs**:
  Agent jobs require custom setup and do not offer centralized backup management.
- **D) Automated backup in the SQL virtual machine settings**:
  This covers basic backup but lacks centralized restore flexibility across instances.

## Question 54

**Correct Answer**: C) Failover groups

**Why this is correct**
- Failover groups allow automatic failover across regions without needing clients to change their connection strings.

**Why the others are incorrect**
- **A) Geo-replication**:
  Geo-replication requires manual failover and connection string updates.
- **B) Availability Zones**:
  Availability Zones provide regional redundancy, not cross-region failover.
- **D) Transactional replication**:
  Transactional replication is not designed for high availability or automatic failover.

## Question 55

**Correct Answer**: A) Sunday full backup  
C) Wednesday differential backup  
D) Log backups from Wednesday 1:05 AM through 10:00 AM

**Why this is correct**
- Restore the latest full backup first.
- Then apply the most recent differential backup (Wednesday's).
- Finally, apply all transaction logs up to the point of failure.

**Why the others are incorrect**
- **B) Tuesday differential backup**:
  Tuesday’s differential is outdated once a newer (Wednesday) differential is available.
- **E) Log backups from Tuesday 1:05 AM through 1:00 PM**:
  These are already included in the Wednesday differential backup and would be redundant.

## Question 56

**Correct Answer**: C) Read-access geo-redundant storage (RA-GRS)

**Why this is correct**
- RA-GRS provides geo-redundant storage with read access, ensuring backups are available even if a region fails, at a lower cost than GZRS options.

**Why the others are incorrect**
- **A) Geo-redundant storage (GRS)**:
  GRS replicates data but does not allow read access to the secondary.
- **B) Zone-redundant storage (ZRS)**:
  ZRS protects against zone failures, not regional outages.
- **D) Locally-redundant storage (LRS)**:
  LRS keeps copies within a single datacenter, not across regions.

## Question 57

**Correct Answer**: B) Restore with NORECOVERY

**Why this is correct**
- Restoring with NORECOVERY keeps the database in a restoring state, allowing additional log backups to be applied for Always On readiness.

**Why the others are incorrect**
- **A) Restore with Standby**:
  Standby allows read-only access but is not ideal for Always On.
- **C) Restore with RECOVERY**:
  Recovery finalizes the restore process and does not allow further log restores.

## Question 58

**Correct Answer**: A) On a region-wide outage, all databases will automatically fail over  
D) Failover to the secondary is immediate upon any service interruption

**Why this is correct**
- Failover groups automatically switch to secondary region databases during outages.
- Failover can happen immediately depending on the grace period configuration.

**Why the others are incorrect**
- **B) Failover will not commence until at least one hour has passed since outage detection**:
  Failover timing is configurable and can occur much faster.
- **C) You can selectively fail over individual databases in the group**:
  Failover groups operate on all databases together, not individually.
- **E) You can configure different grace periods for each database in the group**:
  Grace period is set at the group level, not per database.

## Question 59

**Correct Answer**: D) Azure SQL Database Premium tier

**Why this is correct**
- The Premium tier supports zone redundancy and active geo-replication, meeting requirements for zero data loss and regional failover.

**Why the others are incorrect**
- **A) Azure SQL Database Standard tier**:
  Standard does not offer active geo-replication.
- **B) Azure SQL Database serverless tier**:
  Serverless is cost-efficient but not designed for high availability.
- **C) Azure SQL Managed Instance Business Critical tier**:
  Business Critical tier is powerful but more expensive than Premium SQL Database when minimal cost is a factor.

## Question 60a

**Correct Answer**: B) No

**Why this is correct**
- General Purpose service tier with geo-replication only provides one readable secondary, not two.

## Question 60b

**Correct Answer**: A) Yes

**Why this is correct**
- Business Critical tier with Availability Zones provides at least two readable replicas and maintains availability across zones.

## Question 60c

**Correct Answer**: B) No

**Why this is correct**
- Auto-failover groups provide one readable secondary, but not two during normal operations in the General Purpose tier.

## Question 61

**Correct Answer**: B) Azure SQL Database (single database)

**Why this is correct**
- Azure SQL Database supports Azure SQL Data Sync across multiple regional instances with minimal management effort.

**Why the others are incorrect**
- **A) SQL Server on Azure Virtual Machines**:
  SQL Server VMs require manual configuration and maintenance for Data Sync.
- **C) Azure SQL Managed Instance**:
  Managed Instances do not support Azure SQL Data Sync natively.
- **D) Azure Database for PostgreSQL**:
  PostgreSQL does not use Azure SQL Data Sync.

## Question 62

**Correct Answer**: C) GRANT VIEW DEFINITION TO [User]

**Why this is correct**
- VIEW DEFINITION permission allows the user to see database metadata without accessing the data.

**Why the others are incorrect**
- **A) GRANT SELECT ON DATABASE::DB1 TO [User]**:
  SELECT permission gives access to the data itself, not metadata.
- **B) GRANT SHOWPLAN TO [User]**:
  SHOWPLAN permission allows viewing execution plans, not database properties.
- **D) GRANT VIEW DATABASE STATE TO [User]**:
  VIEW DATABASE STATE allows seeing dynamic activity like running queries, not just metadata.

## Question 63

**Correct Answer**: C) SINGLE_USER

**Why this is correct**
- Setting the database to SINGLE_USER mode with ROLLBACK IMMEDIATE forces other connections to close, allowing maintenance.

**Why the others are incorrect**
- **A) OFFLINE**:
  OFFLINE makes the database unavailable but is not required for repair preparation.
- **B) ONLINE**:
  ONLINE is the normal operating state.
- **D) TRUSTWORTHY**:
  TRUSTWORTHY controls code execution permissions, not connection behavior.

## Question 64

**Correct Answer**: D) REPAIR_ALLOW_DATA_LOSS

**Why this is correct**
- REPAIR_ALLOW_DATA_LOSS fixes corruption even if it requires removing damaged data.

**Why the others are incorrect**
- **A) PHYSICAL_ONLY**:
  PHYSICAL_ONLY limits checking to physical structure but does not repair.
- **B) REPAIR_FAST**:
  REPAIR_FAST is deprecated and does not perform real repair.
- **C) REPAIR_REBUILD**:
  REPAIR_REBUILD can fix only minor issues like missing indexes or nonclustered index corruption, but it cannot fix serious data corruption.

## Question 65

**Correct Answer**: A) MULTI_USER

**Why this is correct**
- Setting MULTI_USER allows multiple users to reconnect to the database after repairs are complete.

**Why the others are incorrect**
- **B) ONLINE**:
  ONLINE refers to database availability but does not control user connections.
- **C) OPEN**:
  OPEN is not a valid database state setting.
- **D) TRUSTWORTHY**:
  TRUSTWORTHY has no impact on connection availability.

## Question 66

**Correct Answer**: C) TO URL = 'https://storage1.blob.core.windows.net/blob1/Sales.bak'

**Why this is correct**
- Backups to Azure Blob Storage require specifying a URL destination.

**Why the others are incorrect**
- **A) TO DISK = 'X:\BAK\Sales.bak'**:
  This is a local disk path, not Blob Storage.
- **B) TO 'Sales_Backup'**:
  This is not a valid T-SQL syntax for specifying backup location.
- **D) TO DISK = '\\BackupSystem\BackupDisk1\Sales.bak'**:
  This is a network share path, not Azure Blob Storage.

## Question 67

**Correct Answer**: D) WITH COPY_ONLY

**Why this is correct**
- WITH COPY_ONLY creates a backup that does not affect the regular backup sequence.

**Why the others are incorrect**
- **A) WITH FILE_SNAPSHOT**:
  FILE_SNAPSHOT is used for snapshot backups, mostly on Azure Storage.
- **B) WITH ENCRYPTION**:
  WITH ENCRYPTION secures the backup but does not affect backup chaining.
- **C) WITH NO_TRUNCATE**:
  NO_TRUNCATE reads inactive portions of the log but does not create an independent backup.

## Question 68

**Correct Answer**: C) ColumnC

**Why this is correct**
- ColumnC subtracts used space from total space, showing free (unused) space.

**Why the others are incorrect**
- **A) ColumnA**:
  ColumnA shows the total file size.
- **B) ColumnB**:
  ColumnB shows the used space only.
- **D) ColumnD**:
  ColumnD shows the maximum file size allowed.

## Question 69

**Correct Answer**: C) ColumnC

**Why this is correct**
- Same as 68 — ColumnC represents the unused space per file.

## Question 70

**Correct Answer**: C) Premium 4-vCore

**Why this is correct**
- Premium 4-vCore DMS tier is required for online migrations of large databases with minimal downtime and performance impact.

**Why the others are incorrect**
- **A) Standard 2-vCore**:
  Standard 2-vCore supports offline migrations only and is insufficient for large workloads.
- **B) Standard 4-vCore**:
  Standard 4-vCore still only supports offline migrations.

## Question 71

**Correct Answer**: C) A virtual network configured with service endpoints

**Why this is correct**
- Service endpoints allow secure, private access from a virtual network to Azure SQL Database without exposing traffic to the public internet.

**Why the others are incorrect**
- **A) An Azure Logic App**:
  Logic Apps automate workflows but do not provide network connectivity.
- **B) A VPN gateway**:
  A VPN connects on-premises to Azure, not needed for VM-to-Azure SQL communication inside Azure.
- **D) Private Link**:
  Private Link is another correct way to secure SQL traffic but the question specifically leads toward service endpoints.

## Question 72

**Correct Answer**: C) vCore

**Why this is correct**
- The vCore purchasing model lets you scale compute and storage independently and allows reserved instance pricing for multiple databases.

**Why the others are incorrect**
- **A) Azure virtual machine reserved instances**:
  Reserved instances apply to VMs, not Azure SQL Database.
- **B) DTU**:
  DTU model bundles compute, memory, and storage together, so they cannot be scaled independently.

## Question 73

**Correct Answer**: C) An Azure SQL Database elastic pool

**Why this is correct**
- Elastic pools let you group multiple single databases together to share resources and elastically scale based on demand.

**Why the others are incorrect**
- **A) An Azure SQL Database managed instance**:
  Managed Instances are for lift-and-shift of entire SQL Server instances, not elastic pools.
- **B) A SQL Server Always On availability group**:
  Availability Groups are for high availability and disaster recovery, not resource sharing.

## Question 74

**Correct Answer**: A) CREATE USER [User] FROM EXTERNAL PROVIDER

**Why this is correct**
- This T-SQL statement creates a contained database user authenticated through Azure Active Directory.

**Why the others are incorrect**
- **B) CREATE USER [User] FROM LOGIN [User]**:
  This maps to an existing server-level login, not an external provider.
- **C) CREATE LOGIN [User] FROM WINDOWS**:
  Used for on-premises Active Directory, not Azure Active Directory.
- **D) CREATE USER [User] FROM ASYMMETRIC KEY**:
  Asymmetric keys are for encryption, not authentication.
- **E) CREATE USER [User] FROM CERTIFICATE**:
  Certificates are also for encryption, not authentication.

## Question 75

**Correct Answer**: A) Create an Azure Entra administrator on the logical server  
B) Enable contained database authentication on the server  
D) Create contained database users in each user database

**Why this is correct**
- You must set an Azure Active Directory (now Entra) admin at the server level.
- Contained database authentication must be enabled to allow Azure AD users.
- Finally, users must be created individually in each database.

**Why the others are incorrect**
- **C) Connect to a database using the Azure Entra administrator account**:
  Necessary for operations but not part of setup.
- **E) Create logins in the master database**:
  Contained users do not require logins in master.
- **F) Modify the existing server administrator account**:
  Modifying the admin account is unrelated to enabling AAD authentication.

## Question 76a

**Correct Answer**: B) No

**Why this is correct**
- The SQL DB Contributor role grants resource management permissions, not database-level login access.

## Question 76b

**Correct Answer**: B) No

**Why this is correct**
- SQL DB Contributor allows managing database resources but does not grant permission to assign roles to other users.

## Question 76c

**Correct Answer**: A) Yes

**Why this is correct**
- The Contributor role allows creating and managing Azure SQL databases on the logical server.

## Question 77

**Correct Answer**: D) The server, the elastic pool, and the database

**Why this is correct**
- Selecting all three scopes ensures that metrics are collected at every level of the resource hierarchy.

**Why the others are incorrect**
- **A) The database only**:
  This misses server and pool-level metrics.
- **B) The elastic pool and the database**:
  This excludes server-level metrics.
- **C) The elastic pool only**:
  This misses database-specific metrics.

## Question 78

**Correct Answer**: C) Azure Event Hubs

**Why this is correct**
- Event Hubs enables real-time streaming of metrics into event ingestion pipelines.

**Why the others are incorrect**
- **A) Azure Storage**:
  Storage collects data but does not support real-time streaming.
- **B) Azure Log Analytics**:
  Log Analytics analyzes and queries metrics but is not a streaming destination.

## Question 79

**Correct Answer**: C) sys.dm_exec_compute_node_errors  
E) sys.dm_pdw_nodes_os_wait_stats

**Why this is correct**
- sys.dm_exec_compute_node_errors helps diagnose compute node issues.
- sys.dm_pdw_nodes_os_wait_stats identifies bottlenecks based on wait types across nodes.

**Why the others are incorrect**
- **A) sys.dm_tran_locks**:
  Shows lock information, not compute node performance.
- **B) sys.dm_exec_requests**:
  Tracks active requests, not distributed performance.
- **D) sys.dm_cdc_errors**:
  Related to Change Data Capture, not analytics database performance.
- **F) sys.dm_pdw_nodes_tran_locks**:
  Shows locking, not overall node bottlenecks.