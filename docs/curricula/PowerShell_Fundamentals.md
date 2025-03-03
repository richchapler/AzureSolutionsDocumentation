# PowerShell: Fundamentals

This curriculum provides a structured introduction to using PowerShell (in both Windows PowerShell 5.1 and cross‑platform PowerShell Core 7+) for SQL Server administration. You will learn how to perform common tasks such as SQL Server setup, user and permission management, automating maintenance, and connecting to Azure SQL resources using dedicated PowerShell cmdlets and scripts. Throughout the course, we’ll highlight differences between Windows PowerShell and PowerShell Core where applicable, offer hands‑on exercises to practice each concept, and provide best practices and exam guidance to reinforce learning.

## Objectives 

- Understand PowerShell Basics for SQL Server: Explain the benefits of using PowerShell for SQL Server administration and recognize differences between Windows PowerShell and PowerShell Core in a database context.  
- Manage On-Premises SQL Server with PowerShell: Use PowerShell cmdlets and scripts to configure SQL Server instances, create databases, manage logins/users, and automate routine maintenance tasks (backups, checks).  
- Automate Azure SQL Operations: Leverage Azure PowerShell modules to deploy and manage Azure SQL resources (servers and databases), handle cloud connectivity (firewall rules, authentication), and integrate on‑premises to cloud automation.  
- Apply Best Practices: Follow scripting best practices for security, reliability, and efficiency in PowerShell when working with SQL Server.  
- Prepare for Exams and Real‑world Scenarios: Tackle exam‑style questions and scenarios that test your knowledge of PowerShell for SQL, ensuring you can apply concepts in practice and in certification environments.

------------------------- ------------------------- ------------------------- -------------------------

## Windows PowerShell (on-prem)

### Virtual Machine

Instantiate an Azure Virtual Machine (or equivalent):
  * Image: "Free SQL Server License: SQL Server 2022 Developer on Windows Server 2022"
  * SKU: "Standard_D2s_v3..."
  * Inbound Ports: RDP (3389) allowed
  * Boot Diagnostics: Disable

------------------------- -------------------------

### PowerShell 7

Navigate to https://learn.microsoft.com/en-us/powershell/scripting/install/installing-powershell-on-windows.

Scroll to "Installing the MSI package".

<img src="https://github.com/user-attachments/assets/697cea24-a25d-4cf0-becc-0100538daf9e" width="800" title="Snipped March 3, 2025" />

Download and install the latest MSI version.

<img src="https://github.com/user-attachments/assets/ec8bfb4c-b506-46a6-908c-fc091642d438" width="400" title="Snipped March 3, 2025" />

Run "PowerShell 7 (x64)" as an administrator.

<img src="https://github.com/user-attachments/assets/9afce18d-9b4c-4d7d-8994-7b591f7bb0ff" width="600" title="Snipped March 3, 2025" />

------------------------- -------------------------








---

### 4. Install and Import the SqlServer Module

Once you have PowerShell 7+ (or if you remain on Windows PowerShell 5.1), you can install the SqlServer module for SQL Server management tasks.

1. Install the SqlServer Module:
   ```powershell
   Install-Module -Name SqlServer -Scope CurrentUser
   ```
2. Import the Module:
   ```powershell
   Import-Module SqlServer
   ```

With the SqlServer module imported, you can use cmdlets like `Invoke-Sqlcmd`, `Backup-SqlDatabase`, and the `SQLSERVER:\` provider path to manage your on-premises SQL Server.

---

### 5. Verify SQL Server Connectivity

- Check Network Access:  
  Ensure your VM can reach your on-premises SQL Server instance.  
- Authentication Mode:  
  By default, Windows Integrated Authentication is used. If you need SQL Authentication, provide credentials when running SQL commands.

---

That’s it! You’ve successfully installed winget without the Microsoft Store, upgraded to PowerShell 7+, and installed the SqlServer module for managing your SQL Server environment.










---

### SQL Server Connectivity

- Ensure Connectivity:  
  Verify that your workstation has network access to your on‑premises SQL Server instance. By default, Windows Integrated Authentication is used (or supply credentials if using SQL Authentication).

---

Using phrases like "execute the following PowerShell command" is clear and direct for instructions. Feel free to use that phrasing to guide users through each step.

### Key Topics and Exercises for On‑Premises Administration

#### 1. Connecting to SQL Server and Exploring with PowerShell

- SQL Server PowerShell Provider:  
  Navigate the SQL Server hierarchy:
  ```powershell
  Set-Location SQLSERVER:\SQL\YOURSERVER\DEFAULT
  Get-ChildItem Databases
  ```
- Running Queries with Invoke‑Sqlcmd:  
  Execute a T‑SQL query:
  ```powershell
  Invoke-Sqlcmd -ServerInstance "YOURSERVER\DEFAULT" -Database master -Query "SELECT name, create_date FROM sys.databases;"
  ```

Hands‑On Exercise 1:  
1. Open PowerShell on your Windows machine.  
2. Import the SqlServer module and navigate to your SQL instance using the SQL provider.  
3. List the databases and retrieve details for a specific database (e.g., `tempdb`).  
4. Return to your normal file system.

#### 2. Automating SQL Server Setup and Configuration

- Verifying SQL Server Services:  
  Check service status:
  ```powershell
  Get-Service -Name MSSQLSERVER
  ```
- Creating a New Database:  
  ```powershell
  Invoke-Sqlcmd -ServerInstance "YOURSERVER\DEFAULT" -Database master -Query "CREATE DATABASE TrainingDB;"
  ```
- Creating a SQL Login and Mapping to a Database User:  
  ```powershell
  Invoke-Sqlcmd -ServerInstance "YOURSERVER\DEFAULT" -Database master -Query "
    CREATE LOGIN TrainingUser WITH PASSWORD = 'Str0ngPass!';
  "
  Invoke-Sqlcmd -ServerInstance "YOURSERVER\DEFAULT" -Database TrainingDB -Query "
    CREATE USER TrainingUser FOR LOGIN TrainingUser;
    EXEC sp_addrolemember 'db_datareader', 'TrainingUser';
  "
  ```

Hands‑On Exercise 2:  
Create a database named "TrainingDB," add a SQL login named `TrainingUser`, map it to "TrainingDB," and assign read permissions. Verify by listing databases.

#### 3. Automating Maintenance Tasks

- Database Backup Example:  
  ```powershell
  Backup-SqlDatabase -ServerInstance "YOURSERVER\DEFAULT" -Database "TrainingDB" -BackupFile "C:\Backup\TrainingDB.bak"
  ```
- Scheduling Scripts:  
  Explain how you would schedule the backup script using SQL Server Agent or Windows Task Scheduler for nightly execution.

Hands‑On Exercise 3:  
Backup "TrainingDB" to a file and verify the backup file. Optionally, describe the process for scheduling the script.

#### 4. PowerShell Core Considerations for On‑Premises

- Ensure the SqlServer module is up‑to‑date when using PowerShell Core.  
- On non‑Windows systems, SQL Authentication is recommended.

---

### Quiz: Test Your Knowledge – On‑Premises

1. On‑Prem Environment: Which cmdlet is used to run a T‑SQL query on a local SQL Server instance?  
   A. `Get-SqlQuery`  
   B. `Invoke-Sqlcmd`  
   C. `Run-SqlQuery`  
   D. `Start-Sqlcmd`

2. SQL Provider Navigation: What is the provider path added by the SqlServer module for on‑premises SQL Server?  
   A. `SQLSERVER:\`  
   B. `SQL:\`  
   C. `DBSERVER:\`  
   D. `SQLDB:\`

3. Creating a Database: Which command creates a new database in your on‑prem SQL Server?  
   A. `CREATE DATABASE TrainingDB;`  
   B. `NEW DATABASE TrainingDB;`  
   C. `MAKE DATABASE TrainingDB;`  
   D. `ADD DATABASE TrainingDB;`

4. User Mapping: After creating a SQL login, what must you do to allow that login to access a specific database?  
   A. Nothing; logins automatically have access.  
   B. Create a corresponding database user and map the login.  
   C. Restart the SQL Server.  
   D. Change the authentication mode.

5. Automated Backups: Which tool is best used to automate database backups across multiple SQL Server instances on‑premises?  
   A. SQL Server Management Studio (SSMS) only  
   B. A PowerShell script using `Backup-SqlDatabase` scheduled via SQL Server Agent or Task Scheduler  
   C. Manual backup through the GUI  
   D. Windows PowerShell ISE without scheduling

---

## Section 2: Azure PowerShell (Managing Azure SQL)

This section focuses on using the Azure PowerShell (Az) modules to deploy and manage Azure SQL resources, including servers and databases.

### Environment Setup for Azure Tasks

- Module Installation:  
  - Install the Az module:
    ```powershell
    Install-Module -Name Az -Scope CurrentUser
    ```
  - Import the module:
    ```powershell
    Import-Module Az
    ```

- Authentication:  
  - Log in to Azure:
    ```powershell
    Connect-AzAccount
    ```
  - If needed, select your subscription:
    ```powershell
    Select-AzSubscription -SubscriptionName "Your Subscription Name"
    ```

- Cloud Shell Option:  
  - Alternatively, use Azure Cloud Shell (available in the Azure Portal) which comes pre‑installed with the Az modules and is already authenticated.

### Key Topics and Exercises for Azure Administration

#### 1. Creating an Azure SQL Server and Database

- Resource Group Creation:  
  ```powershell
  New-AzResourceGroup -Name "PSQL-Lab-RG" -Location "WestEurope"
  ```
- Provisioning the Azure SQL Server:  
  ```powershell
  $sqlAdminCred = Get-Credential -UserName "sqladminuser" -Message "Enter a strong password for the new Azure SQL server admin"
  New-AzSqlServer -ResourceGroupName "PSQL-Lab-RG" -ServerName "youruniquesqlserver" -Location "WestEurope" -SqlAdministratorCredentials $sqlAdminCred
  ```
- Configuring Firewall Rules:  
  Allow your IP:
  ```powershell
  New-AzSqlServerFirewallRule -ResourceGroupName "PSQL-Lab-RG" -ServerName "youruniquesqlserver" -FirewallRuleName "AllowMyIP" -StartIpAddress "X.X.X.X" -EndIpAddress "X.X.X.X"
  ```
- Creating a Database:  
  ```powershell
  New-AzSqlDatabase -ResourceGroupName "PSQL-Lab-RG" -ServerName "youruniquesqlserver" -DatabaseName "LabDB" -Edition "Basic"
  ```

Hands‑On Exercise 4:  
1. Run `Connect-AzAccount` to log in.  
2. Create a resource group, then provision an Azure SQL Server using unique naming and admin credentials.  
3. Configure the firewall to allow your connection.  
4. Create a database named "LabDB" and test connectivity using `Invoke-Sqlcmd`.

#### 2. Azure SQL User Management and Security

- Contained Database Users:  
  Create a contained user in an Azure SQL Database:
  ```powershell
  Invoke-Sqlcmd -ServerInstance "youruniquesqlserver.database.windows.net" -Database "LabDB" -Username "sqladminuser" -Password "<YourPassword>" -Query "
    CREATE USER reportingUser WITH PASSWORD = 'User@1234';
    ALTER ROLE db_datareader ADD MEMBER reportingUser;
  "
  ```
- Azure AD Administration (Optional):  
  Set an Azure AD admin for the server:
  ```powershell
  Set-AzSqlServerActiveDirectoryAdministrator -ResourceGroupName "PSQL-Lab-RG" -ServerName "youruniquesqlserver" -DisplayName "YourADAdmin" -ObjectId "<AzureAD_ObjectID>"
  ```

#### 3. Automating Azure SQL Maintenance

- Scaling the Database:  
  Scale up or down using:
  ```powershell
  Set-AzSqlDatabase -ResourceGroupName "PSQL-Lab-RG" -ServerName "youruniquesqlserver" -DatabaseName "LabDB" -Edition "Standard" -ServiceObjectiveName "S1"
  ```
  Scale down later by switching to a lower tier (e.g., Basic).
- Azure Automation Runbooks:  
  Outline how to schedule recurring tasks (like scaling adjustments or maintenance scripts) using Azure Automation Runbooks that run PowerShell scripts.

Hands‑On Exercise 5:  
Implement or describe a scenario where you scale the "LabDB" database up during business hours and down at night. Discuss how you would schedule these tasks (via Azure Automation Runbooks or Task Scheduler) and monitor performance.

---

### Quiz: Test Your Knowledge – Azure

1. Azure Environment Setup: Which cmdlet logs you in to your Azure account?  
   A. `Connect-AzAccount`  
   B. `Login-AzAccount`  
   C. `Connect-Azure`  
   D. `Authenticate-Az`

2. Resource Provisioning: Which of the following is required before creating an Azure SQL Server?  
   A. Installing SQL Server on an Azure VM  
   B. Creating a Resource Group  
   C. Setting up a VPN  
   D. Configuring Windows Authentication

3. Firewall Configuration: To allow your local machine to connect to an Azure SQL Server, which parameter is essential in creating a firewall rule?  
   A. Database name  
   B. StartIpAddress and EndIpAddress  
   C. Resource Group name only  
   D. Azure AD object ID

4. Database Creation: Which cmdlet is used to create a new Azure SQL Database?  
   A. `New-AzSqlDatabase`  
   B. `New-AzureSqlDatabase`  
   C. `Create-AzSqlDB`  
   D. `New-SqlDatabase`

5. User Management in Azure SQL: How are contained database users created in Azure SQL?  
   A. Using `New-AzSqlUser`  
   B. Via T‑SQL executed with `Invoke-Sqlcmd`  
   C. Through the Azure Portal only  
   D. With the `Create-AzUser` cmdlet

6. Scaling the Database: Which command scales an Azure SQL Database to a higher performance tier?  
   A. `Set-AzSqlDatabase`  
   B. `Scale-AzSqlDatabase`  
   C. `Update-AzSqlDatabase`  
   D. `Invoke-AzSqlScale`

7. Automation: What Azure feature can you use to schedule recurring PowerShell scripts to manage Azure SQL resources?  
   A. Azure Automation Runbook  
   B. Azure Logic Apps  
   C. Azure Functions  
   D. SQL Server Agent

---

## Best Practices for PowerShell and SQL Server

- Use Least Privilege: Run sessions with only the necessary permissions.  
- Secure Credentials: Avoid hard‑coding credentials; use `Get-Credential` or secure vaults.  
- Error Handling and Logging: Implement Try/Catch blocks and log outcomes.  
- Idempotence: Ensure scripts can be safely re‑run.  
- Keep Modules Updated: Regularly update the SqlServer and Az modules.  
- Documentation and Source Control: Comment your scripts and use version control.

---

## Exam Guidance

- Know Key Cmdlets:  
  - On‑prem: `Invoke-Sqlcmd`, `Backup-SqlDatabase`, `Restore-SqlDatabase`, navigating `SQLSERVER:\`  
  - Azure: `New-AzSqlServer`, `New-AzSqlDatabase`, `Set-AzSqlDatabase`, `New-AzSqlServerFirewallRule`
- Understand Environment Differences: Recognize the differences between on‑prem and cloud administration, and between Windows PowerShell and PowerShell Core.
- Practice Scenarios: Simulate tasks such as user management, backups, scaling, and monitoring using the provided exercises.
- Automation Benefits: Know why automating these tasks with PowerShell is more efficient than manual GUI operations.
