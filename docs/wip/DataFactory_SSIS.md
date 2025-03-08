# Data Factory + SSIS

In this demonstration you will:
- Create an ADF instance.
- Set up a Self‑Hosted Integration Runtime (IR) that runs on your VM.
- Install and configure the IR on your on‑premises VM (hosting SSIS).
- Create a pipeline in ADF that calls an SSIS package.

This walkthrough assumes you already have an Azure subscription and a VM (with Windows and SQL Server/SSIS installed) that can communicate with Azure.

------------------------- ------------------------- ------------------------- -------------------------

## Configure On‑Prem

### Pre-Requisite Resources

Provision a Virtual Machine:
- Set up a Windows Server virtual machine
- Install SQL Server with the Integration Services feature enabled
  
Network Configuration:
- Ensure the VM can communicate with Azure by allowing outbound connections
- Configure firewall settings to permit necessary traffic for the Integration Runtime

------------------------- -------------------------

### Visual Studio

Download and install Visual Studio 2022 Community from [Visual Studio Downloads](https://visualstudio.microsoft.com/downloads/)
- In the Visual Studio Installer, check to include the "Data storage and processing" workload

Launch Visual Studio and navigate to Extensions → Manage Extensions
- Search for and download "SQL Server Integration Services Projects 2022"
- Close Visual Studio and run "Microsoft.DataTools.IntegrationServices" from the "Downloads" folder

------------------------- -------------------------

### SQL Server Management Studio

Launch SQL Server Management Studio, connect to your local database, and then click "New Query".

Create demonstration tables and data by executing the following T-SQL:

```sql
CREATE DATABASE SSISDemoDB
GO

USE SSISDemoDB
GO

CREATE TABLE dbo.DemoProducts (
    ProductID INT IDENTITY(1,1) PRIMARY KEY,
    ProductName VARCHAR(100) NOT NULL,
    Quantity INT NOT NULL,
    UnitPrice DECIMAL(10,2) NOT NULL
)
GO

INSERT INTO dbo.DemoProducts (ProductName, Quantity, UnitPrice)
VALUES
('Widget A', 100, 9.99),
('Widget B', 50, 14.99),
('Widget C', 25, 29.99)
GO
```

------------------------- -------------------------

### SQL Server Integration Services

Launch Visual Studio and click "Create a new project"  
- On the "Create a new project" popup, search for and select "Integration Services Project" then click "Next"  
- Name your project (for example, SSISDemoPipeline), confirm location, and click "Create"

In the Data Flow Designer, "Package.dtsx" package should be opened by default
- Drag a "Data Flow Task" onto the "Control Flow" canvas from the "SSIS Toolbox" 
- Double-click the Data Flow Task to switch to the Data Flow designer

Drag "OLE DB Source" from "SSIS Toolbox" >> "Other Sources" onto the canvas  
- Double-click to open the "OLE DB Source Editor"
- Click "New" to create an "OLE DB Connection Manager":
  - Provider: `Native OLE DB\Microsoft OLE DB Driver for SQL Server`
  - Server or file name: `.` or `localhost`
  - Initial Catalog: `SSISDemoDB`
- Click "Test Connection" to confirm success then click "OK"
- Complete "OLE DB Source Editor" configuration:
  - OLE DB Connection Manager: `LocalHost.SSISDemoDB`
  - Data Access Mode: `Table or view`
  - Name of the Table or the View: `[dbo].[DemoProducts]`
- Click "Preview" to confirm success then click "OK"

Drag "Flat File Destination" from "SSIS Toolbox" >> "Other Destinations" onto the canvas  
- Create a data path from "OLE DB Source" to "Flat File Destination"
- Double-click to open the "Flat File Destination Editor"
- Click "New" to create an "Flat File Connection Manager":
  - Flat File Format: `Delimited`
- On the "Flat File Connection Manager Editor", browse to the "Downloads" folder, enter File Name `DemoProducts`, and then click "OK"
- Click "Mappings" and confirm success

Test the package by clicking "Start".

-------------------------

#### Expected Output

```
SSIS package "C:\Users\rchapler\source\repos\SSISDemoPipeline\SSISDemoPipeline\Package.dtsx" starting.
Information: 0x4004300A at Data Flow Task, SSIS.Pipeline: Validation phase is beginning.
Information: 0x4004300A at Data Flow Task, SSIS.Pipeline: Validation phase is beginning.
Information: 0x40043006 at Data Flow Task, SSIS.Pipeline: Prepare for Execute phase is beginning.
Information: 0x40043007 at Data Flow Task, SSIS.Pipeline: Pre-Execute phase is beginning.
Information: 0x402090DC at Data Flow Task, Flat File Destination [2]: The processing of file "C:\Users\rchapler\Downloads\DemoProducts.txt" has started.
Information: 0x4004300C at Data Flow Task, SSIS.Pipeline: Execute phase is beginning.
Information: 0x40043008 at Data Flow Task, SSIS.Pipeline: Post Execute phase is beginning.
Information: 0x402090DD at Data Flow Task, Flat File Destination [2]: The processing of file "C:\Users\rchapler\Downloads\DemoProducts.txt" has ended.
Information: 0x4004300B at Data Flow Task, SSIS.Pipeline: "Flat File Destination" wrote 3 rows.
Information: 0x40043009 at Data Flow Task, SSIS.Pipeline: Cleanup phase is beginning.
SSIS package "C:\Users\rchapler\source\repos\SSISDemoPipeline\SSISDemoPipeline\Package.dtsx" finished: Success.
```

------------------------- -------------------------

### Section Wrap-Up

In this section, we:
- Set up our on‑prem environment by provisioning a Windows Server VM with SQL Server Integration Services
- Configured Visual Studio with the required data tools
- Created demonstration data using SSMS
- Developed and tested an SSIS package locally, ensuring that all components work together as expected

Next, we'll migrate these SSIS packages to the cloud by deploying them to an Azure‑hosted SSISDB and configuring Azure Data Factory to execute them via an Azure‑SSIS Integration Runtime.

------------------------- ------------------------- ------------------------- -------------------------

## Migrate to Azure

In this section, we will migrate your on‑premises SSIS packages to a cloud‑hosted SSIS catalog (SSISDB) in Azure SQL Database, ensuring they are ready for execution in Azure Data Factory pipelines.

------------------------- -------------------------

### Pre-Requisite Resources

- Azure SQL Server and Database, configured for SQL Server Authentication

------------------------- -------------------------

### Build Solution

Open `SSISDemoPipeline` in Visual Studio
- Right-click the project in Solution Explorer, then click "Properties"
- On the "...Property Pages" popup, "Configuration Properties" >> "General" form, change "TargetServerVersion" to "SQL Server 2017" and then click "OK"
- Click "Build" and then "Build Solution" in the toolbar.

#### Expected Output

```
Build started at 12:25 PM...
------ Build started: Project: SSISDemoPipeline (SQL Server 2017), Configuration: Development ------
Build started: SQL Server Integration Services project: Incremental ...
Starting project consistency check ...
Project consistency check completed. The project is consistent.
File 'C:\Users\rchapler\source\repos\SSISDemoPipeline\SSISDemoPipeline\obj\Development\SSISDemoPipeline.dtproj' get updated.
File 'C:\Users\rchapler\source\repos\SSISDemoPipeline\SSISDemoPipeline\obj\Development\Project.params' get updated.
File 'C:\Users\rchapler\source\repos\SSISDemoPipeline\SSISDemoPipeline\obj\Development\Package.dtsx' get updated.
Applied active configuration to 'Project.params'.
Applied active configuration to 'Package.dtsx'.
SSISDemoPipeline -> C:\Users\rchapler\source\repos\SSISDemoPipeline\SSISDemoPipeline\bin\Development\SSISDemoPipeline.ispac
Build complete -- 0 errors, 0 warnings
========== Build: 1 succeeded or up-to-date, 0 failed, 0 skipped ==========
========== Build completed at 12:25 PM and took 01.315 seconds ==========
```

------------------------- -------------------------

### Deploy

Right-click your project (`SSISDemoPipeline`) in Solution Explorer and select "Deploy" to open the "Integration Services Deployment Wizard"
- On the "Select Deployment Target" tab, click "SSIS in Data Factory" and then "Next >"
- Complete the form on the "Select Destination" tab
  - Server name: Enter your Azure SQL Database server name (`{prefix}sds.database.windows.net`)
  - Authentication: Select "SQL Server Authentication"
  - Login: Enter your SQL Server admin username (`<your_admin_user>@{prefix}sds`)
  - Password: Enter the admin password you set when creating the Azure SQL Server
  - Path: After successful authentication, click the "Browse..." button next to "Path" and select the `/SSISDB` catalog (automatically populated once connected)

Click "Connect" after filling in these fields to validate and proceed with deployment.










- When the SSIS Deployment Wizard opens, enter your Azure SQL Database server name (for example, `mydbserver.database.windows.net`)
- Select the appropriate authentication method (Azure AD authentication recommended)
- Confirm "Deploy to the SSIS Catalog" and follow the wizard prompts to deploy your packages to the Azure‑hosted SSISDB
- After deployment completes, verify packages appear in the SSISDB catalog by connecting to Azure SQL with SSMS or Azure Data Studio

### Post‑Migration Considerations

- Update any connection managers or environment references if they must point to Azure-specific resources
- Test the migrated packages directly using SSMS or via Azure‑SSIS Integration Runtime to ensure they run correctly
- Document any configuration changes or issues encountered during migration

------------------------- ------------------------- ------------------------- -------------------------

## Azure Data Factory

- Login to the [Azure Portal](https://portal.azure.com)  
- Click "Create a resource" and search for "Data Factory"  
- Provide the required details (name, subscription, resource group, region, etc.)  
- Click "Review + create," then "Create"  
- After deployment completes, you can access Data Factory Studio to manage and author pipelines  

------------------------- -------------------------

#### System Assigned Managed Identity  
- In the Azure Portal, navigate to your newly created Data Factory resource  
- In the left-hand menu, select Identity  
- On the System assigned tab, toggle Status to On  
- Click Save  
- Once enabled, Data Factory has a managed identity in Azure AD that can be granted access to other Azure services, such as Azure SQL Database

------------------------- -------------------------

#### Master Database Permissions  

Open SQL Server Management Studio (SSMS) and connect to the Azure SQL Server
- Execute the following T-SQL on the `master` database:
  ```sql
  CREATE USER [YourDataFactoryName] FROM EXTERNAL PROVIDER;
  ALTER ROLE dbmanager ADD MEMBER [YourDataFactoryName];
  ```

------------------------- -------------------------

### Integration Runtime

Open Data Factory Studio and navigate to "Manage" >> "Integration Runtimes"
- Click "+ New" under "Integration runtimes" and on the "Integration runtime setup" popout, choose "Azure‑SSIS" as the runtime type then click "Continue"  
- Complete the "General settings" form, then click "Continue"

Complete the "Deployment settings" form, including:
  - `Create SSIS catalog (SSISDB) hosted by Azure SQL Database server/Managed Instance`: CHECKED since we don't already have an existing SSISDB in Azure SQL
    - What it does: Automatically creates (or configures) an SSISDB database to store and manage your SSIS packages
    - Why you’d want it: If you don’t already have an SSISDB, checking this box ensures you have a centralized repository for deploying and executing packages in the cloud
    - If you don’t check it: You must have an existing SSISDB in Azure SQL Database or Managed Instance and plan to manage it yourself
  - `Catalog database server endpoint`: Select {Azure SQL Database Server}
    - What it does: Identifies the Azure SQL Database server or Managed Instance where SSISDB will be hosted (for example, `mydbserver.database.windows.net`)
  - `Use Microsoft Entra authentication...`: Use System Managed Identity for Data Factory
    - What it does: Lets you authenticate to Azure SQL Database/Managed Instance using Azure AD credentials instead of SQL credentials
    - Why you’d want it: Centralizes identity management and may improve security
    - If you don’t enable it: You’ll have to use SQL authentication with the admin username/password
  - `Use dual standby Azure‑SSIS Integration Runtime...`: UNCHECKED since this is only for demonstration
    - What it does: Sets up a standby IR in a paired region for high availability/disaster recovery
    - Why you’d want it: Ensures minimal downtime if there’s a regional outage
    - If you don’t enable it: You’ll have a single IR instance without automatic failover
  - `Catalog database service tier`: "S1" since this is only for demonstration
    - What it does: Determines performance, cost, and resource limits (e.g., S1, S2, etc.)
    - Why you’d want it: Higher tiers can handle more transactions and concurrency, but cost more. For testing or light workloads, S1 is typically sufficient
  - `Create package stores...`: UNCHECKED since this is only for demonstration  
    - What it does: Lets you configure additional storage locations (e.g., file system, Azure Files, SQL) to hold SSIS packages outside of SSISDB
    - Why you’d want it: If you have custom package deployment needs or want to manage packages outside of SSISDB
    - If you don’t enable it: Packages are deployed solely to SSISDB, which is sufficient for most scenarios

Click "Test Connection" and confirm success, then click "Continue". 

Confirm "Advanced Settings" configuration and then click "Continue".

Review "Summary" and then click "Create".

Wait for `Status` to change to "Running".

------------------------- -------------------------

### Deploy and Execute SSIS Packages with Azure‑SSIS IR

- Deploy your SSIS packages to the SSIS catalog (SSISDB) hosted on an Azure SQL Database or a SQL Managed Instance  
- In Data Factory Studio, navigate to the "Author" tab and create a new pipeline  
- Drag the "Execute SSIS Package" activity onto the pipeline canvas  
- Configure the activity with the following settings:  
  - Package location: Select the SSIS package from your SSISDB catalog  
  - Connection details: Provide necessary parameters, such as credentials and any runtime parameters  
- Assign the Azure‑SSIS Integration Runtime to the activity  
- Save and publish your pipeline  
- Trigger the pipeline and monitor its execution via the "Monitor" tab
