# PowerShell Fundamentals for SQL Server (On-Premises and Azure)

## Course Overview 
PowerShell is a powerful scripting tool for automating and managing SQL Server environments both on-premises and in the cloud. This course provides a structured introduction to using PowerShell (in both Windows PowerShell 5.1 and cross-platform PowerShell Core 7+) for SQL Server administration. You will learn how to perform common tasks such as SQL Server setup, user and permission management, automating maintenance, and connecting to Azure SQL resources using dedicated PowerShell cmdlets and scripts. Throughout the course, we'll highlight differences between Windows PowerShell and PowerShell Core where applicable, provide hands-on exercises to practice each concept, and offer best practices and exam guidance to reinforce learning.

## Objectives 
By the end of this course, you will be able to: 

- **Understand PowerShell Basics for SQL Server:** Explain the benefits of using PowerShell for SQL Server administration and recognize differences between Windows PowerShell and PowerShell Core in a database context.  
- **Manage On-Premises SQL Server with PowerShell:** Use PowerShell cmdlets and scripts to configure SQL Server instances, create databases, manage logins/users, and automate routine maintenance tasks (backups, checks).  
- **Automate Azure SQL Operations:** Leverage Azure PowerShell modules to deploy and manage Azure SQL resources (servers and databases), handle cloud connectivity (firewall rules, authentication), and integrate on-premises to cloud automation.  
- **Apply Best Practices:** Follow scripting best practices for security, reliability, and efficiency in PowerShell when working with SQL Server.  
- **Prepare for Exams and Real-world Scenarios:** Tackle exam-style questions and scenarios that test your knowledge of PowerShell for SQL, ensuring you can apply concepts in practice and in certification environments.

## PowerShell Basics and Environment Setup 
Before diving into SQL Server specifics, ensure your PowerShell environment is ready and understand core concepts:

- **Windows PowerShell vs. PowerShell Core:** Windows PowerShell (v5.1) is the legacy Windows-only version, whereas PowerShell Core (v7+) is cross-platform (Windows, Linux, macOS). In practice, both can be used for SQL Server automation. PowerShell Core 7 is built on .NET Core, making it portable, while Windows PowerShell is tied to the older .NET Framework (Windows-only). Most modern SQL PowerShell modules (like the SqlServer module and Azure Az modules) support PowerShell 7+, but some legacy features (e.g., certain older SQL provider functionalities) might work only on Windows PowerShell. We will note any differences in usage as we go.  
- **Installing Required Modules:** For on-premises SQL Server, install the **SqlServer** module which contains cmdlets for SQL Server (this replaces older `SQLPS` module). For Azure SQL, install the **Az** PowerShell modules (specifically the `Az.Sql` sub-module). You can install modules from the PowerShell Gallery by running:  
  ```powershell
  # Install SQL Server module (for SQL Server on-prem tasks)
  Install-Module -Name SqlServer -Scope CurrentUser
  
  # Install Azure module (includes Az.Sql for Azure SQL tasks)
  Install-Module -Name Az -Scope CurrentUser
  ```  
  Ensure you have an up-to-date PowerShell version (Windows PowerShell 5.1 or PowerShell 7.x) before installing. After installation, use `Import-Module SqlServer` or `Import-Module Az` to load them (in PowerShell 7+, modules may auto-import when you use their commands).  
- **Basic PowerShell Concepts Recap:** PowerShell uses a verb-noun syntax for commands (called *cmdlets*), e.g., `Get-Command`, `New-Item`. You can get help on any cmdlet with `Get-Help <Cmdlet-Name> -Online` (which opens the official docs) or `Get-Help <Cmdlet-Name> -Examples` for usage examples. PowerShell pipelines allow chaining commands, and objects output from one cmdlet can be filtered or processed by another (useful for handling lists of databases or servers). Familiarity with these basics will help in understanding the examples that follow.

## Managing On-Premises SQL Server with PowerShell 
This section covers how to use PowerShell for typical on-premises SQL Server administration tasks. We will explore connecting to a SQL Server instance, creating databases, managing logins and permissions, and automating maintenance like backups — all through PowerShell scripts and cmdlets.

### 1. Connecting to SQL Server and Exploring with PowerShell 
PowerShell offers a SQL Server **provider** that lets you navigate SQL Server like a file system, as well as cmdlets to run queries or retrieve SQL information: 

- **SQL Server PowerShell Provider:** When the SqlServer module is imported, it adds a provider path `SQLSERVER:\` in PowerShell. This acts like a drive that contains hierarchical objects for SQL Server instances, databases, tables, etc. For example, you can set your location to a SQL instance and list databases:  
  ```powershell
  Import-Module SqlServer             # Load SQL Server cmdlets and provider
  Set-Location SQLSERVER:\SQL\MYPC\DEFAULT    # Navigate to a SQL instance (machine "MYPC", default instance)
  Get-ChildItem Databases            # List databases on that instance
  ```  
  This will display databases as if they were folders. You can similarly navigate into a database and list Tables, Views, etc. This provider is available in Windows PowerShell and (with SqlServer module updates) in PowerShell Core as well.  
- **Running SQL Queries with PowerShell:** The most straightforward way to interact with SQL Server via PowerShell is the `Invoke-Sqlcmd` cmdlet. This cmdlet allows you to run T-SQL queries or scripts against a SQL Server. For example:  
  ```powershell
  # Execute a simple query on an instance and get results
  Invoke-Sqlcmd -ServerInstance "MYPC\DEFAULT" `
                -Database "master" `
                -Query "SELECT name, create_date FROM sys.databases"
  ```  
  This will connect to the specified instance and return the list of databases with their creation date. By default, if you don't specify credentials, it uses Windows Integrated Authentication (your current user). To use SQL Authentication, provide the `-Username` and `-Password` parameters (or use `-Credential` with a PSCredential object).  
- **PowerShell Core Note:** On PowerShell Core 7+ on Linux/macOS, integrated authentication might not work the same as on Windows. In such cases, use SQL Authentication or other methods. The `Invoke-Sqlcmd` cmdlet as provided by the SqlServer module is compatible with PowerShell Core (ensure you have the latest module version).  

#### Hands-On Exercise 1: Exploring an Instance with the SQL Provider 
1. **Open PowerShell:** Launch PowerShell (Windows PowerShell or PowerShell 7). If using PowerShell Core on a non-Windows OS, ensure you have network connectivity to a SQL Server instance and SQL authentication credentials if needed.  
2. **Import the SQL module:** Run `Import-Module SqlServer`. No output means it loaded successfully (you can verify by running `Get-Module SqlServer`).  
3. **Navigate the SQL provider:** Use `Set-Location SQLSERVER:\SQL\YOURSERVER\DEFAULT` (replace YOURSERVER with your machine or instance name; use `.\` for local default instance). Then run `Get-ChildItem` to list child items. You should see folders like **Databases**, **Security**, etc.  
4. **List databases:** Change to the Databases path: `Set-Location SQLSERVER:\SQL\YOURSERVER\DEFAULT\Databases`. Then run `Get-ChildItem` (or alias `ls`). This lists all databases on the instance as objects.  
5. **Retrieve a specific database:** Try `Get-Item "SQLSERVER:\SQL\YOURSERVER\DEFAULT\Databases\tempdb"` to get the tempdb database object. Notice you get details like Size, Owner, Status. This demonstrates how PowerShell can fetch SQL objects directly.  
6. **Return to normal path:** Use `Set-Location C:\` (or wherever) to exit the SQL provider when done.  

### 2. Automating SQL Server Setup and Configuration 
PowerShell can automate initial setup tasks for SQL Server or configuration changes:

- **Installing SQL Server via PowerShell:** While initial installation of the SQL Server software is often done via GUI or configuration file, PowerShell can be used for unattended installs (e.g., using `Invoke-Sqlcmd` to run configuration scripts or using setup.exe with parameters in a script). In this fundamentals course, we'll assume SQL Server is already installed. However, you can use PowerShell to verify installation or configure services. For example, to check that the SQL Service is running:  
  ```powershell
  Get-Service -Name MSSQLSERVER    # For default instance service status
  # Or for a named instance, service name like 'MSSQL$INSTANCENAME'
  ```  
  You could also use PowerShell to start/stop services (`Start-Service` or `Stop-Service`) as needed.  
- **Configuring Server Settings:** Many server-level configurations (like max memory, default backup directory, etc.) can be changed via T-SQL commands (`sp_configure`), which you could execute with `Invoke-Sqlcmd` in a script. For example:  
  ```powershell
  # Set max server memory to 2GB (2048 MB)
  Invoke-Sqlcmd -ServerInstance "MYPC\DEFAULT" -Database master `
    -Query "EXEC sp_configure 'max server memory (MB)', 2048; RECONFIGURE;"
  ```  
  This approach uses PowerShell as a way to automate running admin T-SQL commands across servers. In large environments, you could loop over a list of servers in PowerShell to apply standard configs.  
- **Creating a New Database:** Instead of using SQL Server Management Studio (SSMS) GUI, you can create a database via PowerShell easily. There isn't a built-in `New-SqlDatabase` cmdlet in the official module, but you can use T-SQL or the SQL Management Objects (SMO) library. The simplest way: run a CREATE DATABASE statement:  
  ```powershell
  $dbname = "TestDB"
  Invoke-Sqlcmd -ServerInstance "MYPC\DEFAULT" -Database master `
                -Query "CREATE DATABASE [$dbname];"
  ```  
  This will create a new database called TestDB on the target instance. You can confirm by running a query or using the provider to see it exists.  
- **Creating SQL Logins and Users:** User management often involves two steps in SQL Server (for on-prem): creating a **Login** at the server level (for SQL Authentication or linking a Windows account), and then creating a **User** in each database that the login should access. PowerShell can automate both. For example, to create a SQL Auth login and a database user:  
  ```powershell
  # Create a new SQL login on the server (SQL Authentication)
  Invoke-Sqlcmd -ServerInstance "MYPC\DEFAULT" -Database master -Query "
    CREATE LOGIN PowerUser WITH PASSWORD = 'P@ssw0rd';
  "
  # Create a user in TestDB for that login and give it db_owner role
  Invoke-Sqlcmd -ServerInstance "MYPC\DEFAULT" -Database TestDB -Query "
    CREATE USER PowerUser FOR LOGIN PowerUser;
    ALTER ROLE db_owner ADD MEMBER PowerUser;
  "
  ```  
  In a real scenario, **avoid using plain text passwords** as shown above. Instead, use secure methods (PowerShell can prompt for a secure password or use integrated Windows authentication). We use a simple example here for illustration.  
- **Permissions and Roles:** You can also automate granting specific permissions or adding users to roles with T-SQL via PowerShell. For instance, you might run a script to grant read access to certain databases for a list of user accounts, all driven by a PowerShell loop.  

#### Hands-On Exercise 2: Creating a Database and User with PowerShell 
1. **Create a new database:** Using PowerShell, create a database named "TrainingDB" on your SQL Server instance. Run:  
   ```powershell
   Invoke-Sqlcmd -ServerInstance "YOURSERVER\INSTANCE" -Database master -Query "CREATE DATABASE TrainingDB;"
   ```  
   (Replace `"YOURSERVER\INSTANCE"` with your SQL Server name; for a default instance on the local machine you can use `"."` or `"localhost"`.)  
2. **Verify creation:** After execution, use either `Invoke-Sqlcmd -Query "SELECT name FROM sys.databases"` or the SQL provider (`Get-ChildItem SQLSERVER:\SQL\YOURSERVER\INSTANCE\Databases`) to confirm "TrainingDB" is listed.  
3. **Create a SQL login:** Run the following to create a login (SQL authentication) called `TrainingUser`:  
   ```powershell
   Invoke-Sqlcmd -ServerInstance "YOURSERVER\INSTANCE" -Database master -Query "CREATE LOGIN TrainingUser WITH PASSWORD = 'Str0ngPass!';"
   ```  
   Ensure the password meets SQL Server complexity requirements.  
4. **Create a user in the new database:** Now map that login to a user in **TrainingDB** and give it a role. For example:  
   ```powershell
   Invoke-Sqlcmd -ServerInstance "YOURSERVER\INSTANCE" -Database TrainingDB -Query "CREATE USER TrainingUser FOR LOGIN TrainingUser; EXEC sp_addrolemember 'db_datareader', 'TrainingUser';"
   ```  
   This creates a user in TrainingDB and adds it to the `db_datareader` role (read access).  
5. **Test the access (optional):** You can test the new login by connecting using `Invoke-Sqlcmd` with `-Username "TrainingUser" -Password "Str0ngPass!"` and running a simple query on TrainingDB. (If the login is SQL Auth, also ensure SQL Server is set to Mixed Authentication mode.)  
6. **Cleanup (optional):** You may drop the user, login, or database via similar commands (`DROP USER`, `DROP LOGIN`, `DROP DATABASE`) if you want to tidy up, or keep them for subsequent exercises.

### 3. Automating Maintenance Tasks (Backups, Monitoring) 
One of the biggest advantages of PowerShell for DBAs is automating repetitive maintenance tasks across one or many servers:

- **Database Backups with PowerShell:** The `Backup-SqlDatabase` cmdlet can perform backups of a SQL database. For example, to backup the TrainingDB we created:  
  ```powershell
  Backup-SqlDatabase -ServerInstance "MYPC\DEFAULT" -Database "TrainingDB" -BackupFile "C:\Backup\TrainingDB_full.bak"
  ```  
  This will initiate a full backup of **TrainingDB** to the specified file path. You can parameterize this for multiple databases or use a loop to backup all user databases:  
  ```powershell
  # Backup all non-system databases on an instance
  $dbs = Invoke-Sqlcmd -ServerInstance "MYPC\DEFAULT" -Database master -Query "SELECT name FROM sys.databases WHERE database_id > 4"
  foreach ($db in $dbs.Name) {
      $filename = "C:\Backup\$db.bak"
      Backup-SqlDatabase -ServerInstance "MYPC\DEFAULT" -Database $db -BackupFile $filename
      Write-Host "Backed up $db to $filename"
  }
  ```  
  In this snippet, we retrieve all databases with an ID > 4 (which filters out system DBs like master, tempdb, etc.), then loop and backup each to a file. **Note:** Ensure the path exists and SQL Server service account has permission to write to that path.  
- **Restoring Databases:** Similarly, `Restore-SqlDatabase` can be used to restore from a .bak file, specifying the file and target instance and database name. (Exercise caution to not overwrite important databases in production!)  
- **Automating Index Maintenance or Checks:** You can use PowerShell to run any T-SQL maintenance script (like rebuilding indexes or updating statistics) across servers. For instance, if you have a T-SQL script for index maintenance, you could call it via `Invoke-Sqlcmd` on each database or each server. There are community modules like **DBATools** that provide rich functions (e.g., `Invoke-DbaDbIndexOptimization` for index maintenance, `Test-DbaLastBackup` to verify backups) which you might explore after this course. In this fundamentals course, we focus on native tools, but be aware of these powerful community scripts.
- **Scheduling Scripts:** To truly automate these tasks regularly, you can schedule PowerShell scripts to run at certain times:
  - On-premises, use **SQL Server Agent** to create a job of type PowerShell (which runs under the agent's context using the SQLPS module) or have a job step execute a PowerShell script (by calling the PowerShell executable).  
  - Alternatively, use **Windows Task Scheduler** to run a PowerShell script on a schedule (e.g., nightly backups). Ensure the task runs with an account that has necessary permissions on SQL and the file system.  
  - In PowerShell 7, you could also use `Register-ScheduledJob` to schedule jobs via PowerShell itself.
  
#### Hands-On Exercise 3: Backup and Task Automation 
1. **Backup a database via PowerShell:** Using the `Backup-SqlDatabase` cmdlet, back up the **TrainingDB** (or another test database) to a file. For example:  
   ```powershell
   Backup-SqlDatabase -ServerInstance "YOURSERVER\INSTANCE" -Database "TrainingDB" -BackupFile "C:\Backup\TrainingDB.bak"
   ```  
   (Make sure the directory `C:\Backup` exists or choose a path that does. If using a named instance, ensure SQL Agent or proper permissions are in place to write to that path.)  
2. **Verify backup success:** PowerShell will throw an error if the backup fails. If it succeeds, check the output or the file system to confirm the .bak file is created. You can also use T-SQL or `Invoke-Sqlcmd -Query "RESTORE HEADERONLY FROM DISK='C:\Backup\TrainingDB.bak'"` to verify the backup file contents.  
3. **Schedule a simple task (theoretical):** Describe how you would schedule this backup script to run every night. (You don't have to actually schedule it for the exercise, but think through it or try setting up a Windows Scheduled Task manually pointing to `powershell.exe` and your script file.) Consider what account it would run as and how it will authenticate to SQL Server.  
4. **Optional - List maintenance info:** Use `Invoke-Sqlcmd` to run a DBCC check or query index stats. For example:  
   ```powershell
   Invoke-Sqlcmd -ServerInstance "YOURSERVER\INSTANCE" -Database TrainingDB -Query "SELECT index_id, avg_fragmentation_in_percent FROM sys.dm_db_index_physical_stats(DB_ID(), NULL, NULL, NULL, 'SAMPLED')"
   ```  
   This returns index fragmentation for the TrainingDB. In a real scenario, you might put logic in PowerShell to act on this data (like rebuild indexes if fragmentation > some threshold).  

### 4. PowerShell Core Considerations for On-Prem SQL 
Most of the above examples will run similarly on PowerShell Core (7+) on Windows. If you are running PowerShell on Linux or Mac and connecting to SQL Server (which could be a remote Windows or Linux SQL Server instance): 
- Make sure the `SqlServer` module is installed on your PowerShell Core. On non-Windows, you might need to install **ODBC drivers** or **System.Data.SqlClient/SqlServer .NET packages** for connectivity. The SqlServer module uses .NET libraries under the hood for SQL connections which are supported cross-platform.  
- Windows authentication from a Linux/Mac environment to SQL Server may not be straightforward. You would typically use SQL authentication in that case (username/password), or if in an Active Directory environment, possibly Kerberos/NTLM setups (beyond scope here).  
- If certain cmdlets are missing on PowerShell Core, ensure you have the latest module. Some older cmdlets or the SQL provider might have had limitations historically, but as of latest versions, core functionality (Invoke-Sqlcmd, Backup/Restore-SqlDatabase, etc.) should work in PowerShell 7. Always test your scripts in the environment you plan to run them.  

## Managing Azure SQL with PowerShell 
Cloud-based SQL Server offerings (Azure SQL Database and Azure SQL Managed Instance) can also be managed with PowerShell. In this section, we cover how to use the Azure PowerShell Az modules to automate common Azure SQL tasks: creating servers and databases, configuring security (firewall rules, Azure AD integration), and connecting your on-prem environment to Azure services.

### 1. Connecting to Azure and Setting Up Environment 
To manage Azure resources, you must use the Azure PowerShell module (Az):

- **Azure Az Module:** Ensure the Az module is installed (`Install-Module Az`). The Az module is compatible with both Windows PowerShell and PowerShell Core. It's actually recommended to use PowerShell 7+ for Az module for best performance and up-to-date features.  
- **Login to Azure:** Before running Azure cmdlets, authenticate your session:  
  ```powershell
  Connect-AzAccount
  ```  
  This will prompt you to sign in (device code or browser). After logging in, your PowerShell session knows your Azure account context. Use `Get-AzSubscription` and `Select-AzSubscription` if you have multiple subscriptions to ensure the correct one is active.  
- **Azure Cloud Shell (alternative):** Azure offers Cloud Shell, an in-browser PowerShell environment with Az modules pre-installed. You can use that for Azure SQL tasks without installing anything locally. In Cloud Shell, you're automatically logged in to your Azure account, but you still need to select the right subscription if more than one. Using Cloud Shell can simplify setup, especially if on a machine where you cannot install modules.

### 2. Creating an Azure SQL Server and Database with PowerShell 
Azure SQL Database (Platform-as-a-Service) resides within an **Azure SQL Server** (a logical server, not a VM). Let's go through provisioning these:

- **Resource Group (RG):** It's typical to create or use an existing resource group to contain your Azure SQL server and databases. For example:  
  ```powershell
  New-AzResourceGroup -Name "SQL-PowerShell-RG" -Location "EastUS"
  ```  
  Choose a location near you or required by your scenario. If the RG exists, this will just return the object. 
- **Create Azure SQL Server:** Now create the logical server:  
  ```powershell
  $cred = Get-Credential -Message "Enter Azure SQL admin credentials (will be created)"
  New-AzSqlServer -ResourceGroupName "SQL-PowerShell-RG" -ServerName "pssqlserver01" -Location "EastUS" -SqlAdministratorCredentials $cred
  ```  
  This command creates an Azure SQL Server named "pssqlserver01" in EastUS with an admin username/password you provide (via the credential prompt). The admin login will be like a "server-level principal" for your Azure SQL instance (analogous to a sa account, but for the Azure SQL logical server).  
- **Configure Firewall for Connectivity:** By default, Azure SQL blocks external connections. To connect from your machine or from certain Azure services, set firewall rules:
  - To allow your current public IP (for connecting from your local PowerShell or SSMS): find out your IP and use:  
    ```powershell
    New-AzSqlServerFirewallRule -ResourceGroupName "SQL-PowerShell-RG" -ServerName "pssqlserver01" -FirewallRuleName "AllowMyIP" -StartIpAddress "<YourPublicIP>" -EndIpAddress "<YourPublicIP>"
    ```  
    Replace `<YourPublicIP>` with your actual IP. This will open the firewall for *just* that IP.  
  - Optionally, allow Azure services to connect (so Azure VMs or Azure Cloud Shell can connect):  
    ```powershell
    New-AzSqlServerFirewallRule -ResourceGroupName "SQL-PowerShell-RG" -ServerName "pssqlserver01" -FirewallRuleName "AllowAzureServices" -StartIpAddress "0.0.0.0" -EndIpAddress "0.0.0.0"
    ```  
    The special rule with start and end as 0.0.0.0 lets Azure data centers through (useful if running scripts from Cloud Shell or Azure Automation). Be cautious: this setting opens access to any Azure resource (still requires valid credentials though).  
- **Create a Database on the Azure SQL Server:** Now that the server is up, create an actual database:  
  ```powershell
  New-AzSqlDatabase -ResourceGroupName "SQL-PowerShell-RG" -ServerName "pssqlserver01" -DatabaseName "DemoDB" -Edition "Basic"
  ```  
  Here we create a database named "DemoDB" on our server, using the **Basic** tier (a small, low-cost option suitable for development/testing). You could choose Standard, Premium, or vCore options as needed (with additional parameters for size, compute, etc.). The command will return details of the new database (status, edition, etc.).  
- **Verify creation:** You can retrieve the database or server information with `Get-AzSqlServer` and `Get-AzSqlDatabase`. For example:  
  ```powershell
  Get-AzSqlServer -ResourceGroupName "SQL-PowerShell-RG" -ServerName "pssqlserver01"
  Get-AzSqlDatabase -ResourceGroupName "SQL-PowerShell-RG" -ServerName "pssqlserver01" -DatabaseName "DemoDB"
  ```  
  These cmdlets show the properties, confirming that the resources exist and are online.

#### Hands-On Exercise 4: Provision Azure SQL and Connect
*Note: This exercise requires an Azure subscription and appropriate permissions.*  
1. **Login to Azure:** Open PowerShell and run `Connect-AzAccount` to authenticate.  
2. **Create resource group:** Choose a region and create a resource group for this exercise:  
   ```powershell
   New-AzResourceGroup -Name "PSQL-Lab-RG" -Location "WestEurope"
   ```  
   (Replace location with one near you or required.)  
3. **Create Azure SQL Server:** Decide on a server name (it must be globally unique across Azure, e.g., use your initials and date to make it unique). Then run:  
   ```powershell
   $sqlAdminCred = Get-Credential -UserName "sqladminuser" -Message "Enter a strong password for the new Azure SQL server admin account"
   New-AzSqlServer -ResourceGroupName "PSQL-Lab-RG" -ServerName "youruniquesqlserver" -Location "WestEurope" -SqlAdministratorCredentials $sqlAdminCred
   ```  
   Wait for the command to complete. It will output the new server details.  
4. **Set firewall rule:** Allow your client IP to access the server:  
   ```powershell
   $myIP = "X.X.X.X"  # replace with your public IP
   New-AzSqlServerFirewallRule -ResourceGroupName "PSQL-Lab-RG" -ServerName "youruniquesqlserver" -FirewallRuleName "AllowClientIP" -StartIpAddress $myIP -EndIpAddress $myIP
   ```  
   If you're running from an Azure VM or Cloud Shell, you might instead use the 0.0.0.0 rule as described above.  
5. **Create a database:** Now create a new database on the server:  
   ```powershell
   New-AzSqlDatabase -ResourceGroupName "PSQL-Lab-RG" -ServerName "youruniquesqlserver" -DatabaseName "LabDB" -Edition "Basic"
   ```  
   This will create a small database. You should see its status and edition in the output.  
6. **Test connectivity:** Use `Invoke-Sqlcmd` from your machine to connect to the new Azure SQL database (requires the SqlServer module on your machine):  
   ```powershell
   Invoke-Sqlcmd -ServerInstance "youruniquesqlserver.database.windows.net" -Database "LabDB" -Username "sqladminuser" -Password "<YourPassword>" -Query "SELECT @@VERSION;"
   ```  
   This should return information about the SQL Azure instance version if successful. (If you get a connectivity timeout, double-check the firewall IP rule. If you get a login error, verify the username/password – Azure SQL admin user is the one you set as $sqlAdminCred.)  

7. **Cleanup (optional):** After verifying, you can remove resources to avoid charges:  
   ```powershell
   Remove-AzSqlDatabase -ResourceGroupName "PSQL-Lab-RG" -ServerName "youruniquesqlserver" -DatabaseName "LabDB" -Force
   Remove-AzSqlServer -ResourceGroupName "PSQL-Lab-RG" -ServerName "youruniquesqlserver" -Force
   Remove-AzResourceGroup -Name "PSQL-Lab-RG" -Force
   ```  
   These commands delete the database, server, and resource group respectively. **Use with caution** to avoid accidentally deleting the wrong resources.

### 3. Azure SQL User Management and Security via PowerShell 
Managing users in Azure SQL Database is similar to on-prem, but with some differences:
- **SQL Auth and Azure AD Auth:** The Azure SQL server admin we created is a SQL authentication account. You can create additional SQL auth logins (which in Azure SQL are scoped to the server or the individual database as *contained users*). There isn't a separate `CREATE LOGIN` for individual DBs because Azure SQL (single database) supports contained database users. To create a new user in Azure SQL Database, you typically connect to the database and run T-SQL `CREATE USER ... WITH PASSWORD ...` (for a contained SQL user) or create an Azure AD user if integrated with Azure AD.  
- **Using PowerShell to create users:** There is no Az cmdlet to create a database user because it's a database-level operation (not an Azure resource management operation). Instead, you’d use `Invoke-Sqlcmd` against the Azure SQL Database to run the T-SQL to create users or roles. For example, to create a read-only user on the *LabDB* we made:  
  ```powershell
  # Connect to LabDB and create a new contained SQL user
  Invoke-Sqlcmd -ServerInstance "youruniquesqlserver.database.windows.net" -Database "LabDB" -Username "sqladminuser" -Password "<YourPassword>" -Query "
    CREATE USER reportingUser WITH PASSWORD = 'User@1234';
    ALTER ROLE db_datareader ADD MEMBER reportingUser;
  "
  ```  
  This assumes the server is set to allow SQL authentication and you use the admin to create a new user. The new user 'reportingUser' can now independently connect to LabDB (with server name, database name, and its own password) but has only read access (db_datareader role).  
- **Azure AD Admin and Users:** Azure SQL allows an Azure AD account to be set as an admin (instead of using the SQL auth admin). Using PowerShell, you can configure an Azure AD admin for the server with `Set-AzSqlServerActiveDirectoryAdministrator`. You could then create Azure AD-based users in the database (e.g., `CREATE USER [user@tenant.com] FROM EXTERNAL PROVIDER;`). This requires Azure AD setup and is beyond basic fundamentals, but be aware it's an option for managing access in enterprise scenarios (and it avoids needing multiple SQL auth logins).  
- **Firewall and Network Security Recap:** Remember that any PowerShell script targeting Azure SQL must consider firewall rules. If you plan to run automation (like an Azure Automation Runbook or a local scheduled script) that connects to Azure SQL, ensure the IP or service is allowed. Azure Automation, for example, can use a managed identity or you might allow its outbound IP via firewall rule.  

### 4. Automating Azure SQL Maintenance 
Just like on-prem, you may need to automate maintenance tasks for Azure SQL databases:
- **Backups:** One big difference is that Azure SQL databases are automatically backed up by the platform (point-in-time restore is built-in). You typically do not need to run manual backups to file as you would on-prem. In fact, Azure SQL does not expose a direct `Backup-SqlDatabase` capability to dump .bak files (except for Managed Instance, which is more like on-prem). If you need a copy of the database, you might use **bacpac** exports or `New-AzSqlDatabaseExport` to export to Azure storage, or use `New-AzSqlDatabaseCopy` to copy the database. These are done via Azure commands or Data-tier Application Framework, not via the on-prem Backup-SqlDatabase cmdlet.  
- **Scaling and Performance:** Azure PowerShell can automate actions like scaling a database or pausing/resuming (for Azure SQL elastic pools or Hyperscale etc.). For example, using `Set-AzSqlDatabase` you could change the performance tier or compute size. Monitoring can be automated by querying Azure metrics or using `Get-AzMetric` on the database resource.  
- **Azure Automation & Runbooks:** For recurring tasks in Azure (like scaling at certain hours or running cleanup scripts), Azure Automation can run PowerShell scripts in the cloud on a schedule. You could write a runbook that uses `Invoke-Sqlcmd` with an Azure run-as account or managed identity to perform tasks on the Azure SQL DB (such as purging old records, updating data, etc.). The advantage is no VM or on-prem scheduler needed, and the runbook has network access within Azure (still needs proper firewall/credentials).  

#### Hands-On Exercise 5: Azure SQL Automation Scenario 
*(Thought exercise – optionally implement if you have resources)*  
- **Scenario:** Your Azure SQL Database *LabDB* should be scaled up to a higher performance tier during business hours and scaled down at night to save cost. How can PowerShell help?  
- **Solution Outline:** You could use an Azure Automation account or a local scheduled script that runs `Set-AzSqlDatabase` to change the edition or compute capacity on a schedule. For example, at 8am run:  
  ```powershell
  Set-AzSqlDatabase -ResourceGroupName "PSQL-Lab-RG" -ServerName "youruniquesqlserver" -DatabaseName "LabDB" -Edition "Standard" -ServiceObjectiveName "S1"
  ```  
  and at 8pm run:  
  ```powershell
  Set-AzSqlDatabase -ResourceGroupName "PSQL-Lab-RG" -ServerName "youruniquesqlserver" -DatabaseName "LabDB" -Edition "Basic"
  ```  
  This would scale the database up in the morning (to Standard S1 tier) and down to Basic at night. You'd schedule these with Azure Automation runbooks or Windows Task Scheduler (with `Connect-AzAccount` or a saved service principal login in the script for non-interactive auth).  
- **Monitoring Example:** You might also write a PowerShell script to monitor DTU or vCore usage of your Azure SQL DB and email an alert if usage is consistently high. While Azure has built-in alerting, doing it via PowerShell might involve retrieving metrics and using `Send-MailMessage` for notifications (or better, integrate with Azure Monitor cmdlets).  

## Best Practices for PowerShell and SQL Server 
When using PowerShell for SQL Server administration, keep these best practices in mind to ensure safety, security, and efficiency:

- **Use Least Privilege:** Run your PowerShell sessions with an account that has the necessary SQL permissions for the task, and no more. Avoid using the system administrator (`sa`) or high-privileged accounts for routine tasks if a lower privilege account suffices. In Azure, prefer Azure AD accounts or managed identities with narrow scopes for automation tasks.  
- **Secure Credentials:** Never hard-code sensitive credentials (like passwords) in scripts. Use `Get-Credential` to prompt for credentials or store them securely (for example, using the Windows Credential Manager, Azure Key Vault, or an encrypted file). In scripts, you can accept a PSCredential object as a parameter. For scheduled tasks, consider using managed service accounts or other secure methods to authenticate without exposing passwords.  
- **Error Handling and Logging:** Add error handling to your scripts (`Try/Catch` in PowerShell) especially for automation that runs unattended. Log the outcomes of operations to a file or monitoring system. For example, when running a batch of maintenance commands across servers, log successes and failures so you can review them later.  
- **Idempotence and Testing:** Write scripts that can be run multiple times without adverse effects. For instance, a script that creates a login should check if the login exists first to avoid errors. Always test your PowerShell scripts in a non-production environment or against a test SQL instance before running in production. This helps catch mistakes that could have major impact (like running a wrong query via `Invoke-Sqlcmd`).  
- **Use Modules and Community Tools Wisely:** The built-in SqlServer and Az modules cover most needs, but don't reinvent the wheel. If you find yourself needing complex tasks, explore community modules like **DBATools** (which offers hundreds of functions for SQL Server) or Azure automation scripts available on platforms like GitHub. However, ensure you understand what a script does before running it in your environment. Stick to well-known sources and test them yourself.  
- **Keep PowerShell and Modules Updated:** Microsoft regularly updates the SqlServer and Az modules with new cmdlets and bug fixes (for example, improved compatibility with PowerShell Core). Update your modules (`Update-Module SqlServer`, `Update-Module Az`) periodically. Also, use the latest PowerShell version where possible for better performance and features. As of now, PowerShell 7.x is the recommended version for most scenarios, while Windows PowerShell 5.1 is in maintenance mode.  
- **Documentation and Source Control:** Document your scripts with comments (describe what each part does, any prerequisites). Use source control (e.g., git) for your PowerShell scripts, especially if working in a team. This way, you track changes and can roll back if a change introduces issues.  

By following these best practices, you ensure that your use of PowerShell in managing SQL Server is secure, robust, and maintainable.

## Exam Guidance 
If you are preparing for a certification exam or an interview that covers PowerShell for SQL Server (for example, Microsoft Database Administration exams often include some PowerShell content), consider the following guidance:

- **Know the Core Cmdlets:** Be sure you remember the key PowerShell cmdlets for SQL Server and Azure SQL management. For on-prem, this includes `Invoke-Sqlcmd`, `Backup-SqlDatabase`, `Restore-SqlDatabase`, and understanding the SQL provider (`SQLSERVER:\`). For Azure, know `New-AzSqlServer`, `New-AzSqlDatabase`, `Get/Set-AzSqlDatabase`, `New-AzSqlServerFirewallRule`. Exams may ask which cmdlet is appropriate for a given task.  
- **Understand Differences Between Environments:** Be prepared for questions on when you would use Windows PowerShell vs PowerShell Core, or how running a script on Linux might differ (e.g., you might need to use SQL authentication on Linux). Know that the Az module works in both, and that the SqlServer module now supports cross-platform in PowerShell 7.  
- **Practice Common Scenarios:** Exams may present a scenario: "You need to automate X... which tool or approach do you use?" For example, scheduling regular index maintenance – the expected answer might be "use a PowerShell script scheduled via SQL Agent or Task Scheduler that runs the necessary T-SQL or uses a cmdlet". Or a question might describe a series of commands and ask what the result is (e.g., showing a snippet that creates a user and grants role, and asking what permissions the user will have). Make sure you understand the outcomes of the PowerShell commands you learned.  
- **Scripting and Syntax:** Sometimes, exams include partial scripts and ask you to fill in a missing cmdlet or parameter. Pay attention to PowerShell syntax (such as using the backtick `` ` `` for line continuation, or the difference between using a cmdlet vs. an operator). You likely won't need to write long scripts from scratch in an exam, but you should be comfortable reading and interpreting PowerShell code related to SQL tasks.  
- **Focus on Automation Benefits:** Be ready to answer conceptual questions like "What are the advantages of using PowerShell over manual GUI methods for SQL administration?" or "Which tasks in SQL Server are well-suited for PowerShell automation?" – answers would include consistency, the ability to run on multiple servers, scheduling, integration with other automation, etc.  
- **Azure Specifics:** If the exam is cloud-focused, know how to use PowerShell to deploy Azure SQL resources versus using the Azure Portal. Understand concepts like Azure Cloud Shell (PowerShell in the browser) and Azure Automation as it relates to running scripts. You might get a question like "How can an administrator schedule a PowerShell script to run against an Azure SQL Database without needing to manage a server?" (Answer could be: use an Azure Automation Runbook with appropriate credentials).  

**Tip:** Even if not required, practicing hands-on is the best way to reinforce this knowledge for an exam. Try to write small scripts for each objective and run them. It will help you remember cmdlet names and gotchas when answering questions under pressure.

## Quiz: Test Your Knowledge 
Finally, try these exam-style questions to reinforce key concepts from this course. The answers aren't provided here, so use them to gauge your understanding and identify areas to review:

1. **PowerShell Basics:** What is the primary difference between Windows PowerShell 5.1 and PowerShell 7 in the context of SQL Server automation?  
   A. There is no difference; both are identical.  
   B. PowerShell 7 is cross-platform and often requires updated modules for SQL, while Windows PowerShell 5.1 runs only on Windows.  
   C. PowerShell 7 cannot use the SqlServer module.  
   D. Windows PowerShell 5.1 is open-source, whereas PowerShell 7 is not.  

2. **On-Premises Cmdlets:** Which PowerShell cmdlet would you use to execute a T-SQL query on a SQL Server instance and get results in PowerShell?  
   A. `Get-SqlQuery`  
   B. `Invoke-Sqlcmd`  
   C. `Start-SqlQuery -Execute`  
   D. `Select-Sql`  

3. **Automating Backups:** You want to automate full backups for all user databases on a SQL Server nightly. Which combination of tools is most appropriate?  
   A. Use `Backup-SqlDatabase` in a PowerShell script and schedule it with SQL Server Agent or Task Scheduler.  
   B. Use the Azure portal to schedule backups.  
   C. Use `sqlcmd` in a DOS batch file for each database.  
   D. Set up a SQL maintenance plan and no PowerShell is needed.  

4. **Azure Provisioning:** Which of the following is **not** required when creating a new Azure SQL Database via PowerShell?  
   A. Logging in with `Connect-AzAccount`.  
   B. Creating or specifying a Resource Group.  
   C. Specifying a SQL Server name and admin credentials.  
   D. Installing SQL Server on an Azure VM first.  

5. **Azure Connectivity:** After deploying an Azure SQL Database, your PowerShell `Invoke-Sqlcmd` from your local machine fails to connect. What is the most likely cause, and solution?  
   A. The Azure SQL Database does not support PowerShell connections; use only SSMS.  
   B. The firewall is blocking your IP; create a firewall rule for your IP on the Azure SQL Server.  
   C. You must use Windows Authentication only; create an Azure AD account.  
   D. The database is not started; you need to start the Azure SQL Database service.  

6. **User Management:** True or False: In Azure SQL Database, you use `New-AzSqlUser` cmdlet to create a new database user.  

7. **Scripting Practices:** You have a script that creates a SQL login with a password. Which of these is a best practice improvement for that script?  
   A. Hard-code a strong password in the script so it’s the same every time.  
   B. Convert it to use Windows Authentication only and remove password handling.  
   C. Prompt the admin running the script for a password or retrieve it securely, instead of plain text in the script.  
   D. Email the password to the DBA team after creation for record-keeping.  

8. **PowerShell Pipeline:** If you run the following, what will it do?  
   ```powershell
   Get-AzSqlDatabase -ResourceGroupName "SQL-RG" -ServerName "prodserver" | Where-Object {$_.Status -eq "Online"} | Measure-Object
   ```  
   A. Change the status of all databases on prodserver to "Online".  
   B. Count how many databases on "prodserver" are currently Online.  
   C. List all online databases and measure their size.  
   D. It will throw an error because `Measure-Object` cannot be used on databases.  

9. **PowerShell vs T-SQL:** Which task is **easier or more practical** to do with PowerShell rather than with only T-SQL scripts?  
   A. Creating a table and inserting data.  
   B. Running a one-time ad-hoc query.  
   C. Backing up all databases across multiple SQL Server instances.  
   D. Selecting records with a complex JOIN.  

10. **Azure Automation:** You want to regularly run a housekeeping T-SQL script on your Azure SQL Database without running it from your local machine each time. What Azure feature can you use in combination with PowerShell to achieve this?  
    A. Azure Functions with a SQL trigger.  
    B. Azure Logic Apps SQL Connector.  
    C. Azure Automation Runbook with PowerShell.  
    D. SQL Server Agent on Azure SQL (single database).  

_Review your answers and ensure you can justify why each choice is correct or incorrect. Revisit the relevant sections of this course for any topics you're unsure about. Good luck with your PowerShell and SQL Server automation journey!_
