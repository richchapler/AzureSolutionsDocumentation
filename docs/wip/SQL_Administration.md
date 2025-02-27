# SQL: Fundamentals  

## Resources

* Azure Virtual Machine
  * Image: "Free SQL Server License: SQL Server 2022 Developer on Windows Server 2022"
  * SKU: "Standard_D2s_v3..."
  * Inbound Ports: RDP (3389) allowed
  * Boot Diagnostics: Disable
* Azure SQL

Overview:  
This lab offers hands-on experience with core administrative tasks—covering server setup, maintenance, troubleshooting, database creation, and user/permission management. It is structured to support both on-premises SQL Server and Azure SQL environments, with exam-style questions to reinforce key concepts. Enhancements include updated guidance on modern T-SQL commands, detailed troubleshooting steps, and best practices for both environments.

------------------------- -------------------------

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

Launch SQL Server Management Studio and connect to your SQL Server instance.  

<img src="https://github.com/user-attachments/assets/f1238467-a0f9-4569-936b-79f19b1351ff" width="500" title="Snipped February 27, 2025" />

Click "New Query" and execute the following T-SQL to verify connectivity and confirm the server version:  

```sql
SELECT @@VERSION;
```

<img src="https://github.com/user-attachments/assets/f0fbb736-ba1b-47dc-83d5-46e54e1a83cd" width="500" title="Snipped February 27, 2025" />

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

<img src="https://github.com/user-attachments/assets/377b9aa6-0326-4e28-b563-1ff4c27bf713" width="500" title="Snipped February 27, 2025" />

### Object Explorer

Expand and review folders in the Object Explorer.

#### Databases

<img src="https://github.com/user-attachments/assets/818b2f7e-a1a4-4e40-9282-90ad915e7889" width="500" title="Snipped February 27, 2025" />

##### System Databases

- **master** – Stores essential system information, including login accounts, system configuration settings, and metadata for all other databases.  
- **model** – Acts as a template for all newly created databases; any objects or settings here are inherited by new databases.  
- **msdb** – Used by SQL Server Agent for scheduling jobs, alerts, and operators; also stores backup and restore history.  
- **tempdb** – A temporary workspace for operations such as sorting, query work tables, temporary tables, and more. It’s recreated fresh each time SQL Server restarts.

##### Database Snapshots

Contains any read-only, static snapshots of databases. Snapshots are often used for reporting or quickly reverting to a known point in time.

- **User Databases**  
- Each user database is listed by name (e.g., **cnbtraining**, **cnbtrainingChap1**, etc.). When you expand a specific database, you’ll see these primary subfolders:

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

###### ...using the app interface

1. **Open SSMS and Connect**  
   - Launch SQL Server Management Studio (SSMS).  
   - Connect to the SQL Server instance where you want to create the database.

2. **Create a New Database**  
   - In the **Object Explorer**, right-click **Databases** and select **New Database…**.  
   - In the **New Database** dialog, enter **TrainingDB** as the database name.  
   - Click **OK** to create the database.

3. **Verify Creation**  
   - Expand **Databases** in Object Explorer to confirm that **TrainingDB** now appears in the list.

---

###### ...using the T-SQL

1. **Open a New Query Window**  
   - In SSMS, click **New Query** on the toolbar.

2. **Run the CREATE DATABASE Command**  
   ```sql
   CREATE DATABASE TrainingDB;
   GO
   ```
   - Press **F5** or click **Execute** to run the command.

3. **Verify Creation**  
   - In Object Explorer, right-click **Databases** and select **Refresh**.  
   - Confirm that **TrainingDB** appears in the list of databases.

















- **Security** – Contains server-level security objects such as logins, server roles, credentials, and certificates. This is where you manage authentication and authorization for your SQL Server instance.
- **Server Objects** – Includes items like backup devices, linked servers, endpoints, and triggers. It’s useful for configuring integrations and managing various server-level features.
- **Replication** – Displays replication settings if configured, including publishers, distributors, and subscribers. This node is used to manage replication tasks.
- **Management** – Provides access to tools and utilities like the SQL Server Agent, Activity Monitor, maintenance plans, and policy management. It's essential for monitoring server health and automating routine tasks.

This Object Explorer structure helps you navigate and manage different aspects of your SQL Server environment efficiently. Let me know if you'd like to dive deeper into any specific section!







### 3. Database Creation (On-Premises)

Using T-SQL:  

- Create a new database named `TrainingDB`:

  ```sql
  CREATE DATABASE TrainingDB;
  GO
  ```

- Refresh Object Explorer to confirm the new database.

Using SSMS GUI (Optional):  

- Right-click Databases, select New Database…, and follow the prompts.

Enhanced Best Practice:  

- Consider reviewing the file layout (data and log files) for performance tuning and recovery objectives.

Exam Guidance:  

- Know both methods; exam questions may ask which T-SQL command creates a new database.

### 4. Creating and Managing Users (On-Premises)

Creating a SQL Server Login:  

- Execute the following to create a login:

  ```sql
  CREATE LOGIN TestUser WITH PASSWORD = 'YourPassword123';
  GO
  ```

Mapping Login to a Database User:  

- Switch to the `TrainingDB` database and create a user:

  ```sql
  USE TrainingDB;
  GO
  CREATE USER TestUser FOR LOGIN TestUser;
  GO
  ```

- Assigning a Role:  

  - Legacy Method (still common in exam scenarios):

    ```sql
    EXEC sp_addrolemember 'db_datareader', 'TestUser';
    GO
    ```

  - Modern Alternative:  

    ```sql
    ALTER ROLE db_datareader ADD MEMBER TestUser;
    GO
    ```

  - Enhanced Note: Although many exams reference `sp_addrolemember`, note that Microsoft recommends the `ALTER ROLE ... ADD MEMBER` syntax in newer versions of SQL Server.

Verifying Permissions:  

- Optionally, log in as TestUser and run a query (e.g., `SELECT TOP 10 * FROM SomeTable;`) to validate access.

Exam Guidance:  

- Understand the distinction between logins and users, and be prepared to map them correctly in exam questions.

### 5. Basic Troubleshooting (On-Premises)

Simulated Scenario:  

- If TestUser is unable to query data from TrainingDB, review the error message and follow these steps:

Troubleshooting Checklist:  

- Confirm that TestUser exists in TrainingDB.  
- Verify that TestUser is assigned to the appropriate role (e.g., `db_datareader`).  
- Check SQL Server error logs and Windows Event Logs for any related diagnostic information.  
- Enhanced Tip: Use the SQL Server Activity Monitor to check for blocking or performance issues.

Exam Guidance:  

- Exam questions may involve troubleshooting permission issues, so be comfortable with these steps.

---

## Part 2: Azure SQL Lab

### 1. Environment Setup (Azure SQL)

Pre-requisites:  

- Have an active Azure subscription and access to the [Azure Portal](https://portal.azure.com/).  
- Ensure you have the latest version of SSMS or Azure Data Studio installed.

Initial Connection:  

- Connect via SSMS using your Azure SQL server details.  

- Execute:

  ```sql
  SELECT @@VERSION;
  ```

  - Note: The output will indicate that this is an Azure SQL Database instance.

Enhanced Considerations:  

- Verify that your client IP is allowed through the Azure SQL firewall settings.

Exam Guidance:  

- Recognize the connectivity differences and how version information is presented in Azure SQL.

### 2. Exploring Azure SQL Configuration

Reviewing Database Settings:  

- In the Azure portal, navigate to your Azure SQL Database and review:
  - Performance tier and DTU/vCore settings.
  - Firewall rules and virtual network configurations.
  - Auditing and threat detection settings.

Using SSMS:  

- You can also execute T-SQL commands to review configurations and monitor performance metrics.

Enhanced Note:  

- Leverage the built-in diagnostic tools and performance insights available in the Azure portal for deeper analysis.

Exam Guidance:  

- Understanding both the Azure portal and SSMS methods is key for troubleshooting and exam scenarios.

### 3. Database Creation (Azure SQL)

Using T-SQL in SSMS:  

- Create a new database named `AzureTrainingDB`:

  ```sql
  CREATE DATABASE AzureTrainingDB;
  GO
  ```

Using the Azure Portal (Optional):  

- Select Create a resource, choose SQL Database, and follow the guided steps.

Enhanced Best Practice:  

- Configure backup and geo-replication settings based on your data protection requirements.

Exam Guidance:  

- Be familiar with both creation methods; exam questions may include Azure-specific features.

### 4. Creating and Managing Users (Azure SQL)

Creating Contained Database Users:  

- SQL-Authenticated User:

  ```sql
  CREATE USER AzureUser WITH PASSWORD = 'YourAzurePassword123';
  GO
  ```

- Azure AD User:  
  (For environments integrated with Azure Active Directory)

  ```sql
  CREATE USER [user@domain.com] FROM EXTERNAL PROVIDER;
  GO
  ```

Assigning Roles:  

- Add the user to a role using:

  ```sql
  EXEC sp_addrolemember 'db_datareader', 'AzureUser';
  GO
  ```

- Modern Alternative (if supported):

  ```sql
  ALTER ROLE db_datareader ADD MEMBER AzureUser;
  GO
  ```

Enhanced Note:  

- Azure SQL often uses contained database users, so ensure you understand the differences from on-premises user creation.

Exam Guidance:  

- Examine the differences in authentication methods and role assignment for Azure SQL.

### 5. Basic Troubleshooting (Azure SQL)

Simulated Scenario:  

- If an Azure SQL user cannot access data, perform these checks:

Troubleshooting Checklist:  

- Verify the user’s existence and role membership in the database.  
- Confirm that the client IP address is allowed by the Azure SQL firewall rules.  
- Use the Azure portal’s diagnostic tools to monitor and identify issues (e.g., Query Performance Insight).  
- Review auditing logs for additional context on failed connections.

Exam Guidance:  

- Azure-specific issues (like firewall settings) are common exam topics, so be sure to cover these aspects.

------------------------- ------------------------- ------------------------- -------------------------

## Quiz: SQL Server (on-prem)

### On-Premises SQL Server Exam-Style Questions

Question 1:  
You have just created a new SQL Server login named TestUser. Which of the following commands correctly maps this login to a database user in TrainingDB and assigns the user to the db_datareader role?

A.  

```sql
USE TrainingDB;  
GO  
CREATE USER TestUser FOR LOGIN TestUser;  
GO  
EXEC sp_addrolemember 'db_datareader', 'TestUser';  
GO
```

B.  

```sql
CREATE USER TestUser FOR LOGIN TestUser;  
ALTER ROLE db_datareader ADD MEMBER TestUser;  
GO
```

C.  

```sql
USE TrainingDB;  
GO  
CREATE USER TestUser FOR LOGIN TestUser;  
GO  
GRANT SELECT TO TestUser;  
GO
```

D. Both A and C are correct.

Answer: A  
Enhanced Explanation: Option A explicitly creates the database user and assigns it to the `db_datareader` role using the legacy (but exam-recognized) stored procedure. Although Option B uses the modern `ALTER ROLE ... ADD MEMBER` syntax—which is valid on newer versions—exams often expect Option A.

---

Question 2:  
Which T-SQL command creates a new database named TrainingDB on an on-premises SQL Server?

A. `CREATE DATABASE TrainingDB;`  
B. `NEW DATABASE TrainingDB;`  
C. `MAKE DATABASE TrainingDB;`  
D. `ADD DATABASE TrainingDB;`

Answer: A  
Explanation: The correct T-SQL command to create a new database is `CREATE DATABASE TrainingDB;`.

---

Question 3:  
A user is unable to query data from TrainingDB on your on-premises server. What is the first step you should take to troubleshoot this issue?

A. Verify that the SQL Server instance is running.  
B. Check the SQL Server error logs.  
C. Verify that the user is correctly mapped to TrainingDB and has the appropriate role.  
D. Restart the SQL Server service.

Answer: C  
Explanation: The most direct troubleshooting step is to ensure the user is properly mapped to the database and has the necessary permissions.

---

### Azure SQL Exam-Style Questions

Question 4:  
You have created a contained database user in an Azure SQL Database named AzureTrainingDB. Which of the following commands creates a contained user with SQL authentication?

A.  

```sql
CREATE USER AzureUser WITH PASSWORD = 'YourAzurePassword123';
GO
```

B.  

```sql
CREATE LOGIN AzureUser WITH PASSWORD = 'YourAzurePassword123';
GO
```

C.  

```sql
CREATE USER [user@domain.com] FROM EXTERNAL PROVIDER;
GO
```

D. Both A and C are correct.

Answer: A  
Explanation: In Azure SQL, a contained database user is created directly in the database using `CREATE USER` with a password. Option C is used for Azure AD authentication.

---

Question 5:  
Which method is commonly used to connect to an Azure SQL Database for administrative tasks?

A. SQL Server Management Studio (SSMS)  
B. Azure Data Studio  
C. Azure Portal Query Editor  
D. All of the above

Answer: D  
Explanation: All these methods can be used to connect to and manage an Azure SQL Database.

---

Question 6:  
When troubleshooting access issues in an Azure SQL Database, what is one additional configuration to verify that is not typically an on-premises concern?

A. Database user mapping  
B. Role membership  
C. Firewall rules in the Azure portal  
D. SQL Server error logs

Answer: C  
Explanation: In Azure SQL, firewall rules set in the Azure portal can block access, making them an important troubleshooting aspect that is unique to cloud environments.

---

Reference:  
• [SQL Server Educational Resources](https://learn.microsoft.com/en-us/sql/sql-server/educational-sql-resources)
