# PowerShell: Fundamentals

This curriculum provides a structured introduction to using PowerShell (in both Windows PowerShell 5.1 and cross‑platform PowerShell Core 7+) for SQL Server administration. You will learn how to perform common tasks such as SQL Server setup, user and permission management, automating maintenance, and connecting to Azure SQL resources using dedicated PowerShell cmdlets and scripts. Throughout the course, we’ll highlight differences between Windows PowerShell and PowerShell Core where applicable, offer hands‑on exercises to practice each concept, and provide best practices and exam guidance to reinforce learning.

## Objectives 

- Understand PowerShell Basics for SQL Server
  - Explain the benefits of using PowerShell for SQL Server administration and recognize differences between Windows PowerShell and PowerShell Core in a database context
- Manage On-Premises SQL Server with PowerShell
  - Use PowerShell cmdlets and scripts to configure SQL Server instances, create databases, manage logins/users, and automate routine maintenance tasks (backups, checks)
- Automate Azure SQL Operations
  - Leverage Azure PowerShell modules to deploy and manage Azure SQL resources (servers and databases), handle cloud connectivity (firewall rules, authentication), and integrate on‑premises to cloud automation 
- Apply Best Practices
  - Follow scripting best practices for security, reliability, and efficiency in PowerShell when working with SQL Server
- Prepare for Exams and Real‑world Scenarios
  - Tackle exam‑style questions and scenarios that test your knowledge of PowerShell for SQL, ensuring you can apply concepts in practice and in certification environments

------------------------- ------------------------- ------------------------- -------------------------

## General Concepts

Before you start using PowerShell for SQL Server administration, it's essential to understand the core concepts of PowerShell scripting. This section covers syntax, scripting best practices, credential management, error handling, logging, and considerations for script portability between local environments and Azure Cloud Shell.

### Syntax and Command Structure

#### Cmdlet Naming Convention
PowerShell cmdlets follow a consistent Verb-Noun naming pattern, which helps you understand what action is performed and on what object.

```powershell
Get-Process
```
Retrieves a list of all running processes. "Get" indicates the action of retrieving, and "Process" is the object being retrieved.

-------------------------

#### Pipelines
Pipelines allow you to pass the output of one cmdlet directly into another, making it easy to filter and process data.

```powershell
Get-Process | Where-Object { $_.CPU -gt 100 }
```
Retrieves all running processes and pipes the output to `Where-Object` to filter out only those processes that are using more than 100 units of CPU.

-------------------------

#### Parameters and Aliases
Cmdlets accept parameters that modify their behavior, and many cmdlets have aliases—shorter or alternative names for convenience.

```powershell
Get-ChildItem -Path "C:\Windows" -Recurse
```
Lists all items (files and folders) in the "C:\Windows" directory and all its subdirectories. The `-Recurse` parameter tells the cmdlet to include all nested items.

------------------------- -------------------------

### Basic Scripting Concepts

Here's an updated version with distinct examples for "Writing Scripts" and "Capturing Script Output":

#### Writing Scripts
Scripts allow you to automate tasks by combining multiple cmdlets and logic.

```powershell
# This script retrieves the current date and writes it to a text file.
$currentDate = Get-Date
$currentDate | Out-File -FilePath "C:\Temp\CurrentDate.txt"
```

Writes the current date and time to "C:\Temp\CurrentDate.txt".

-------------------------

#### Using Functions
Functions help organize your code into reusable blocks.

```powershell
function Get-HighCPUProcesses {
    param(
        [int]$MinCPU = 50
    )
    Get-Process | Where-Object { $_.CPU -gt $MinCPU }
}

# Call the function with a custom threshold:
Get-HighCPUProcesses -MinCPU 100
```
This function retrieves processes with CPU usage above a specified threshold, making the code modular and reusable.

------------------------- -------------------------

### Credential Management

#### Securely Handling Credentials
PowerShell provides the `Get-Credential` cmdlet to securely prompt for user credentials.

```powershell
$cred = Get-Credential
```
This command prompts you for a username and password, storing them in the `$cred` variable without hard-coding sensitive information.

-------------------------

#### Using Credentials in Commands
You can pass credentials to cmdlets that require authentication. To avoid remote WinRM configuration issues during testing, use "localhost" as the computer name.

```powershell
$cred = Get-Credential
Invoke-Command -ComputerName "localhost" -Credential $cred -ScriptBlock { Get-Service }
```

Executes the `Get-Service` command on the local machine using the credentials stored in `$cred`. For remote connections, ensure that WinRM is properly configured (for example, using HTTPS transport or adding the destination machine to the TrustedHosts list) to avoid authentication errors.

------------------------- -------------------------

### Error Handling and Debugging

#### Using Try/Catch Blocks
Error handling in PowerShell is achieved using Try/Catch blocks.

```powershell
try {
    Get-Content "C:\NonExistentFile.txt"
}
catch {
    Write-Error "An error occurred: $_"
}
```
This script attempts to read a non-existent file and, upon failure, catches the error and outputs a custom error message.

-------------------------

#### Controlling Error Behavior
You can control how errors are handled using the `-ErrorAction` parameter.

```powershell
Get-ChildItem -Path "C:\InvalidPath" -ErrorAction SilentlyContinue
```
This command suppresses errors when attempting to list a non-existent directory.

------------------------- -------------------------

### Logging and Output Management

#### Capturing Script Output
It is important to capture and manage output for logging and later analysis.

```powershell
$output = Get-Process
$output | Out-File "C:\Temp\ProcessOutput.txt"
```
Captures the output of `Get-Process` into a variable and writes it to a file for review.

-------------------------

#### Transcript Logging
Transcript logging records all commands and output during a session.

```powershell
Start-Transcript -Path "C:\Temp\ScriptLog.txt"
# Execute various commands...
Stop-Transcript
```
Starts and stops a transcript, saving the entire session output to a log file.

------------------------- -------------------------

### Platform Considerations

#### Local vs. Azure Cloud Shell
- Local Environment: When running PowerShell on your local machine or on-premises (or on your own Azure VM), you are responsible for installing and updating modules.
- Azure Cloud Shell: In Azure Cloud Shell, the Az modules are pre-installed and managed by Azure. This means you don't need to worry about manual module installation or version upgrades.
  
#### Script Portability
Design your scripts to run in different environments by using parameters and environment variables.

```powershell
$dataPath = $env:DATA_PATH  # Use an environment variable for file paths
Get-ChildItem -Path $dataPath
```
This approach makes your script adaptable, so you don't have to hard-code paths for different environments.

------------------------- ------------------------- ------------------------- -------------------------

## Windows PowerShell (on-prem)

### Virtual Machine

Instantiate an Azure Virtual Machine (or equivalent):
  * Image: "Free SQL Server License: SQL Server 2022 Developer on Windows Server 2022"
  * SKU: "Standard_D2s_v3..."
  * Inbound Ports: RDP (3389) allowed
  * Boot Diagnostics: Disable

-------------------------

### PowerShell 7

Navigate to https://learn.microsoft.com/en-us/powershell/scripting/install/installing-powershell-on-windows.

Scroll to "Installing the MSI package".

<img src="https://github.com/user-attachments/assets/697cea24-a25d-4cf0-becc-0100538daf9e" width="800" title="Snipped March 3, 2025" />

Download and install the latest MSI version.

<img src="https://github.com/user-attachments/assets/ec8bfb4c-b506-46a6-908c-fc091642d438" width="400" title="Snipped March 3, 2025" />

------------------------- -------------------------

### SQL Server

#### Configuration

Below is an example of a "SQL Server Configuration" section that covers both Windows and SQL Server Authentication settings, including enabling the sa account and setting its password. You can integrate this section into the Windows PowerShell (on‑prem) part of your curriculum.

---

### Configuration

This section covers how to configure your SQL Server instance on the VM to support both Windows and SQL Server Authentication. It also includes steps to enable the sa account and set a secure password.

#### Configuring Authentication Mode

1. Verify Mixed Mode Authentication is Enabled:  
   Mixed Mode Authentication allows both Windows and SQL Server Authentication.  
   - In SQL Server Management Studio (SSMS), right‑click the server in Object Explorer, select Properties, and navigate to the Security page.  
   - Under Server authentication, ensure that SQL Server and Windows Authentication mode is selected.  
   - If you change this setting, restart SQL Server for the changes to take effect.

2. Using PowerShell to Verify (Optional):  
   Although you typically configure authentication via SSMS, you can run a query to check the authentication mode if needed:
   ```powershell
   Invoke-Sqlcmd -ServerInstance "YourSQLServerName" -Database master -Query "EXEC xp_instance_regread N'HKEY_LOCAL_MACHINE', N'Software\Microsoft\MSSQLServer\MSSQLServer', N'LoginMode';"
   ```
   The value of 2 indicates Windows Authentication mode and 1 indicates Mixed Mode; a combination (e.g., 2+1=3) may be used to indicate mixed mode in some versions. Refer to Microsoft documentation for your SQL Server version for specifics.

#### Enabling and Configuring the sa Account

1. Enable the sa Account:  
   To enable the sa account (if it's disabled), execute the following T-SQL command in SSMS or via PowerShell using Windows Integrated Authentication:
   ```sql
   ALTER LOGIN [sa] ENABLE;
   ```
   This command enables the sa login if it is currently disabled.

2. Set or Change the sa Password:  
   For security, set a strong password for the sa account:
   ```sql
   ALTER LOGIN [sa] WITH PASSWORD = 'YourNewStrongPassword';
   ```
   Replace `'YourNewStrongPassword'` with a password that meets your organization's security policies.

3. Testing the Configuration Using PowerShell:  
   You can test the new settings using `Invoke-Sqlcmd`. For example, after enabling and updating the sa account, verify connectivity using SQL Authentication:
   ```powershell
   $saCred = Get-Credential -UserName "sa" -Message "Enter the new sa password"
   Invoke-Sqlcmd -ServerInstance "YourSQLServerName" -Database master -Credential $saCred -Query "SELECT @@VERSION;" -TrustServerCertificate:$true
   ```
   This command confirms that the sa account is enabled, that the new password works, and that the SQL Server instance is reachable using SQL Authentication.

#### SqlServer Module

Execute the following PowerShell command to install the SqlServer Module:

```powershell
Install-Module -Name SqlServer -Scope CurrentUser
```

Then, execute the following PowerShell command to import the SqlServer Module and load the module into your current session (making its cmdlets immediately available):

```powershell
Import-Module SqlServer
```

<img src="https://github.com/user-attachments/assets/3813766e-4379-4fa1-9da8-ceb98c5955b8" width="600" title="Snipped March 3, 2025" />

-------------------------

#### SQL Server Connectivity

Execute the following PowerShell command to confirm network connectivity:

```powershell
Test-NetConnection -ComputerName "YourSQLServerName" -Port 1433
```

This command tells you whether the SQL Server's port is accessible, indicating that your VM has network access to the SQL Server instance.

<img src="https://github.com/user-attachments/assets/b0c2be09-a50e-42f9-aaf0-62494f016a93" width="600" title="Snipped March 3, 2025" />

-------------------------

#### SQL Server Authentication

To verify your connection to SQL Server and test the authentication method, you can run a simple T-SQL query using the `Invoke-Sqlcmd` cmdlet.

##### Windows Integrated Authentication

By default, this command uses Windows Integrated Authentication.

```powershell
Invoke-Sqlcmd -ServerInstance "YourSQLServerName" -Database master -TrustServerCertificate:$true -Query "SELECT @@VERSION;"
```

<img src="https://github.com/user-attachments/assets/272191e7-441f-4f39-b7d2-b16b8368b970" width="600" title="Snipped March 10, 2025" />

##### SQL Authentication

If SQL Authentication is required, first obtain credentials with `Get-Credential` and then supply them to the cmdlet:

```powershell
$cred = Get-Credential
Invoke-Sqlcmd -ServerInstance "cnbtraining" -Database master -Credential $cred -Query "SELECT @@VERSION;" -TrustServerCertificate:$true
```

<img src="https://github.com/user-attachments/assets/3785690f-768f-4784-86db-a5607c6f951d" width="600" title="Snipped March 10, 2025" />

------------------------- -------------------------

### Hands‑On Exercise #1 – Automate Setup and Configuration

In this exercise, you will verify that SQL Server is running, create a new database named TrainingDB, and create a SQL login named TrainingUser that is mapped to TrainingDB with read permissions

Step 1: Verify that SQL Server is running  
```powershell
Get-Service -Name MSSQLSERVER
```

Step 2: Create a new database named TrainingDB  
```powershell
Invoke-Sqlcmd -ServerInstance "YOURSERVER\DEFAULT" -Database master -Query "CREATE DATABASE TrainingDB;"
```

Step 3: Create a SQL login named TrainingUser and map it to TrainingDB with read permissions  
```powershell
Invoke-Sqlcmd -ServerInstance "YOURSERVER\DEFAULT" -Database master -Query "CREATE LOGIN TrainingUser WITH PASSWORD = 'Str0ngPass!';"
Invoke-Sqlcmd -ServerInstance "YOURSERVER\DEFAULT" -Database TrainingDB -Query "CREATE USER TrainingUser FOR LOGIN TrainingUser; EXEC sp_addrolemember 'db_datareader', 'TrainingUser';"
```

-------------------------

### Hands‑On Exercise #2

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

------------------------- -------------------------

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
