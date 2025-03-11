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

This section focuses on using Windows PowerShell to deploy and manage SQL Server in an on‑premises environment. You'll work on a virtual machine configured with SQL Server and learn how to install PowerShell 7 and the SqlServer module, configure SQL Server authentication settings, verify connectivity, and automate administrative tasks such as creating databases, managing logins and users, and scheduling backups using dedicated PowerShell cmdlets and scripts.

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
   ALTER LOGIN [sa] WITH PASSWORD = '{password}';
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
Invoke-Sqlcmd -ServerInstance "YourSQLServerName" -Database master -Credential $cred -Query "SELECT @@VERSION;" -TrustServerCertificate:$true
```

<img src="https://github.com/user-attachments/assets/3785690f-768f-4784-86db-a5607c6f951d" width="600" title="Snipped March 10, 2025" />

------------------------- -------------------------

### Hands‑On Exercise: Automated Setup and Configuration

In this exercise, you will verify that SQL Server is running, create a new database named trainingdb, and create a SQL login named traininguser that is mapped to trainingdb with read permissions

Step 1: Verify that SQL Server is running  
```powershell
Get-Service -Name MSSQLSERVER
```

Step 2: Create a new database named `trainingdb` 
```powershell
Invoke-Sqlcmd -ServerInstance "YourSQLServerName" -Database master -TrustServerCertificate:$true -Query "CREATE DATABASE trainingdb;"
```

Use SQL Server Management Studio to confirm database creation.

Step 3: Create login `traininguser`  
```powershell
Invoke-Sqlcmd -ServerInstance "YourSQLServerName" -Database master -TrustServerCertificate:$true -Query "CREATE LOGIN traininguser WITH PASSWORD = '{password}';"
```

Use SQL Server Management Studio to confirm login creation.

Step 4: Map login `traininguser` database `trainingdb` with read permissions  
```powershell
Invoke-Sqlcmd -ServerInstance "YourSQLServerName" -Database trainingdb -TrustServerCertificate:$true -Query "CREATE USER traininguser FOR LOGIN traininguser; EXEC sp_addrolemember 'db_datareader', 'traininguser';"
```

Use SQL Server Management Studio to confirm user creation and permissions.

------------------------- -------------------------

### Quiz: Windows PowerShell (on‑prem)

1. Which cmdlet is used to run a T‑SQL query on a SQL Server instance?  
   A. `Get-SqlQuery`  
   B. `Invoke-Sqlcmd`  
   C. `Run-SqlQuery`  
   D. `Start-Sqlcmd`

2. What is the provider path added by the SqlServer module for on‑premises SQL Server?  
   A. `SQLSERVER:\`  
   B. `SQL:\`  
   C. `DBSERVER:\`  
   D. `SQLDB:\`

3. Which T‑SQL statement creates a new database named trainingdb on SQL Server?  
   A. `NEW DATABASE trainingdb;`  
   B. `MAKE DATABASE trainingdb;`  
   C. `ADD DATABASE trainingdb;`  
   D. `CREATE DATABASE trainingdb;`

4. After creating a SQL login, what must you do to allow that login to access a specific database?  
   A. Nothing; logins automatically have access  
   B. Create a corresponding database user and map the login  
   C. Restart the SQL Server  
   D. Change the authentication mode

5. Which parameter should be included with `Invoke-Sqlcmd` to bypass SSL certificate validation issues?  
   A. `-IgnoreCertificateErrors`  
   B. `-SkipCertValidation`  
   C. `-BypassSSL`  
   D. `-TrustServerCertificate`

6. Which cmdlet is used to verify that the SQL Server service is running?  
   A. `Get-Service`  
   B. `Test-Service`  
   C. `Check-Service`  
   D. `Show-Service`

7. When testing remote commands locally to avoid WinRM issues, which computer name should you use?  
   A. Your machine’s fully qualified domain name  
   B. The domain name  
   C. `localhost`  
   D. `127.0.0.1`

8. Which cmdlet is commonly used to capture the output of a command and write it to a file?  
   A. `Out-File`  
   B. `Write-Output`  
   C. `Export-Csv`  
   D. `Set-Content`

9. What is the recommended method for securely obtaining user credentials in PowerShell?  
   A. `Read-Host`  
   B. `Import-Credential`  
   C. `Secure-Credential`  
   D. `Get-Credential`

10. What is the primary purpose of using a pipeline (`|`) in PowerShell?  
    A. To execute multiple commands concurrently  
    B. To format the output automatically  
    C. To pass the output of one cmdlet as input to another  
    D. To connect to remote systems

-------------------------

#### Answer Key

1. Answer: B  
   Explanation: The `Invoke-Sqlcmd` cmdlet is specifically designed to execute T‑SQL queries on a SQL Server instance, whereas the other options are not valid cmdlets for this purpose.

2. Answer: A  
   Explanation: The SqlServer module adds the provider path `SQLSERVER:\` for on‑premises SQL Server, making it easy to navigate SQL Server objects.

3. Answer: D  
   Explanation: `CREATE DATABASE trainingdb;` is the correct T‑SQL statement to create a new database named trainingdb, following the standard SQL syntax.

4. Answer: B  
   Explanation: After creating a SQL login, you must create a corresponding database user and map the login to the database to allow access.

5. Answer: D  
   Explanation: The `-TrustServerCertificate` parameter instructs `Invoke-Sqlcmd` to bypass SSL certificate validation issues, which is necessary when the server’s certificate is not trusted.

6. Answer: A  
   Explanation: The `Get-Service` cmdlet is used to check the status of Windows services, including the SQL Server service.

7. Answer: C  
   Explanation: Using `localhost` when testing remote commands locally helps avoid WinRM configuration issues.

8. Answer: A  
   Explanation: The `Out-File` cmdlet is commonly used to capture command output and write it to a file for review.

9. Answer: D  
   Explanation: The `Get-Credential` cmdlet is the recommended method for securely obtaining user credentials in PowerShell.

10. Answer: C  
    Explanation: The primary purpose of a pipeline (`|`) in PowerShell is to pass the output of one cmdlet as input to another, enabling streamlined data processing.

------------------------- ------------------------- ------------------------- -------------------------

## Azure PowerShell (via Cloud Shell)

This section focuses on using Azure PowerShell modules to deploy and manage Azure SQL resources. You will use Azure Cloud Shell to create and configure resource groups, Azure SQL servers, firewall rules, and databases, and verify connectivity using PowerShell.

Click the "Cloud Shell" icon in the upper-right of the Azure Portal interface.

<img src="https://github.com/user-attachments/assets/2a19d4bb-83ce-4c9e-b7a4-ad2d3bc02d08" width="800" title="Snipped March 3, 2025" />

-------------------------

### Create a Resource Group

Execute the following PowerShell command to create a resource group:

```powershell
New-AzResourceGroup -Name "prefixrg" -Location "westus"
```

This command creates a resource group named `prefixrg` in the West US region.

<img src="https://github.com/user-attachments/assets/b0c2be09-a50e-42f9-aaf0-62494f016a93" width="600" title="Snipped March 3, 2025" />

-------------------------

### Provision an Azure SQL Server  

Provision a new Azure SQL server by prompting for an admin credential:
```powershell
$sqlAdminCred = Get-Credential -UserName "sqladminuser" -Message "Enter a strong password for the new Azure SQL server admin"
New-AzSqlServer -ResourceGroupName "PSQL-Lab-RG" -ServerName "youruniquesqlserver" -Location "WestEurope" -SqlAdministratorCredentials $sqlAdminCred
```
This command creates an Azure SQL server with a unique name. Replace "youruniquesqlserver" with a globally unique server name.

- Configuring Firewall Rules  
Configure a firewall rule to allow your local machine's IP to connect to the Azure SQL server:
```powershell
New-AzSqlServerFirewallRule -ResourceGroupName "PSQL-Lab-RG" -ServerName "youruniquesqlserver" -FirewallRuleName "AllowMyIP" -StartIpAddress "X.X.X.X" -EndIpAddress "X.X.X.X"
```
Replace "X.X.X.X" with your public IP address. This ensures that your client can access the server.

- Creating a Database  
Create a new Azure SQL Database on the server:
```powershell
New-AzSqlDatabase -ResourceGroupName "PSQL-Lab-RG" -ServerName "youruniquesqlserver" -DatabaseName "LabDB" -Edition "Basic"
```
This command provisions a new database named "LabDB" using the Basic pricing tier.

- Verifying Connectivity (Optional)  
Although Azure Cloud Shell typically provides a managed environment, you can verify connectivity by running a simple T-SQL query. For example, using `Invoke-Sqlcmd` (ensure that your server’s connection settings allow SQL queries):
```powershell
Invoke-Sqlcmd -ServerInstance "youruniquesqlserver.database.windows.net" -Database "LabDB" -Query "SELECT @@VERSION;" -TrustServerCertificate:$true
```
This confirms that your Azure SQL database is reachable.

------------------------- -------------------------

### Hands‑On Exercise: Deploy and Configure an Azure SQL Database

In this exercise you will create a resource group, provision an Azure SQL server, configure a firewall rule, and create a new database named "LabDB." Finally, you'll verify connectivity by running a simple query.

Step 1: Create a resource group
```powershell
New-AzResourceGroup -Name "PSQL-Lab-RG" -Location "WestEurope"
```

Step 2: Provision an Azure SQL server and set the admin credentials
```powershell
$sqlAdminCred = Get-Credential -UserName "sqladminuser" -Message "Enter a strong password for the new Azure SQL server admin"
New-AzSqlServer -ResourceGroupName "PSQL-Lab-RG" -ServerName "youruniquesqlserver" -Location "WestEurope" -SqlAdministratorCredentials $sqlAdminCred
```

Step 3: Configure a firewall rule to allow your IP address (replace X.X.X.X with your public IP)
```powershell
New-AzSqlServerFirewallRule -ResourceGroupName "PSQL-Lab-RG" -ServerName "youruniquesqlserver" -FirewallRuleName "AllowMyIP" -StartIpAddress "X.X.X.X" -EndIpAddress "X.X.X.X"
```

Step 4: Create a new database named LabDB
```powershell
New-AzSqlDatabase -ResourceGroupName "PSQL-Lab-RG" -ServerName "youruniquesqlserver" -DatabaseName "LabDB" -Edition "Basic"
```

Step 5: Verify connectivity by running a simple query (optional)
```powershell
Invoke-Sqlcmd -ServerInstance "youruniquesqlserver.database.windows.net" -Database "LabDB" -Query "SELECT @@VERSION;" -TrustServerCertificate:$true
```

------------------------- -------------------------

### Quiz: Azure PowerShell (Managing Azure SQL)

1. Azure Environment Setup: Which cmdlet logs you in to your Azure account?  
   A. `Connect-AzAccount`  
   B. `Login-AzAccount`  
   C. `Connect-Azure`  
   D. `Authenticate-Az`

2. Resource Provisioning: Which command creates a new resource group in Azure?  
   A. `New-AzResourceGroup`  
   B. `New-AzureResourceGroup`  
   C. `Create-AzResourceGroup`  
   D. `Set-AzResourceGroup`

3. Azure SQL Server Provisioning: Which parameter is required when creating a new Azure SQL server using PowerShell?  
   A. `-SqlAdministratorCredentials`  
   B. `-AdminPassword`  
   C. `-ServerAdmin`  
   D. `-Credential`

4. Firewall Configuration: Which cmdlet is used to configure firewall rules for an Azure SQL server?  
   A. `New-AzSqlServerFirewallRule`  
   B. `Set-AzSqlFirewallRule`  
   C. `Configure-AzSqlFirewall`  
   D. `Add-AzSqlServerFirewallRule`

5. Database Creation: Which cmdlet is used to create a new Azure SQL Database?  
   A. `New-AzSqlDatabase`  
   B. `New-AzureSqlDatabase`  
   C. `Create-AzSqlDB`  
   D. `New-SqlDatabase`

6. In Azure Cloud Shell, which module is pre‑installed for managing Azure resources?  
   A. `Az`  
   B. `SqlServer`  
   C. `AzureRM`  
   D. `MSOnline`

7. When connecting to an Azure SQL Database using PowerShell, what is the correct format for the server name?  
   A. `yourserver.database.windows.net`  
   B. `yourserver.cloud.windows.net`  
   C. `yourserver.sql.azure.com`  
   D. `yourserver.windows.net`

8. Which cmdlet can be used to execute a T‑SQL query against an Azure SQL Database?  
   A. `Invoke-Sqlcmd`  
   B. `Test-AzSqlConnection`  
   C. `Get-AzSqlDatabase`  
   D. `Invoke-AzSqlQuery`

9. Which of the following is a key benefit of using Azure Cloud Shell for managing Azure SQL resources?  
   A. The Az modules are pre‑installed and maintained by Azure  
   B. It provides direct access to the Windows registry  
   C. It allows installation of custom modules without restrictions  
   D. It supports only Windows PowerShell 5.1

10. Which approach is recommended for automating recurring maintenance tasks for Azure SQL resources?  
    A. Azure Automation Runbooks  
    B. SQL Server Agent  
    C. Manual execution via SSMS  
    D. Windows Task Scheduler on a local machine

-------------------------

### Answer Key

1. **Answer: A**  
   Explanation: The `Connect-AzAccount` cmdlet is used to log in to your Azure account, providing the necessary authentication for managing Azure resources.

2. **Answer: A**  
   Explanation: The `New-AzResourceGroup` cmdlet is the correct command to create a new resource group in Azure, which organizes related resources.

3. **Answer: A**  
   Explanation: When provisioning an Azure SQL server, the `-SqlAdministratorCredentials` parameter is required to specify the admin credentials for the server.

4. **Answer: A**  
   Explanation: The `New-AzSqlServerFirewallRule` cmdlet is used to create firewall rules for an Azure SQL server, allowing you to control which IP addresses can access the server.

5. **Answer: A**  
   Explanation: The `New-AzSqlDatabase` cmdlet is used to create a new Azure SQL Database on a given server.

6. **Answer: A**  
   Explanation: In Azure Cloud Shell, the Az modules are pre‑installed and managed by Azure, removing the need for manual installation.

7. **Answer: A**  
   Explanation: Azure SQL Database uses the fully qualified domain name format `yourserver.database.windows.net` for its server name.

8. **Answer: A**  
   Explanation: The `Invoke-Sqlcmd` cmdlet can be used to execute T‑SQL queries against an Azure SQL Database, verifying connectivity and running commands.

9. **Answer: A**  
   Explanation: A key benefit of using Azure Cloud Shell is that the Az modules are pre‑installed and maintained by Azure, which reduces setup time and ensures you have the latest functionality.

10. **Answer: A**  
    Explanation: Azure Automation Runbooks are recommended for automating recurring maintenance tasks for Azure SQL resources because they provide a managed, scalable solution for automation in the cloud.
