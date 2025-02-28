# SQL: Fundamentals  

This lab offers hands-on experience with core administrative tasks—covering server setup, maintenance, troubleshooting, database creation, and user/permission management. It is structured to support both on-premises SQL Server and Azure SQL environments, with exam-style questions to reinforce key concepts. Enhancements include updated guidance on modern T-SQL commands, detailed troubleshooting steps, and best practices for both environments.

## Objectives

- Fundamentals of Database Administration  
  - Master server installation, configuration, and routine maintenance.  
  - Learn to monitor and adjust performance settings, including memory, CPU, and error logs.

- Database Creation, Users & Permissions  
  - Create databases using T-SQL and the SSMS GUI.  
  - Understand the difference between logins and database users, and practice mapping logins to users with appropriate roles.  
  - Compare legacy commands (e.g., `sp_addrolemember`) with modern alternatives (e.g., `ALTER ROLE ... ADD MEMBER`)—noting compatibility and exam expectations.

- Exam Guidance & Best Practices  
  - Develop familiarity with exam-style scenarios such as user authentication, role membership, and troubleshooting access issues.  
  - Reinforce learning with exam-style questions that mirror real-world and exam scenarios.

- Dual Platform Coverage  
  - On-Premises SQL Server: Detailed procedures, updated best practices, and troubleshooting guidelines for a traditional on-prem environment.  
  - Azure SQL: Essential tasks including setup via the Azure portal, firewall configurations, and diagnostic tools alongside SSMS.

------------------------- ------------------------- ------------------------- -------------------------

## SQL Server (on-prem)

### Virtual Machine

Instantiate an Azure Virtual Machine (or equivalent):
  * Image: "Free SQL Server License: SQL Server 2022 Developer on Windows Server 2022"
  * SKU: "Standard_D2s_v3..."
  * Inbound Ports: RDP (3389) allowed
  * Boot Diagnostics: Disable

### SQL Server Management Studio

On your virtual machine, launch SQL Server Management Studio and connect to your SQL Server instance.  

<img src="https://github.com/user-attachments/assets/f1238467-a0f9-4569-936b-79f19b1351ff" width="400" title="Snipped February 27, 2025" />

Click "New Query" and execute the following T-SQL to verify connectivity and confirm the server version:  

```sql
SELECT @@VERSION;
```

<img src="https://github.com/user-attachments/assets/f0fbb736-ba1b-47dc-83d5-46e54e1a83cd" width="800" title="Snipped February 27, 2025" />

### Server Properties

Right-click the server node and select "Properties" from the resulting dropdown and review settings on resulting pages.

#### General

<img src="https://github.com/user-attachments/assets/20b04821-e1a5-408c-b5a1-8d06ba8e9275" width="500" title="Snipped February 27, 2025" />

- **Server Name** – The name of the SQL Server instance  
- **Product** – The edition and version of SQL Server  
- **Operating System** – The OS hosting the SQL Server instance  
- **Platform** – The underlying platform (e.g., Windows)  
- **Version** – The specific build or patch level of SQL Server  
- **Language** – The language setting for the server  
- **Memory** – The amount of memory recognized or allocated by SQL Server  
- **Root Directory** – The installation path for SQL Server binaries  
- **Collation** – The default collation defining how text is sorted and compared  
- **Is XTP Supported** – Indicates whether In-Memory OLTP (XTP) is supported  
- **Is HADR Enabled** – Reflects whether high availability/disaster recovery (Always On) is enabled

------------------------- -------------------------

#### Memory

<img src="https://github.com/user-attachments/assets/037448cf-cca6-4bc8-8d17-db6a0fbf590a" width="500" title="Snipped February 27, 2025" />

- **Server memory options (Minimum server memory in MB)** – The lowest amount of memory SQL Server will attempt to keep allocated  
- **Server memory options (Maximum server memory in MB)** – The highest amount of memory SQL Server can consume  
- **Other memory options (Index creation memory in KB)** – The memory allocated for index creation operations (if set to 0, SQL Server manages it automatically)  
- **Other memory options (Minimum memory per query in KB)** – The minimum memory each query can use during execution  
- **Configured values** – Displays the memory settings that have been explicitly configured  
- **Running values** – Displays the memory settings currently in effect on the running instance  

------------------------- -------------------------

#### Processors

<img src="https://github.com/user-attachments/assets/71623999-94a4-4c6f-948e-8fe9b28d46e2" width="500" title="Snipped February 27, 2025" />

- **Enable processors (Automatically set processor affinity mask for all processors)** – Determines if SQL Server automatically manages CPU affinity for all available processors  
- **Automatically set I/O affinity mask for all processors** – Determines if SQL Server automatically manages I/O affinity across all processors  
- **Processor / Processor Affinity / I/O Affinity** – Allows manual configuration of which processors handle CPU and I/O tasks  
- **Maximum worker threads** – Sets the upper limit of worker threads SQL Server can use  
- **Configured values** – The settings explicitly defined in this dialog  
- **Running values** – The current settings in effect on the running SQL Server instance

------------------------- -------------------------

#### Security

<img src="https://github.com/user-attachments/assets/dbd96ba9-ab50-433e-9c11-0e4c6a718281" width="500" title="Snipped February 27, 2025" />

- **Server authentication (Windows Authentication mode)** – Restricts authentication to Windows accounts/groups only.  
- **Server authentication (SQL Server and Windows Authentication mode)** – Allows both Windows logins and SQL Server logins to authenticate.  

- **Login auditing (None)** – No login events are logged.  
- **Login auditing (Failed logins only)** – Logs only failed login attempts.  
- **Login auditing (Successful logins only)** – Logs only successful login attempts.  
- **Login auditing (Both failed and successful logins)** – Logs all login attempts, both failed and successful.  

- **Server proxy account (Enable server proxy account)** – Lets you specify a security context (account) for operations like SQL Agent job steps.  
- **Server proxy account (Proxy account)** – The user or credential used as the proxy account.  
- **Server proxy account (Password)** – The password associated with the proxy account.  

- **Enable Common Criteria compliance** – Enables additional security features to meet Common Criteria standards.  
- **Enable c2 audit tracing** – Activates a legacy auditing standard that logs all SQL Server events to a trace file.  
- **Cross database ownership chaining** – Allows objects owned by the same user in different databases to reference each other without requiring extra permissions.

------------------------- -------------------------

#### Connections

<img src="https://github.com/user-attachments/assets/26573867-beac-4727-a4b0-61b5519bf7f4" width="500" title="Snipped February 27, 2025" />

- **Maximum number of concurrent connections (0 = unlimited)** – Sets the maximum number of simultaneous user connections to SQL Server  
- **Use query governor to prevent long-running queries** – When enabled, imposes a time limit on query execution, rejecting queries that exceed the specified cost limit  
- **Default connection options** – A list of optional behaviors (e.g., implicit transactions, ANSI settings) that are applied to every new connection  
- **Allow remote connections to this server** – Determines whether the SQL Server instance can accept connections from remote clients  
- **Remote query timeout (in seconds)** – The duration after which a remote query is canceled if it does not complete  
- **Require distributed transactions for server-to-server communication** – Enforces distributed transaction support for queries that involve multiple servers  
- **Configured values** – Displays the connection settings explicitly defined in this dialog  
- **Running values** – Displays the connection settings currently in effect on the running SQL Server instance

------------------------- -------------------------

#### Database Settings

<img src="https://github.com/user-attachments/assets/f95ac9d0-0cd8-4a3b-ab0e-732ec3aa0ec7" width="500" title="Snipped February 27, 2025" />

- **Default index fill factor** – Determines the percentage of free space left in each index page when an index is created or rebuilt. A setting of 0 typically means 100% fill.  
- **Backup and restore** – Specifies how SQL Server handles backup media requests (e.g., “Wait indefinitely” or “Try for x hours”).  
- **Default backup media retention (in days)** – Defines how many days must pass before backup media can be overwritten.  
- **Compress backup** – Enables default backup compression to reduce backup file size (at the cost of higher CPU usage).  
- **Backup checksum** – Generates checksums during backup to help detect corruption.  
- **Recovery interval (minutes)** – The approximate time SQL Server is allowed for crash recovery; 0 indicates automatic management.  
- **Database default locations** – The default file paths where new data and log files will be stored.  
- **Configured values** – Shows the explicitly defined settings in this dialog.  
- **Running values** – Shows the settings currently in effect on the running SQL Server instance.

------------------------- -------------------------

#### Advanced

<img src="https://github.com/user-attachments/assets/fcd9ebcd-f7b0-4b45-b6f0-2000fa1a2350" width="500" title="Snipped February 27, 2025" />

- **Enable Contained Databases** – Toggles support for partially or fully contained databases that do not rely on instance-level settings  
- **FILESTREAM Access Level** – Enables or disables FILESTREAM for storing unstructured data on the file system with T-SQL access  
- **FILESTREAM Share Name** – Specifies the network share name for FILESTREAM data  

- **Allow Triggers to Fire Others** – Determines whether a trigger can fire additional triggers  
- **Boost SQL Server Priority** – Changes the Windows scheduling priority of SQL Server (not recommended in most scenarios)  
- **Cost Threshold for Parallelism** – Sets the threshold for when SQL Server considers parallel query execution  
- **Default Full-Text Language** – Specifies the default language for full-text indexing  
- **Default Language** – Sets the default language for logins without an explicitly assigned language  
- **Max Text Replication Size** – Defines the maximum size (in bytes) of text or image data that can be replicated  
- **Optimize for Ad Hoc Workloads** – Reduces plan cache overhead for single-use (ad hoc) query plans  
- **Locks** – Controls the total number of available locks (0 allows SQL Server to manage locks dynamically)  
- **Max Degree of Parallelism** – Limits the number of processors used for parallel plan execution  
- **Query Wait** – Specifies how long a query waits for memory resources before timing out  
- **Network Packet Size** – The size of network packets (in bytes) for SQL Server communication  
- **Remote Login Timeout** – The number of seconds to wait before timing out a remote login attempt  
- **Scan for Startup Procs** – Determines whether SQL Server executes any stored procedures marked for startup upon service start  
- **Two Digit Year Cutoff** – Defines the last year of the 100-year range for interpreting two-digit years  

- **Configured values** – Shows the settings explicitly defined in this dialog  
- **Running values** – Shows the settings currently in effect on the running SQL Server instance

------------------------- -------------------------

#### Permissions

<img src="https://github.com/user-attachments/assets/49b06ffc-662d-40aa-95dd-10c4d9fcb7f6" width="500" title="Snipped February 27, 2025" />

- **Logins or roles** – Displays a list of server-level logins and roles that you can select to view or modify their server-level permissions  
- **Permissions for [Selected Login or Role]** – Shows the set of server-level permissions (e.g., “Alter any database,” “View any definition,” etc.) that can be granted, denied, or granted with the ability to grant to others  
- **Grant** – Assigns the permission to the selected login or role  
- **With Grant** – Assigns the permission and also allows the selected login or role to grant it to others  
- **Deny** – Explicitly disallows the permission, overriding any existing grants  

------------------------- ------------------------- ------------------------- -------------------------

### Object Explorer

Expand and review folders in the Object Explorer.

#### Databases

<img src="https://github.com/user-attachments/assets/818b2f7e-a1a4-4e40-9282-90ad915e7889" width="800" title="Snipped February 27, 2025" />

##### System Databases

- **master** – Stores essential system information, including login accounts, system configuration settings, and metadata for all other databases.  
- **model** – Acts as a template for all newly created databases; any objects or settings here are inherited by new databases.  
- **msdb** – Used by SQL Server Agent for scheduling jobs, alerts, and operators; also stores backup and restore history.  
- **tempdb** – A temporary workspace for operations such as sorting, query work tables, temporary tables, and more. It’s recreated fresh each time SQL Server restarts.

##### Database Snapshots

Contains any read-only, static snapshots of databases. Snapshots are often used for reporting or quickly reverting to a known point in time.

##### User Databases
_not depicted, will be named during creation_

Each user database is listed by name. When you expand a specific database, you’ll see these primary subfolders:

- **Tables**  
  - Holds all user-defined tables and system tables specific to this database.  
  - Expanding this node reveals individual tables, plus folders for **System Tables** and sometimes **External Tables** (if configured).

- **Views**  
  - Contains views defined in the database. Views are virtual tables that provide a custom, filtered, or aggregated representation of data from one or more tables.

- **Synonyms**  
  - Lists any synonyms created to provide alternate names for database objects (like tables, views, stored procedures in local or remote databases).

- **Programmability**  
  - **Stored Procedures** – Contains T-SQL or CLR stored procedures.  
  - **Functions** – Houses scalar functions, table-valued functions, and system functions specific to the database.  
  - **Database Triggers** – Displays DDL triggers at the database level (not the same as table triggers).  
  - **Assemblies** (if any) – Shows CLR assemblies registered in the database.

- **Service Broker** (if enabled)  
  - Provides messaging and queuing features. You’ll see queues, message types, contracts, and more under this node.

- **Storage**  
  - **Full Text Catalogs** – Lists any catalogs for full-text search indexing.  
  - **Partition Schemes** and **Partition Functions** – Used for partitioning large tables and indexes across multiple filegroups.

- **Security**  
  - **Users** – Lists database-level users mapped from logins or containing user accounts (for contained databases).  
  - **Roles** – Contains both fixed database roles (e.g., db_datareader, db_owner) and user-defined database roles.  
  - **Schemas** – Defines logical containers for objects; used to group and manage objects under different namespaces.  
  - **Asymmetric Keys**, **Certificates**, **Database Audit Specifications** (if configured) – For advanced security and encryption features.

- **External Resources** (if applicable)  
  - **External Tables**, **External Data Sources**, **External File Formats** – Appear if you’re using PolyBase or external data connections.

- **Graph** (if enabled)  
  - If you have SQL Graph features in use, you’ll see **Graph Tables** or **Node/Edge Tables** here.

- **Database Snapshots** (per-database)  
  - If you’ve created a snapshot for a specific user database, it may appear here as well.

------------------------- -------------------------

##### Exercise: Create Database

In the Object Explorer, right-click "Databases" and select "New Database" from the resulting menu.

<img src="https://github.com/user-attachments/assets/526e6c2a-38d5-4ccb-a8d9-9f0104f7e1cd" width="500" title="Snipped February 27, 2025" />

Complete the "New Database" form and click "OK".

<img src="https://github.com/user-attachments/assets/526e6c2a-38d5-4ccb-a8d9-9f0104f7e1cd" width="500" title="Snipped February 27, 2025" />

Expand the new database in the Object Explorer and review.

------------------------- ------------------------- -------------------------

#### Security

<img src="https://github.com/user-attachments/assets/f00e8fac-8d07-4ed2-80c7-023108e8d20b" width="800" title="Snipped February 27, 2025" />

- **Logins**  
  - Contains all server-level logins, including Windows logins/groups and SQL Server logins.  
  - Each login can be mapped to one or more database users.

- **Server Roles**  
  - Lists fixed server roles (e.g., `sysadmin`, `serveradmin`, `securityadmin`) as well as any user-defined server roles.  
  - Roles group logins together for simplified permission management at the server level.

- **Credentials**  
  - Holds credentials used to access external resources under a specific security context.  
  - Often used for tasks like running SQL Server Agent jobs or accessing external files.

- **Audits**  
  - Configures audit objects that capture server- or database-level events.  
  - Audits define where and how event logs are stored (e.g., file, security log, application log).

- **Server Audit Specifications**  
  - Specifies which events (e.g., login changes, schema modifications) are collected for a given audit.  
  - Ties directly to the **Audits** object for storing captured data.

- **Always Encrypted Keys**  
  - **Column Master Keys** – Protect the keys used to encrypt sensitive columns.  
  - **Column Encryption Keys** – The actual keys that encrypt data in Always Encrypted columns.

- **Asymmetric Keys**  
  - Public/private key pairs used for encryption or signing.  

- **Certificates**  
  - Digital certificates for encryption or authentication (e.g., SSL certificates, signing certificates).  

- **Symmetric Keys**  
  - Single-key encryption objects used for data encryption tasks within SQL Server.  

------------------------- -------------------------

##### Exercise: New User

Create a new user in Windows.

<img src="https://github.com/user-attachments/assets/4fa6a554-2db0-4510-9d97-0f989ac52452" width="300" title="Snipped February 27, 2025" />

In the SQL Server Management Studio Object Explorer, expand the "Security" folder, right-click "Logins" and select "New Login" from the resulting menu.

<img src="https://github.com/user-attachments/assets/6bf615a2-078e-4ab2-9fd4-a51c960b3c94" width="500" title="Snipped February 27, 2025" />

Complete the "Login - New" form and click "OK".

Navigate to "Databases" >> "trainingdb" >> "Security", right-click "Users" and select "New User" from the resulting menu.

<img src="https://github.com/user-attachments/assets/77de7e87-906c-4f18-8bb9-28c18822a106" width="500" title="Snipped February 27, 2025" />

Complete the "Database User - New" form.

Navigate to "Membership" and check "db_datareader".

<img src="https://github.com/user-attachments/assets/a2f718c8-dd0e-4eba-b110-4d3b39ec8cb8" width="500" title="Snipped February 27, 2025" />

Click "OK".

------------------------- ------------------------- -------------------------

#### Server Objects

<img src="https://github.com/user-attachments/assets/83326ac6-9f04-45d5-bc4d-a309ba94eb2e" width="800" title="Snipped February 27, 2025" />

- **Backup Devices**  
  - Manually defined references to backup file locations (e.g., disk or tape). These let you manage backups more consistently by naming the device instead of specifying paths each time.

- **Endpoints**  
  - Configured network endpoints that enable specific types of communication (e.g., T-SQL, Service Broker, Database Mirroring). They define how SQL Server listens for or sends out data.

- **Linked Servers**  
  - References to external data sources (another SQL Server instance, Oracle, etc.). You can query remote databases as if they were local, using four-part naming conventions.

- **Triggers**  
  - Server-level DDL triggers that fire in response to events such as CREATE_LOGIN, DROP_DATABASE, or ALTER_SERVER_ROLE. Different from database triggers, these affect the entire server.

- **Server Audit Specifications**  
  - If server-level auditing is configured, this node lists which events or actions (e.g., login changes, role modifications) are captured and logged as part of the audit.

------------------------- ------------------------- -------------------------

#### Replication

<img src="https://github.com/user-attachments/assets/8a4269a5-caf0-4321-8004-d3dc5eeb67df" width="800" title="Snipped February 27, 2025" />

- **Local Publications**  
  - Displays any publications defined on this server. A publication is a collection of database objects and data to be replicated to one or more subscribers.  
  - You can right-click **Local Publications** to create a new publication (e.g., Snapshot, Transactional, or Merge replication).

- **Local Subscriptions**  
  - Shows any subscriptions that this server is receiving from another publisher.  
  - A subscription tells the subscriber how and when to receive published data.

- **Other Replication Objects** (as configured)  
  - Depending on your setup, you may see additional nodes or references to distribution databases, remote publishers, or push/pull subscriptions.

**Key Replication Concepts:**

- **Publisher** – The source server and database that make data available for replication.  
- **Subscriber** – The destination server and database that receive replicated data from the publisher.  
- **Distributor** – A server or database that manages the flow of data and metadata between publishers and subscribers.

------------------------- ------------------------- -------------------------

#### Always On High Availability

_Note: Must be enabled via SQL Server Configuration Manager_

<img src="https://github.com/user-attachments/assets/bc4c7115-6706-4bd1-b6ba-27da0a00189c" width="800" title="Snipped February 28, 2025" />

- **Availability Groups**  
  - Lists the high availability groups configured on this SQL Server instance.  
  - Each availability group is a logical container that manages one or more user databases, along with one or more replicas (primary and secondary).

  - **Availability Group <GroupName>**  
    - Represents an individual availability group. Under each group, you’ll typically see the following subfolders:

    - **Availability Replicas**  
      - Displays all replicas participating in the availability group: one primary replica (which handles read/write) and one or more secondary replicas (which can be read-only, backup targets, or standby for failover).  
      - Each replica node shows its role (primary/secondary), connection mode (synchronous/asynchronous), and operational state.

    - **Availability Databases**  
      - Shows the user databases that are part of the availability group.  
      - Each database node indicates whether it’s synchronized, synchronizing, or not synchronized, and whether it’s in a healthy state for failover.

    - **Availability Group Listeners** (if configured)  
      - Displays any configured listener for the group. The listener is a virtual network name and IP address that client applications can use to connect without needing to know which replica is currently primary.

**Key Always On Concepts:**

- **Primary Replica** – Hosts the read/write copy of the databases in the group.  
- **Secondary Replicas** – Maintain copies of the databases and can be used for read-only queries or backups, depending on the configuration.  
- **Failover** – The process by which the secondary replica can become the primary if the current primary becomes unavailable (automatic or manual, based on configuration).  
- **Synchronization Mode** – Either **synchronous** (no data loss, typically for local high availability) or **asynchronous** (allows potential data loss, often used for remote disaster recovery).

------------------------- ------------------------- -------------------------

#### Management

<img src="https://github.com/user-attachments/assets/538bdebf-4324-43fe-9b33-ecd9d7ab2f00" width="800" title="Snipped February 28, 2025" />

Below is a bullet-point list of the **Management** folder items in the **exact** order they appear in your shared image, with brief explanations for each:

- **Policy Management**  
  Lets you create and enforce policies to ensure SQL Server objects (e.g., databases, logins, configurations) comply with defined standards or best practices.

- **Data Collection**  
  Gathers performance and usage metrics for later analysis, often used in conjunction with a Management Data Warehouse (MDW).

- **Resource Governor**  
  Enables you to manage CPU and memory usage by defining resource pools and workload groups, ensuring critical workloads have the resources they need.

- **Extended Events**  
  A flexible, lightweight framework for capturing and analyzing SQL Server events. You can create sessions to collect detailed data for performance monitoring or troubleshooting.

- **Maintenance Plans**  
  Offers a wizard-driven interface to automate routine tasks such as backups, index maintenance, and database integrity checks.
  
- **SQL Server Logs**  
  Provides quick access to the SQL Server error log and related Windows Event Log entries, helping you diagnose and troubleshoot issues.

- **Database Mail**  
  Allows SQL Server to send email alerts or notifications (e.g., job status updates). You can configure SMTP settings, profiles, and accounts here.

- **Distributed Transaction Coordinator**  
  Coordinates transactions that span multiple resource managers (e.g., multiple databases or external systems), ensuring consistency and atomicity across all involved resources.

------------------------- ------------------------- -------------------------

#### Integration Services Catalogs

<img src="https://github.com/user-attachments/assets/e0438d4c-1618-45fd-8c2d-871ba7d3d25b" width="800" title="Snipped February 28, 2025" />

- **SSISDB Catalog**  
  - A system database (SSISDB) that hosts deployed SSIS projects and packages.  
  - Introduced with the Project Deployment Model (SQL Server 2012 and later) to streamline deployment and management of SSIS solutions.

- **Projects**  
  - Logical containers within the SSIS catalog that hold one or more SSIS packages.  
  - Each project typically corresponds to a Visual Studio solution or an SSIS project file you deploy to the server.

- **Packages**  
  - Individual SSIS workflows, each containing control flow and data flow tasks.  
  - Stored inside the SSISDB catalog under their respective project.

- **Environments**  
  - A feature used to manage environment-specific parameters or connection strings (e.g., Dev, Test, Production).  
  - Lets you dynamically change package configurations without altering package code.

- **Configurations and Parameters**  
  - Each SSIS project or package can have parameters that control runtime behavior.  
  - You can bind these parameters to environment variables, simplifying deployments across multiple servers.

- **Execution and Monitoring**  
  - SSIS packages can be executed manually from the catalog or scheduled via SQL Server Agent jobs.  
  - Logging, error messages, and performance statistics are stored in SSISDB for monitoring and troubleshooting.

- **Security**  
  - You can manage permissions at the project or package level, restricting who can deploy or execute SSIS packages.  
  - SSISDB also supports encryption at rest and secure storage of sensitive data (e.g., connection passwords).

------------------------- ------------------------- -------------------------

#### SQL Server Agent

<img src="https://github.com/user-attachments/assets/bac9a298-7017-4063-875a-afe484652b6b" width="800" title="Snipped February 28, 2025" />

- **Jobs**  
  - Lists all scheduled tasks and scripts that SQL Server Agent will run at specified times or in response to events.  
  - You can right-click **Jobs** to create, edit, or manage new jobs (e.g., database backups, maintenance tasks, custom T-SQL scripts).

- **Job Activity Monitor**  
  Provides a real-time overview of SQL Server Agent job statuses (running, succeeded, failed, etc.), along with job history for quick troubleshooting and scheduling insights.
  
- **Alerts**  
  - Enables you to configure automated responses to specific SQL Server events or performance conditions (e.g., a low disk space event).  
  - Alerts can trigger emails, pages, or job executions.

- **Operators**  
  - Defines contact information (email addresses, pagers) for individuals or groups who should be notified about alerts or job statuses.  
  - You can set up different operators for various teams (e.g., DBAs, network admins).

- **Proxies**  
  - Allows you to specify alternate security credentials for running specific job steps (e.g., SSIS packages, OS-level commands) without granting the entire SQL Server Agent service elevated permissions.

- **Error Logs**  
  - Displays messages recorded by SQL Server Agent (separate from the main SQL Server error logs).  
  - Useful for diagnosing issues specifically related to Agent jobs, alerts, or other scheduled tasks.

------------------------- ------------------------- -------------------------

#### XEvent Profiler

<img src="https://github.com/user-attachments/assets/cd128892-a85d-4d6b-81db-188e3ac73e75" width="800" title="Snipped February 28, 2025" />

- **XEvent Profiler**  
  Provides a simplified interface for real-time monitoring of SQL Server activity using Extended Events. Under **XEvent Profiler**, you’ll typically see two default sessions:

  - **Standard**  
    Captures a broad range of SQL Server events, including batch starts/completions, errors, and more—useful for general performance monitoring or troubleshooting.

  - **TSQL**  
    Focuses on T-SQL statements, batches, and stored procedure calls—ideal for diagnosing query-related issues and understanding SQL execution patterns.

------------------------- ------------------------- ------------------------- -------------------------

### Quiz: SQL Server (on-prem)

1. **Which T-SQL command creates a new database named SalesDB on an on-premises SQL Server?**  
   A. `MAKE DATABASE SalesDB;`  
   B. `NEW DATABASE SalesDB;`  
   C. `CREATE DATABASE SalesDB;`  
   D. `ADD DATABASE SalesDB;`

2. **You want to ensure your SQL Server instance does not exceed 8 GB of RAM usage. Which setting should you modify under Server Properties → Memory?**  
   A. Minimum server memory (in MB)  
   B. Maximum server memory (in MB)  
   C. Index creation memory (in KB)  
   D. Minimum memory per query (in KB)

3. **A newly created login can connect to the server, but cannot access a user database. Which of the following is the most likely cause?**  
   A. The user’s password is expired.  
   B. The login has not been mapped to a database user.  
   C. The login is using Windows Authentication.  
   D. The server is in single-user mode.

4. **Which T-SQL command is recommended in modern SQL Server versions for adding a database user to the db_datareader role?**  
   A. `sp_addrolemember 'db_datareader', 'UserName';`  
   B. `ALTER ROLE db_datareader ADD MEMBER UserName;`  
   C. `EXEC sp_add_user_to_role 'UserName', 'db_datareader';`  
   D. `EXEC sp_roleadd 'db_datareader', 'UserName';`

5. **You wish to reduce page splits when rebuilding an index by leaving extra free space on each page. Which setting should you adjust?**  
   A. Default backup media retention (in days)  
   B. Default index fill factor  
   C. Max Degree of Parallelism (MAXDOP)  
   D. Optimize for Ad Hoc Workloads

6. **In the Server Properties → Security page, switching from “Windows Authentication mode” to “SQL Server and Windows Authentication mode” does what?**  
   A. Allows Windows logins only  
   B. Disables all existing logins  
   C. Enables mixed authentication, allowing both SQL and Windows logins  
   D. Forces the server to use the ‘sa’ account exclusively

7. **You want to limit CPU usage by certain queries on a busy SQL Server instance. Which feature should you configure under the Management folder?**  
   A. Data Collection  
   B. Policy Management  
   C. Resource Governor  
   D. Maintenance Plans

8. **Which of the following best describes the ‘Collation’ setting in Server Properties → General?**  
   A. Controls how data is encrypted at rest  
   B. Defines how text is sorted and compared  
   C. Determines the maximum database size  
   D. Specifies the default network packet size

9. **You need to schedule a weekly backup job that runs every Sunday at 2:00 AM. Which SQL Server component do you use to set this up?**  
   A. SQL Server Agent Jobs  
   B. Replication (Local Publications)  
   C. Database Snapshots  
   D. Policy Management

10. **Which statement about server-level roles in SQL Server is true?**  
   A. They can be applied to individual tables only.  
   B. They are defined within each user database.  
   C. They group logins for managing permissions at the server scope.  
   D. They are deprecated in the latest SQL Server versions.

------------------------- -------------------------

### Answer Key

1. **Answer: C**  
   Explanation: The correct T-SQL command to create a new database is `CREATE DATABASE SalesDB;`.

2. **Answer: B**  
   Explanation: “Maximum server memory (in MB)” limits how much RAM SQL Server can consume. Setting this to 8192 MB (8 GB) prevents SQL Server from exceeding that usage.

3. **Answer: B**  
   Explanation: A login must be mapped to a corresponding user in each database for which it needs access.

4. **Answer: B**  
   Explanation: `ALTER ROLE db_datareader ADD MEMBER UserName;` is the modern command. `sp_addrolemember` still works but is considered legacy.

5. **Answer: B**  
   Explanation: The **Default index fill factor** determines how much free space to leave on each page when creating or rebuilding an index.

6. **Answer: C**  
   Explanation: Changing from Windows Authentication mode to SQL Server and Windows Authentication mode (mixed mode) enables both SQL logins and Windows logins.

7. **Answer: C**  
   Explanation: **Resource Governor** allows you to define resource pools and workload groups to control CPU and memory usage for specific workloads.

8. **Answer: B**  
   Explanation: Collation determines how text data is sorted and compared, including case sensitivity and accent sensitivity.

9. **Answer: A**  
   Explanation: SQL Server Agent Jobs allow you to schedule recurring tasks such as backups, index maintenance, or custom T-SQL scripts.

10. **Answer: C**  
   Explanation: Server-level roles (e.g., `sysadmin`, `serveradmin`) manage permissions at the server scope, not the database or table scope.

------------------------- ------------------------- ------------------------- -------------------------

## Azure SQL

Navigate to the Azure Marketplace and create a "Single database" instance of SQL.

<img src="https://github.com/user-attachments/assets/84e8e6d7-d0ec-49db-ac5b-c2aee66698b5" width="800" title="Snipped February 28, 2025" />

- **SQL Database**  
  - A fully managed PaaS offering for modern cloud applications.  
  - Ideal for single-database workloads, hyperscale scenarios, and minimal management overhead.  
  - Provides built-in high availability, automatic backups, and simplified scaling.  

- **SQL Managed Instance**  
  - A near 100% compatible instance-level PaaS solution.  
  - Retains many on-prem features (e.g., cross-database queries, SQL Agent) with lower management demands.  
  - Great for lift-and-shift migrations needing instance-level functionality and broader T-SQL compatibility.  

- **SQL Virtual Machines**  
  - An IaaS solution where you run SQL Server on an Azure VM.  
  - Grants full OS-level and SQL Server control, similar to on-prem deployments.  
  - Ideal for complex customizations or legacy applications requiring direct access to the operating system.

### Basics

<img src="https://github.com/user-attachments/assets/fcca33f6-3804-41a0-b066-e5d60151e132" width="800" title="Snipped February 28, 2025" />

#### Project Details

- **Subscription**  
  Select the Azure subscription under which this SQL Database will be billed. If you have multiple subscriptions, ensure you pick the one appropriate for your project or organization.

- **Resource Group**  
  Choose an existing resource group or create a new one. Resource groups help you logically organize and manage Azure resources (e.g., databases, servers, VMs) that share a common lifecycle.

#### Database Details

- **Database Name**  
  Enter a unique name for your Azure SQL Database. This is how the database will appear in the Azure Portal and in client connections.

- **Server**  
  Choose an existing logical Azure SQL server or create a new one. A logical server acts as a central point for managing databases, firewall rules, and logins.

##### Server >> Create New

<img src="https://github.com/user-attachments/assets/c29a410f-fd63-4656-a634-398187571bb0" width="800" title="Snipped February 28, 2025" />

- **Server name** – A globally unique name for your new Azure SQL logical server (e.g., `myserver.database.windows.net`).  
- **Location** – The Azure region where the server (and its databases) will physically reside.  
- **Authentication method** – Choose how users will authenticate:  
  - **Use only Microsoft Entra ID only authentication** – Restricts authentication to Microsoft Entra (Azure AD) identities.  
  - **Use both SQL and Microsoft Entra ID authentication** – Allows logins using either SQL credentials or Microsoft Entra identities.  
  - **Use only SQL authentication** – Permits only SQL logins and passwords for server access.  
- **Set Microsoft Entra Admin** – (Optional) Assign a Microsoft Entra user or group as the server-level administrator, enabling them to manage the server and databases without requiring SQL authentication.

##### Compute + Storage

<img src="https://github.com/user-attachments/assets/26835cb8-8601-440d-bfcc-8a1b6e906421" width="800" title="Snipped February 28, 2025" />

- **Service and compute tier**  
  - Select a purchasing model (**vCore** or **DTU**) and a corresponding service tier (e.g., **General Purpose**, **Business Critical**, **Hyperscale**).  
  - This choice determines overall performance, storage limits, and high availability or redundancy features.

- **Compute tier**  
  - **Provisioned**: Allocates the chosen compute resources (CPU/memory) continuously for predictable performance and costs.  
  - **Serverless**: Dynamically scales compute based on workload demand and can pause during inactivity to reduce costs.

- **Hardware configuration**  
  - Choose the hardware generation (e.g., **Standard Series (Gen5)**) based on your workload’s requirements for availability, memory optimization, and performance.  

- **Max Cores / Min Cores**  
  - **Max Cores**: The upper limit of CPU cores your database can use.  
  - **Min Cores**: The minimum CPU cores that remain allocated (relevant in serverless mode to ensure a baseline level of performance).

- **Auto-pause delay**  
  - The amount of idle time (in minutes) after which a serverless database automatically pauses, minimizing compute costs.

- **Data max size (GB)**  
  - The maximum storage allocated to the database. Operations fail if this limit is reached and cannot be automatically expanded.

------------------------- -------------------------

### Networking

<img src="https://github.com/user-attachments/assets/278c9a86-8bdf-4b4c-bb7f-f672a47300b2" width="800" title="Snipped February 28, 2025" />

- **Connectivity method**  
  - **Public endpoint:** Allows connections over the public internet, managed via firewall rules.  
  - **Private endpoint:** Restricts traffic to a private IP address in your virtual network, avoiding public exposure.

- **Firewall rules**  
  - **Allow Azure services and resources to access this server:** Toggles whether Azure resources (e.g., Azure App Service) can connect without specifying IP addresses explicitly.  
  - **Add client IP:** Automatically creates a firewall rule for your current public IP address, ensuring immediate access from your local machine.  
  - **Yes/No toggle:** Determines whether to apply these firewall settings. “Yes” applies the above rules; “No” leaves them disabled or requires manual firewall configuration.

- **Connection policy**  
  - **Default (recommended):** Routes connections through the best path automatically (proxy or redirect) based on client location and performance considerations.  
  - **Proxy:** All client connections pass through the Azure SQL Database gateway, which may add latency but can simplify firewall configurations.  
  - **Redirect:** Clients connect directly to the database after an initial handshake with the gateway, reducing latency by bypassing the proxy for subsequent traffic.

- **Encrypted connections**  
  - **Transport Layer Security (TLS):** Ensures data in transit is protected against interception.  
  - **Enforce specific TLS version:** May be required for compliance or best-practice security.

------------------------- -------------------------

### Security

<img src="https://github.com/user-attachments/assets/a6d3650f-c7ae-48ce-b928-246b6c249796" width="800" title="Snipped February 28, 2025" />

- **Microsoft Defender for SQL**  
  - Provides advanced threat protection and vulnerability assessments for your database.  
  - Can be enabled on a paid basis or via a free trial (if available).

- **Ledger**  
  - Enables tamper-evident storage, ensuring transactions are cryptographically verifiable and cannot be modified without detection.

- **Server Identity**  
  - Uses Azure-managed identities (system-assigned or user-assigned) to securely access other Azure resources without storing credentials.

- **Transparent Data Encryption (TDE)**  
  - Encrypts data at rest by default using an Azure-managed key.  
  - Optionally configure a customer-managed key stored in Azure Key Vault for greater control.

------------------------- -------------------------

### Additional Settings

  <img src="https://github.com/user-attachments/assets/4c1ad060-051b-4267-93fa-517ab987c358" width="800" title="Snipped February 28, 2025" />

- **Use existing data**  
  - **None**: Creates an empty database with no pre-loaded schema or data.  
  - **Backup**: Restores from an existing backup within the same subscription.  
  - **Sample**: Deploys a Microsoft-provided sample database (e.g., AdventureWorks) to explore features or quickly test queries.

- **Collation**  
  - Determines how text data is sorted and compared.  
  - You can accept the default collation (e.g., `SQL_Latin1_General_CP1_CI_AS`) or select a custom collation that matches your language and sorting requirements.

- **Maintenance window**  
  - **System default**: Azure automatically schedules patching and updates.  
  - **Custom**: Specify a preferred time range to reduce potential downtime during critical business hours.

------------------------- -------------------------

### SQL Server Management Studio >> Azure SQL

In this section we'll focus on what is different from SSMS >> SQL On-Prem.











